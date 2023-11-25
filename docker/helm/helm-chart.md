## Chart
{docsify-updated}

- [Chart](#chart)
  - [Chart 目录结构](#chart-目录结构)
  - [Chart.yaml](#chartyaml)
  - [自定义创建一个 Chart](#自定义创建一个-chart)
    - [templates](#templates)
    - [内置对象](#内置对象)
    - [Values 文件](#values-文件)


### Chart 目录结构
chart是一个组织在文件目录中的集合。目录名称就是chart名称（没有版本信息）。因而描述WordPress的chart可以存储在wordpress/目录中。当你创建一个新的 Chart 时，Helm有一定的结构,。要创建，运行 `helm create YOUR-CHART-NAME` 。一旦创建完毕，目录结构应该是这样的：
```
wordpress/
  Chart.yaml          # 包含了chart信息的YAML文件
  LICENSE             # 可选: 包含chart许可证的纯文本文件
  README.md           # 可选: 可读的README文件
  values.yaml         # chart 默认的配置值
  values.schema.json  # 可选: 一个使用JSON结构的values.yaml文件
  charts/             # 包含chart依赖的其他chart
  crds/               # 自定义资源的定义
  templates/          # 模板目录， 当和values 结合时，可生成有效的Kubernetes manifest文件
  templates/NOTES.txt # 可选: 包含简要使用说明的纯文本文件
```

### Chart.yaml
Chart.yaml文件是chart必需的。包含了以下字段：
```
apiVersion: chart API 版本 （必需）
name: chart名称 （必需）
version: 语义化2 版本（必需）
kubeVersion: 兼容Kubernetes版本的语义化版本（可选）
description: 一句话对这个项目的描述（可选）
type: chart类型 （可选）
keywords:
  - 关于项目的一组关键字（可选）
home: 项目home页面的URL （可选）
sources:
  - 项目源码的URL列表（可选）
dependencies: # chart 必要条件列表 （可选）
  - name: chart名称 (nginx)
    version: chart版本 ("1.2.3")
    repository: （可选）仓库URL ("https://example.com/charts") 或别名 ("@repo-name")
    condition: （可选） 解析为布尔值的yaml路径，用于启用/禁用chart (e.g. subchart1.enabled )
    tags: # （可选）
      - 用于一次启用/禁用 一组chart的tag
    import-values: # （可选）
      - ImportValue 保存源值到导入父键的映射。每项可以是字符串或者一对子/父列表项
    alias: （可选） chart中使用的别名。当你要多次添加相同的chart时会很有用
maintainers: # （可选）
  - name: 维护者名字 （每个维护者都需要）
    email: 维护者邮箱 （每个维护者可选）
    url: 维护者URL （每个维护者可选）
icon: 用做icon的SVG或PNG图片URL （可选）
appVersion: 包含的应用版本（可选）。不需要是语义化，建议使用引号
deprecated: 不被推荐的chart （可选，布尔值）
annotations:
  example: 按名称输入的批注列表 （可选）.
```

### 自定义创建一个 Chart
当我们使用下述命令创建一个chart时：
```
helm create mychart
```
会发现 mychart/templates/ 目录，会注意到一些文件已经存在了：
+ NOTES.txt: chart的"帮助文本"。这会在你的用户执行helm install时展示给他们。
+ deployment.yaml: 创建Kubernetes 工作负载的基本清单
+ service.yaml: 为你的工作负载创建一个 service终端基本清单。
+ _helpers.tpl: 放置可以通过chart复用的模板辅助对象

#### templates
第一个创建的模板是ConfigMap。Kubernetes中，配置映射只是用于存储配置数据的对象。其他组件，比如pod，可以访问配置映射中的数据。  
创建一个名为 `mychart/templates/configmap.yaml`的文件开始：
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```
然后执行`helm install full-coral ./mychart` 安装。
`helm get manifest` 命令后跟一个发布名称(full-coral),可以查看所有已经上传到server的Kubernetes资源。 每个文件以---开头表示YAML文件的开头，然后是自动生成的注释行，表示哪个模板文件生成了这个YAML文档。
使用 `helm get manifest full-coral` 可以查看我们安装的内容。

模板命令：
```
{{ .Release.Name }}
```
将发布名称注入了模板。值作为一个 **命名空间对象** 传给了模板，用点(.)分隔每个**命名空间**的元素。  
Release前面的点表示从作用域最顶层的命名空间开始（稍后会谈作用域）。这样.Release.Name就可解读为“通顶层命名空间开始查找 Release对象，然后在其中找Name对象”。  
Release是一个Helm的内置对象。稍后会更深入地讨论。但现在足够说明它可以显示从库中赋值的发布名称。  


当你想测试模板渲染的内容但又不想安装任何实际应用时，可以使用
```
helm install --debug --dry-run goodly-guppy ./mychart
```
这样不会安装应用(chart)到你的kubenetes集群中，只会渲染模板内容到控制台（用于测试）。

#### 内置对象
对象可以通过模板引擎传递到模板中。 当然你的代码也可以传递对象。（我们在使用with和range语句时，会看到示例）。有几种方式可以在模板中创建新对象，比如说我们后面会看到的tuple功能。

对象可以是非常简单的:仅有一个值。或者可以包含其他对象或方法。比如，Release对象可以包含其他对象（比如：Release.Name）和Files对象有一组方法。

在上一部分中，我们用`{{ .Release.Name }}`在模板中插入版本名称。Release是你可以在模板中访问的顶层对象之一。

1. Release： Release对象描述了版本发布本身。包含了以下对象：
    + Release.Name： release名称
    + Release.Namespace： 版本中包含的命名空间(如果manifest没有覆盖的话)
    + Release.IsUpgrade： 如果当前操作是升级或回滚的话，该值将被设置为true
    + Release.IsInstall： 如果当前操作是安装的话，该值将被设置为true
    + Release.Revision： 此次修订的版本号。安装时是1，每次升级或回滚都会自增
    + Release.Service： 该service用来渲染当前模板。Helm里始终Helm
2. Values：Values对象是从**values.yaml**文件和用户提供的文件传进模板的。默认为空
3. Chart： Chart.yaml文件内容。 Chart.yaml里的所有数据在这里都可以可访问的。比如 `{{ .Chart.Name }}-{{ .Chart.Version }}` 会打印出 mychart-0.1.0
4. Files： 在chart中提供访问所有的非特殊文件的对象。你不能使用它访问Template对象，只能访问其他文件。 请查看这个 文件访问部分了解更多信息
    + Files.Get 通过文件名获取文件的方法。 （.Files.Getconfig.ini）
    + Files.GetBytes 用字节数组代替字符串获取文件内容的方法。 对图片之类的文件很有用
    + Files.Glob 用给定的shell glob模式匹配文件名返回文件列表的方法
    + Files.Lines 逐行读取文件内容的方法。迭代文件中每一行时很有用
    + Files.AsSecrets 使用Base 64编码字符串返回文件体的方法
    + Files.AsConfig 使用YAML格式返回文件体的方法
5. Capabilities： 提供关于Kubernetes集群支持功能的信息
    + Capabilities.APIVersions 是一个版本列表
    + Capabilities.APIVersions.Has $version 说明集群中的版本 (比如,batch/v1) 或是资源 (比如, apps/v1/Deployment) 是否可用
    + Capabilities.KubeVersion 和Capabilities.KubeVersion.Version 是Kubernetes的版本号
    + Capabilities.KubeVersion.Major Kubernetes的主版本
    + Capabilities.KubeVersion.Minor Kubernetes的次版本
    + Capabilities.HelmVersion 包含Helm版本详细信息的对象，和 helm version 的输出一致
    + Capabilities.HelmVersion.Version 是当前Helm语义格式的版本
    + Capabilities.HelmVersion.GitCommit Helm的git sha1值
    + Capabilities.HelmVersion.GitTreeState 是Helm git树的状态
    + Capabilities.HelmVersion.GoVersion 是使用的Go编译器版本
6. Template： 包含当前被执行的当前模板信息
    + Template.Name: 当前模板的命名空间文件路径 (e.g. mychart/templates/mytemplate.yaml)
    + Template.BasePath: 当前chart模板目录的路径 (e.g. mychart/templates)

内置的值都是以大写字母开始。 这是符合Go的命名惯例。当你创建自己的名称时，可以按照团队约定自由设置。 就像很多你在 Artifact Hub 中看到的chart，其团队选择使用首字母小写将本地名称与内置对象区分开，本指南中我们也遵循该惯例。

#### Values 文件
在上一部分我们了解了Helm模板提供的内置对象。其中一个是Values对象。该对象提供了传递值到chart的方法，

其内容来自于多个位置：

+ chart中的values.yaml文件
+ 如果是子chart，就是父chart中的values.yaml文件
+ 使用-f参数(helm install -f myvals.yaml ./mychart)传递到 helm install 或 helm upgrade的values文件
+ 使用--set (比如helm install --set foo=bar ./mychart)传递的单个参数

以上列表有明确顺序：默认使用values.yaml，可以被父chart的values.yaml覆盖，继而被用户提供values文件覆盖， 最后会被--set参数覆盖，优先级为values.yaml最低，--set参数最高。

如果需要从默认的value中删除key，可以将key设置为null，Helm将在覆盖的value合并时删除这个key。
```
helm install stable/drupal --set image=my-registry/drupal:0.1.0 --set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null
```
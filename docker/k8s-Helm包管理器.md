## k8s Helm包管理器

简单地说，Helm是Kubernetes的一个软件包管理器。Helm相当于K8s的 `yum` 或 `apt` 。Helm部署 `Chart` ，你可以把它看成是一个打包的应用程序。它是你所有版本的、预配置的应用资源的集合，可以作为一个单元部署。然后你可以用不同的配置来部署另一个版本的 `Chart` 。

Helm `Chart` 是简单的Kubernetes YAML清单，结合成一个单一的包，可以宣传给你的Kubernetes集群。一旦打包，将Helm Chart 安装到你的集群中就像运行单一的Helm安装一样容易，这确实简化了容器化应用的部署。

Helm 有两个部分：
+ 客户端（CLI），安装在本地工作站上。
+ 服务器（Tiller），Kubernetes集群上，执行需要的东西。(Helm 3 已经完全移除Tiller) 
+ 
我们的想法是，你用CLI来推送你需要的资源，而Tiller将通过创建/更新/删除图表中的资源来确保该状态是真实的。为了完全掌握 Helm ，有3个概念我们需要熟悉一下：
+ `Chart` ：一个预配置的Kubernetes资源包。
+ `Release`：一个特定的 chart 部署实例，已经用Helm部署到集群上。
+ `Repository` ： 一组已发布的 Chart ，可以提供给其他人使用。


### 创建一个Helm Chart
chart是一个组织在文件目录中的集合。目录名称就是chart名称（没有版本信息）。因而描述WordPress的chart可以存储在wordpress/目录中。当你创建一个新的 Chart 时，Helm有一定的结构,。要创建，运行 `helm create YOUR-CHART-NAME` 。一旦创建完毕，目录结构应该是这样的：
```
your-chart-name/
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
+ Chart.yaml：这是把所有关于你要打包的图表的信息放在这里。所以，比如说，你的版本号，等等。这就是你要放所有这些细节的地方。
+ Values.yaml：这是你定义所有你想注入模板的值的地方。如果你熟悉terraform，可以把它看作是helms variable.tf文件。
+ Charts/：这是你存储你的图表所依赖的其他图表的地方。你可能会调用另一个图表，而你的图表需要正常运行。
+ templates/：这个文件夹用于存放您要部署的图表的实际清单。例如，你可能要部署一个nginx，需要一个服务、configmap和secrets。你将把你的deployment.yaml、service.yaml、config.yaml和secrets.yaml都放在模板目录下。它们都将从上面的values.yaml中获得其值。

### Helm 常见操作
1. 从 Artifact Hub 添加一个chart 仓库 ：`helm repo add bitnami https://charts.bitnami.com/bitnami`
2. 更新仓库 ：`helm repo update`
3. 搜索 Charts: `helm search repo bitnami`
4. 查看某个 Chart 的信息: `helm show [all] chart bitnami/mysql`
5. 如果你想下载和查看一个发布的chart，但不安装它： `helm pull chartrepo/chartname`
6. Helm可以通过多种途径查找和安装chart， 但最简单的是安装官方的bitnami charts: `helm install bitnami/mysql --generate-name`
   每当您执行 helm install 的时候，都会创建一个新的发布版本。 所以一个chart在同一个集群里面可以被安装多次，每一个都可以被独立的管理和升级。
7. 查看发布的列表： `helm list`
8. 卸载版本： `helm uninstall mysql-1612624192`
9. 帮助：`helm help` 或者在任意命令后添加 -h 选项：`helm get -h`
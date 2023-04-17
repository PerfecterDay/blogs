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


### 创建一个Helm图表
当你创建一个新的图表时，Helm有一定的结构。要创建，运行 `helm create YOUR-CHART-NAME` 。一旦创建完毕，目录结构应该是这样的：
```
your-chart-name/
|
|- .helmignore
|
|- Chart.yaml
|
|- values.yaml
|
|- charts/
|
|- templates/
```
+ .helmignore：这保存了打包图表时要忽略的所有文件。类似于.gitignore，如果你熟悉git的话。
+ Chart.yaml：这是把所有关于你要打包的图表的信息放在这里。所以，比如说，你的版本号，等等。这就是你要放所有这些细节的地方。
+ Values.yaml：这是你定义所有你想注入模板的值的地方。如果你熟悉terraform，可以把它看作是helms variable.tf文件。
+ Charts/：这是你存储你的图表所依赖的其他图表的地方。你可能会调用另一个图表，而你的图表需要正常运行。
+ templates/：这个文件夹用于存放您要部署的图表的实际清单。例如，你可能要部署一个nginx，需要一个服务、configmap和secrets。你将把你的deployment.yaml、service.yaml、config.yaml和secrets.yaml都放在模板目录下。它们都将从上面的values.yaml中获得其值。
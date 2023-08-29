## k8s Helm包管理器
{docsify-updated}

- [k8s Helm包管理器](#k8s-helm包管理器)
  - [Helm 常见操作](#helm-常见操作)

简单地说，Helm是Kubernetes的一个软件包管理器。Helm相当于K8s的 `yum` 或 `apt` 。Helm部署 `Chart` ，你可以把它看成是一个打包的应用程序。它是你所有版本的、预配置的应用资源的集合，可以作为一个单元部署。然后你可以用不同的配置来部署另一个版本的 `Chart` 。

Helm `Chart` 是简单的Kubernetes YAML清单，结合成一个单一的包，可以宣传给你的Kubernetes集群。一旦打包，将Helm Chart 安装到你的集群中就像运行单一的Helm安装一样容易，这确实简化了容器化应用的部署。

Helm 有两个部分：
+ 客户端（CLI），安装在本地工作站上。
+ 服务器（Tiller），Kubernetes集群上，执行需要的东西。(Helm 3 已经完全移除Tiller) 
+ 
我们的想法是，你用CLI来推送你需要的资源，而Tiller将通过创建/更新/删除图表中的资源来确保该状态是真实的。为了完全掌握 Helm ，有3个概念我们需要熟悉一下：
+ `Chart` ：一个预配置的Kubernetes资源包。
+ `Release`：是一个与特定配置相结合的chart的部署运行实例，已经用Helm部署到集群上。
+ `config` 包含了可以合并到打包的chart中的配置信息，用于创建一个可发布的对象。
+ `Repository` ： 一组已发布的 Chart ，可以提供给其他人使用。




### Helm 常见操作
1. 从 Artifact Hub 添加一个chart 仓库 ：`helm repo add bitnami https://charts.bitnami.com/bitnami`
2. 更新仓库 ：`helm repo update`
3. 搜索 Charts: `helm search repo bitnami`
4. 查看某个 Chart 的信息: `helm show [all] chart bitnami/mysql`
5. 如果你想下载和查看一个发布的chart，但不安装它： `helm pull chartrepo/chartname`,执行完会在当前目录看到对应的文件
6. 上传Chart到阿里云： `helm cm-push consul-1.1.1.tgz gtja`
7. Helm可以通过多种途径查找和安装chart， 但最简单的是安装官方的bitnami charts: `helm install bitnami/mysql --generate-name`
   每当您执行 helm install 的时候，都会创建一个新的发布版本。 所以一个chart在同一个集群里面可以被安装多次，每一个都可以被独立的管理和升级。
8. 查看发布的列表： `helm list`,`helm list -n namespace`
9. 卸载版本： `helm uninstall mysql-1612624192`
10. 帮助：`helm help` 或者在任意命令后添加 -h 选项：`helm get -h`
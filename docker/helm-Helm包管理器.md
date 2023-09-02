## Helm包管理器
{docsify-updated}

- [Helm包管理器](#helm包管理器)
  - [Helm的目标](#helm的目标)
  - [Helm 架构组件](#helm-架构组件)
  - [Helm 常见操作](#helm-常见操作)

简单地说，Helm是Kubernetes的一个软件包管理器。Helm相当于K8s的 `yum` 或 `apt` 。Helm部署 `Chart` ，你可以把它看成是一个打包的应用程序。它是你所有版本的、预配置的应用资源的集合，可以作为一个单元部署。然后你可以用不同的配置来部署另一个版本的 `Chart` 。

Helm `Chart` 是简单的Kubernetes YAML清单，结合成一个单一的包，可以宣传给你的Kubernetes集群。一旦打包，将Helm Chart 安装到你的集群中就像运行单一的Helm安装一样容易，这确实简化了容器化应用的部署。

### Helm的目标
Helm管理名为chart的Kubernetes包的工具。Helm可以做以下的事情：

+ 从头开始创建新的chart
+ 将chart打包成归档(tgz)文件
+ 与存储chart的仓库进行交互
+ 在现有的Kubernetes集群中安装和卸载chart
+ 管理与Helm一起安装的chart的发布周期

对于Helm，有三个重要的概念：
1. chart： 创建Kubernetes应用程序所必需的一组信息。
2. config： 包含了可以合并到打包的chart中的配置信息，用于创建一个可发布的对象。
3. release： 是一个与特定配置相结合的chart的运行实例。

### Helm 架构组件
Helm是一个可执行文件，执行时分成两个不同的部分：
1. Helm客户端: 是终端用户的命令行客户端。负责以下内容：

    + 本地chart开发
    + 管理仓库
    + 管理发布
    + 与Helm库建立接口
    + 发送安装的chart
    + 发送升级或卸载现有发布的请求

2. Helm库: 提供执行所有Helm操作的逻辑。与Kubernetes API服务交互并提供以下功能：
    + 结合chart和配置来构建版本
    + 将chart安装到Kubernetes中，并提供后续发布对象
    + 与Kubernetes交互升级和卸载chart
    + 独立的Helm库封装了Helm逻辑以便不同的客户端可以使用它。

Helm客户端和库是使用Go编程语言编写的，这个库使用Kubernetes客户端库与Kubernetes通信。现在，这个库使用REST+JSON。它将信息存储在Kubernetes的密钥中。 不需要自己的数据库。


### Helm 常见操作
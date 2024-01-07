## Helm包管理器
{docsify-updated}

- [Helm包管理器](#helm包管理器)
  - [Helm的目标](#helm的目标)
  - [Helm 架构组件](#helm-架构组件)
  - [Helm 安装](#helm-安装)

如果我们开发的是微服务架构的应用，组成应用的服务可能多达十个甚至几十上百个，这种组织和管理应用的方式就不好使了:
1. 很难管理、编辑和维护如此多的服务。每个服务都有若干配置，缺乏一个更高层次的工具将这些配置组织起来。
2. 不容易将这些服务作为一个整体统一发布。部署人员需要首先理解应用都包含哪些服务，然后按照逻辑顺序依次执行 kubectl apply ，即缺少一种工具来定义应用与服务，以及服务与服务之间的依赖关系。
3. 不能高效地共享和重用服务。比如两个应用都要用到 MySQL 服务，但配置的参数不一样，这两个应用只能分别复制一套标准的 MySQL 配置文件，修改后通过 kubectl apply部署。也就是说，不支持参数化配置和多环境部署。
4. 不支持应用级别的版本管理。虽然可以通过 kubectl rollout undo 进行回滚，但这只能针对单个 Deployment ，不支持整个应用的回滚。
5. 不支持对部署的应用状态进行验证。比如是否能通过预定义的账号访问 MySQL 。虽然 Kubernetes 有健康检查，但那是针对单个容器，我们需要应用（服务）级别的健康检查。
 
Helm 能够解决上面这些问题， Helm 帮助 Kubernetes 成为微服务架构应用理想的部署平台

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

### Helm 安装
1. curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
2. chmod 700 get_helm.sh
3. ./get_helm.sh

# docker compose
{docsify-updated}



我们知道使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

Compose 中有两个重要的概念：

+ 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
+ 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker compose.yml 文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。


```
identidock: ➊  
  build: .  ➋ 
ports:    ➌ 
 - "5000:5000"  
environment: ➍ 
  ENV: DEV  
volumes: ➎ 
  - ./app:/app
```

+ ➊ 第一行声明构建的容器名称。一个 YAML 文件中可以定义多个容器（在 Compose 的术语中称为服务）。
+ ➋ 这 里 的 build 关 键 字 告 诉 Compose， 这 个 容 器 的 镜 像 是 通 过 当 前 目 录（.） 下的Dockerﬁle 构建。每个容器的定义必须包括一个 build 或 image 关键字。image 关键字的值，是用于启动容器的镜像的标签或 ID，与 docker run 命令的参数相同。
+ ➌ ports 关键字相当于 docker run 命令的 -p 参数，用于声明对外开放的端口。这里我们把容器的 5000 端口映射到主机的 5000 端口。列出端口时可以不带引号，但最好避免这样做，因为当遇到像 56:56 这种值的时候，YAML 会把它解析为以 60 为基数的六十进制数字。
+ ➍ environment 关键字相当于 docker run 命令的 -e 参数，用来设置容器的环境变量。为了运行用于开发的 Flask Web 服务器，这里我们把 ENV 变量设成 DEV 。
+ ➎ volumes 关键字相当于 docker run 的 -v 参数，用于配置数据卷。这里与之前的做法一样，将 app 目录通过绑定挂载的方式挂载到容器，以便让我们能够从主机修改代码。


下面是使用 Compose 时常用的命令。大多数命令都不言自明，而且 Docker 也有同样名字的命令，不过还是值得在这里提一下。

#### up
启动所有在 Compose 文件中定义的容器，并且把它们的日志信息汇集一起。通常会使用 -d 参数使 Compose 在后台运行。

#### build
重新建造由 Dockerﬁle 构建的镜像。除非镜像不存在，否则 up 命令不会执行构建的动作，因此需要更新镜像时便使用这个命令。

#### ps
获取由 Compose 管理的容器的状态信息。

#### run
启动一个容器，并运行一个一次性的命令。被连接的容器会同时启动，除非用了 --no-deps 参数。

#### logs
汇集由 Compose 管理的容器的日志，并以彩色输出。

#### stop
停止容器，但不会删除它们。

#### rm
删除已停止的容器。不要忘记使用 -v 参数来删除任何由 Docker 管理的数据卷。

一个普通的工作流程以 `docker compose up -d` 命令启动应用程序开始。`docker compose logs` 和 `ps` 命令可以用来验证应用程序的状态，还能帮助调试。修改代码后，先执行 `docker compose build` 构建新的镜像，然后执行 `docker compose up -d` 取代运行中的容器。注意，Compose 会保留原来容器中所有旧的数据卷，这意味着即使容器更新后，数据库和缓存也依旧在容器内（这很可能会造成混淆，因此要特别小心）。如果你修改了 Compose 的 YAML 文件，但不需要构建新镜像，可以通过 `up -d` 参数使Compose 以新的配置替换容器。如果想要强制停止 Compose 并重新创建所有容器，可以使用 `--force-recreate` 选项来达到目的。

当你不再需要使用该应用时，可以执行 `docker compose stop` 来停止应用程序。假设代码没有变更，可以通过 `docker compose start` 或 `up` 来重启相同的容器。使用 `docker compose rm` 彻底把容器删除。
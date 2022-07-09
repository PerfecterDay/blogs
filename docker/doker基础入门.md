# docker 入门
{docsify-updated}

#### docker 基本概念
1. Docker客户端与服务器端：Docker是 C/S 架构的，
2. 镜像  
    Docker 镜像 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 不包含 任何动态数据，其内容在构建之后也不会被改变。对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 root 文件系统。
    因为镜像包含操作系统完整的 root 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 Union FS 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。
    镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。
    分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。
3. 容器  
    镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
    容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。
    前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 容器存储层。
    容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。
    按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者 绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。
    数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。
4. Registry：相当于 GitHub，用来存储、发布镜像的仓库。
5. 如何确定自己是否是登录到一台物理机/虚拟机/容器shell中： `systemd-detect-virt -c`

#### 镜像操作
1. 获取镜像 
   + 在 Registry 仓库中查找镜像: `docker search xxx`
   + 从 Registry 下载镜像： `docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]`
   + 利用 Dockerfile 来制作镜像:  `docker build -t [<仓库名>:<标签>] 镜像构建上下文路径`
   + 利用指定的 Dockerfile 来制作镜像: `docker build -f path/to/dockerfile -t [<仓库名>:<标签>] 镜像构建上下文路径`
2. 查看本地镜像 
    + 查看镜像： `docker images(docker image ls)`
    + 查看所有镜像： `docker image ls -a`
    + 查看指定镜像： `docker image ls <image_name>`
    + 查看指定镜像的详细信息: `docker image inspect <image_name>`
3. 删除镜像 
    + 删除镜像：`docker image rm [选项] <镜像1< [<镜像2< ...]`
    + 删除虚悬镜像(dangling image): ``docker image prune`
4. 上传镜像 
    + 上传到 Registry ，默认dockerHub：`docker push <image_name>`

##### Dockerfile 镜像指令
当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。  
Dockerfile 中每一个指令都会建立一层，每一条指令执行过程都类似于：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。
1. `FROM`: 指定基础镜像 ，后续指令都是基于基础镜像的
2. `CMD` ：容器就是进程，既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。如果启动容器时，添加了指令，则会覆盖CMD指令
    `CMD ['/bin/bash','-l']`
3. `ENTRYPOINT` ： 类似于CMD，但是启动容器时添加的指令将会作为参数传递到ENTRYPOINT的指令中
4. `WORKDIR` ：在容器内部切换工作目录，CMD和ENTRYPOINT的指令就运行在这个目录中
5. `ENV` : 用来在构建镜像过程中设置环境变量，这个环境变量可以在后续RUN指令中使用
6. `USER` ：用来指定该镜像会以什么用户运行
7. `VOLUME` ：为基于镜像创建的容器添加卷。
8. `ADD` : 用来将构建环境下的文件和目录复制到镜像中，不能对构建目录或者上下文之外的文件进行ADD操作。ADD归档文件时会自动解压
    `ADD hello.jar /opt/application/hello.jar`
9. `COPY` ： 类似于ADD，区别在于不会做文件提取和解压的工作。
10. `RUN` : 用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。
11. `EXPOSE <端口1> [<端口2>...]` : 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

#### 容器操作
简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。

##### 创建容器
1. 创建并启动容器执行 command 命令 `docker run [options] image_name [command]` ，容器是否长久运行和 command 命令有关，相关选项:
    + `-d` :代表后台守护进程运行
    + `-p 宿主机:宿主机端口：容器端口`：将容器端口映射到宿主机端口
    + `--restart=always`：自动重启容器
    + `--name=yourname`：指定容器名字
    + `-v 宿主机目录：容器目录：ro|rw`： 将宿主机的目录作为卷，挂载到容器对应目录
    + `-e, --env=[]`: 设置环境变量  
示例：`docker run -p 3316:3306 -e MYSQL_ROOT_PASSWORD=123456 -d --name=mysql-master  mysql`
2. 创建一个新的容器但不启动它： `docker create [options] image_name [command]` 示例：`docker create  --name myrunoob  nginx:latest`

##### 启动/停止容器
1. 创建并启动容器: `docker run`
2. 重启已经停止的容器: `docker start container_id`
3. 停止容器: `docker stop container_id` 
4. 重启容器: `docker restart container_id|tag`，  示例-`docker restart master slave`

##### 容器查看/删除
1. 查看正在运行的容器: `docker ps`
2. 查看容器（默认指显示正在运行的容器，与ps一样）， -a 选项查看所有容器: `docker container ls`
3. 查看指定容器的详细信息: `docker inspect container_id`
4. 查看镜像、容器、数据卷所占用的空间: `docker system df`
5. 删除指定容器: `docker container rm container_id`
6. 清理所有处于终止状态的容器: `docker container prune` 

##### 其它容器操作
1. 查看容器的运行日志: `docker logs container_id`
2. 查看容器内的进程: `docker top container_id`
3. 进入容器运行指定命令: `docker exec -it container_id command` 
4. 复制宿主机文件到容器目录：`docker cp slave.cnf mysql-slave:/etc/mysql/conf.d`

#### 网络管理
1. 新建网络：`docker network create -d bridge my-net`
2. 创建容器的时候将容器加入到指定网络： `docker create|run --network my-net [options] image_name [command]`
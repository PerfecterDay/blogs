## Docker 镜像
{docsify-updated}

- [Docker 镜像](#docker-镜像)
  - [镜像的分层结构](#镜像的分层结构)
  - [镜像的构建](#镜像的构建)
    - [Dockerfile 常用指令](#dockerfile-常用指令)
    - [镜像操作](#镜像操作)


Docker 发明的初衷就是为了方便软件的部署，通常我们开发完一款服务，部署的时候需要在服务器上安装运行环境、依赖库、配置环境等等，这些工作在不同的操作系统或者不同的系统版本上都要求部署人员具有相关的知识。而且可能开发环境和部署环境的不同，导致服务在部署环境的行为会有差异。这些种种原因就驱使人们想研究出一种能屏蔽这些差异的工具，Docker 就是这样一款工具。

Docker 容器底层使用的是 Host 主机的内核，自己**通过镜像为 kernel 提供 rootfs** ，rootfs 就是用户空间的文件系统。事实上，我们每次部署发布服务，就是为我们的服务配置提供一个运行时的环境，而这本质上与提供一个 rootfs 没有差别。所以 Docker 镜像是为服务提供/配置运行环境的。

理论上， 一个容器镜像能运行在任何一个运行Docker的机器上。 但有一个小警告：一个关于运行在一台机器上的所有容器共享主机Linux内核的警告。 如果一个容器化的应用需要一个特定的内核版本， 那它可能不能在每台机器上都工作。 如果一台机器上运行了一个不匹配的Linux内核版本， 或者没有相同内核模块可用，那么此应用就不能在其上运行。虽然容器相比虚拟机轻量许多， 但也给运行于其中的应用带来了一些局限性。虚拟机没有这些局限性， 因为每个虚拟机都运行自己的内核。还不仅是内核的问题。 一个在特定硬件架构之上编译的容器化应用， 只能在有相同硬件架构的机器上运行。 不能将一个x86架构编译的应用容器化后， 又期望它能运行在ARM架构的机器上。 你仍然需要一台虚拟机来做这件事情。

### 镜像的分层结构
Docker 通过扩展现有镜像，创建新的镜像。特殊情况下，基于空白镜像 scratch 创建。
事实上，Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的。 base 镜像通常不依赖其它镜像，从 scratch 构建出来。

镜像构建时，通常每安装一个软件或者添加文件，就会在现有镜像基础上增加一层。
<center>
<img src="pics/image-layer.png" width="30%" style="inline"> 
</center>

这种分层机制最大的好处就是能够**共享资源**。 比如大多数镜像都从相同的 base 镜像构建出来，这样 base 镜像只要保存一份 base 镜像即可；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务。

如果多个容器共享同一份基础镜像，当某个容器修改了基础镜像的内容，比如/etc 目录下的某个文件，这时其它容器 /etc 目录下的文件是否会改变 ？
答案是不会！修改会被限制在单个容器内。

**容器启动时，会在镜像层的顶部创建一个可写的容器层。所有对容器的更改，无论是添加、删除、修改文件都只会发生在容器层中。只有容器层是可写的，底下的镜像层都是只读的。**

<center>
<img src="pics/sharing-layers.jpg" width="50%">
</center>

镜像层的数量可能会很多，所有的镜像层会联合起来组成一个统一的文件系统。如果不同层提供了相同路径的文件，比如/a.txt，那么上层的文件会覆盖下层的文件。
+ 添加文件：容器创建文件时，新文件被添加到容器层中
+ 读取文件：容器在读取文件时，会从上往下依次在各层中查找此文件，一旦找到就打开并读入内存。
+ 修改文件：在容器中修改文件时，会从上往下依次在各层中查找此文件，一旦找到立即复制到容器层然后修改它。
+ 删除文件：在容器中修改文件时，会从上往下依次在各层中查找此文件，一旦找到会在容器层记录下此删除操作。

**只有在修改时才复制一份数据，这种特性叫做 copy-on-write 。容器层保存的是镜像变化的部分，不会对底下的镜像层进行任何修改。**

### 镜像的构建
1. docker commit  
	```docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]```使用docker commit 创建镜像一般包含三个步骤：
    1. 运行容器
    2. 修改容器
    3. 使用 `docker commit` 将容器保存为新的镜像
   
	这种方式创建的镜像无法审计，使用者不知道镜像如何创建出来的，里面是否有恶意程序，存在安全隐患。

2. Dockerfile  
	Docker 可以根据 Dockerfile 的指令为我们构建镜像，当我们编写完 Dockerfile 后，可以执行 `docker build -t [imagename] [buildcontext]`。`buildcontext` 路径下的所有文件会被发送给 Docker daemon 服务器，这样当我们使用如 `ADD`/`COPY` 等命令时，才能找到相应的要添加的文件。Docker 使用 Dockerfile 构件镜像的过程如下：
	1. 从base 镜像运行一个临时容器
	2. 执行一条指令对容器进行修改
	3. 执行类似 docker commit 的操作，生成一个新的镜像层
	4. 再基于刚刚提交的镜像运行一个新的临时容器
	5. 重复2～4步，直到 Dockerfile 中所有的指令执行完毕

#### Dockerfile 常用指令
1. `FROM`: 指定基础镜像 ，后续指令都是基于基础镜像的
2. `CMD` ：容器就是进程，既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。如果启动容器时，添加了指令，则会覆盖CMD指令。可以有多个CMD指令，但是只有最后一个生效：`CMD ['/bin/bash','-l']`
3. `ENTRYPOINT` ： 类似于CMD，但是启动容器时添加的指令将会作为参数传递到ENTRYPOINT指定的指令中，可以有多个，但是只有最后一个生效
4. `WORKDIR` ：指定容器内部的工作目录，后续的CMD和ENTRYPOINT等指令会运行在这个目录中
5. `ENV` : 用来在构建镜像过程中设置环境变量，这个环境变量可以在后续RUN指令中使用
6. `USER` ：用来指定制作该镜像的时候以什么用户运行，**这并不是运行容器时的用户**
7. `VOLUME` ：为基于镜像创建的容器添加卷。
8. `ADD` : 用来将构建环境下的文件和目录复制到镜像中，不能对构建目录或者上下文之外的文件进行ADD操作。ADD归档文件时会自动解压文件
    `ADD hello.jar /opt/application/hello.jar`
9. `COPY` ： 类似于ADD，区别在于不会做文件提取和解压的工作。
10. `RUN` : 用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。
11. `EXPOSE <端口1> [<端口2>...]` : 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主机进行端口映射。

#### 镜像操作
1. 获取镜像 
   + 在 Registry 仓库中查找镜像: `docker search xxx`
   + 从 Registry 下载镜像： `docker pull [选项] [Docker Registry 地址[:端口号]/]repository[:tag]`
   + 拉取指定平台的镜像： `docker pull envoyproxy/envoy:v1.23-latest --platform linux/amd64`
   + 利用 Dockerfile 来制作镜像:  `docker build -t [<repository>:<tag>] 镜像构建上下文路径`
   + 利用指定的 Dockerfile 来制作镜像: `docker build -f path/to/dockerfile -t [<repository>:<tag>] 镜像构建上下文路径`
2. 查看本地镜像 
    + 查看镜像： `docker images(docker image ls)`
    + 查看所有镜像： `docker image ls -a`
    + 查看指定镜像： `docker image ls <image_name>`
    + 查看指定镜像的详细信息: `docker image inspect <image_name>`
3. 删除镜像 
    + 删除镜像：`docker image rm [选项] <镜像1< [<镜像2< ...]` 或者更短的 `docker rmi <imgid>`
    + 删除虚悬镜像(dangling image): `docker image prune`
    + 删除所有镜像： `docker rmi -f $(docker images -aq)`
4. 上传镜像 
    + 上传到 Registry ，默认dockerHub：`docker push user_name/<repository>:tag`
5. 导入/导出镜像  
   + `docker save [imgId] -o [imgfile]`
   + `docker load -i [imgfile]`  
	这种导出的镜像是没有镜像名字的，需要手动修改导入的镜像名字：`docker tag [imgId] [<repository>:<tag>]`
6. 查看 docker 空间占用
   + `docker system df`
   + `docker system prune -a --volumes`

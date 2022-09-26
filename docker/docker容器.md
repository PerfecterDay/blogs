# docker 容器
{docsify-updated}

### 容器操作的命令

1. 创建容器  
	1. 创建并启动容器执行 command 命令 `docker run [options] image_name [command]` ，实际上是 `docker create` 和 `docker start` 的组合，容器是否**长久运行和 command 命令有关,只要命令不结束，容器也不会退出（exit 状态）**，相关选项:
		+ `-d` :代表后台守护进程运行
		+ `-p 宿主机IP:宿主机端口：容器端口`：将容器端口映射到宿主机端口
		+ `--restart=always`：自动重启容器
		+ `--name=yourname`：指定容器名字
		+ `-v 宿主机目录(文件)：容器目录（文件）：ro|rw`： 将宿主机的目录或文件作为卷，挂载到容器对应目录或文件
		+ `-e, --env=[]`: 设置环境变量  
	示例：`docker run -p 3316:3306 -e MYSQL_ROOT_PASSWORD=123456 -d --name=mysql-master  mysql`
	1. 创建一个新的容器但不启动它： `docker create [options] image_name [command]` 示例：`docker create  --name myrunoob  nginx:latest`

2. 查看容器  
	2. 查看正在运行的容器: `docker ps` 或者 `docker container ls`
	3. 查看所有容器， `-a` 选项: `docker ps -a` 或者`docker container ls -a `
	4. 查看指定容器的详细信息: `docker inspect container_id`
	5. 查看镜像、容器、数据卷所占用的空间: `docker system df`
	6. 删除指定容器: `docker container rm container_id` 或者 `docker rm container_id`
	7. 清理所有处于终止状态的容器: `docker container prune` 

3. 启动/停止容器  
	8. 创建并启动容器: `docker run`
	9. 重启已经停止的容器: `docker start container_id`
	10. 停止容器: `docker stop container_id` 
	11. 重启容器: `docker restart container_id|tag`，  示例-`docker restart master slave`

4. 进入容器
	12. `docker attach [containerid]`
	13. `docker exec -it [containerid]` 退出按 Ctrl+p后再按 Ctrl+q
	14. 容器与宿主机之间拷贝文件：`docker cp ~/Documents/gtja/com_gmas/com_gmas.tar.gz ubuntu:/home`,`docker cp ubuntu:/home/test.txt ~/Documents/gtja/com_gmas/`

5. 查看容器运行日志
	1. `docker logs -f`:持续输出docker 日志
### 实现容器的底层技术
cgroup 和 namespace 是最重要的两种技术——cgroup 实现资源限额，namespace 实现资源隔离。

1. cgroup : 提供资源分配与管理功能。
2. namespace : 提供环境隔离功能。
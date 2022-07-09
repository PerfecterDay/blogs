# mysql高可用-主从复制
{docsify-updated}


### 复制原理
复制是 mysql 数据库提供的一种高可用高性能解决方案，分为3个步骤：
1. 主服务器把数据更改（写）记录到二进制日志文件中，
2. 从服务器把主服务器的二进制日志复制到自己的中继日志中（slave IO 线程）
3. 从服务器重做中继日志中的日志，把更改应用到本地数据库上，以达到数据的最终一致性。

![mysql 复制原理](../../pics/mysql-replacation.jpg)

一般要设置 slave 机器：
```mysql> CHANGE MASTER TO
    ->     MASTER_HOST='master_host_name',
    ->     MASTER_USER='replication_user_name',
    ->     MASTER_PASSWORD='replication_password',
    ->     MASTER_LOG_FILE='recorded_log_file_name',
    ->     MASTER_LOG_POS=recorded_log_position;```
然后哦开启 slave 线程：`start slave`.

使用 `show master status` 或 `show slave status` 可以分别查看主/从服务器上的复制状态。
![slave status 状态说明](../pics/slave-status.jpg)

### Docker主从复制实战
1. 下载 mysql 的docker 镜像：`docker pull mysql`
2. 启动主mysql：`docker run -p 3316:3306 -e MYSQL_ROOT_PASSWORD=123456 -d --name=mysql-master  mysql`
3. 启动从mysql：`docker run -p 3326:3306 -e MYSQL_ROOT_PASSWORD=123456 -d --name=mysql-slave  mysql`
4. 进入docker容器，允许 root 任意IP登录： `update mysql.user set host='%' where user='root';`,然后 `flush privileges`
5. 宿主机连接mysql测试：`mysql -hlocalhost --protocol=TCP -P3316 -umaster -p`，切记要加上 --protocol=TCP 选项
6. 宿主机新建主服务器配置文件 master.cnf:
    ```[mysqld]
    server-id = 1
    log-bin=mysql-bin
    binlog-ignore-db=mysql
    # binlog-do-db
    # binlog_format=mixed```
7. 复制 master.cnf 到 mysql-master 的 /etc/mysql/conf.d目录下：  
    `docker cp master.cnf mysql-master:/etc/mysql/conf.d`
8. 宿主机新建主服务器配置文件 slave.cnf:
    ```[mysqld]
    server-id = 2
    # relay_log = relay_bin
    # relay-log-index = relay-bin.index```
9. 复制 slave.cnf 到 mysql-slave 的 /etc/mysql/conf.d目录下：  
    `docker cp slave.cnf mysql-slave:/etc/mysql/conf.d`
10. 重启主从服务器：`docker restart mysql-master mysql-slave`
11. 进入 mysql-master 主容器，创建同步账号并授权： 
    + 创建同步账号：`create user repl@'%' identified by 'repl';`
    + mysql8 可能需要修改密码认证方式：`ALTER USER 'repl'@'%' IDENTIFIED WITH mysql_native_password;`
    + 授权：`GRANT REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO repl@'%';`
    + 查看同步状态：`show master status;`
    + 记下容器 IP：`hostname -I` -> 假设为 172.17.0.2
12. 进入 mysql-slave 从容器，配置 slave：
    + 配置同步账号信息：`CHANGE MASTER TO MASTER_HOST='172.17.0.2', MASTER_USER='repl', MASTER_PASSWORD='repl';`
    + 启动同步：`start slave;`
    + 查看同步状态：`show slave status\G;`

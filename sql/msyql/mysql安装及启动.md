# mysql的安装及配置
{docsify-updated}

# windwos 下 mysql 的安装及配置

## mysql 安装
1. 下载zip包，解压到某个目录
2. 解压目录下创建my.ini文件，配置basedir和datadir路径
   ```
    [mysqld]
    # set basedir to your installation path
    basedir=C:/Program Files (x86)/mysql-8.0.21-winx64
    # set datadir to the location of your data directory
    datadir=D:/mysqldata/data
    ```
3. 执行 mysqld --initialize --console --user=root ,初始化数据库，会在控制台打印出root账户的随机密码。必须保证配置的 data 目录是空的，否则会报错。
4. 启动mysqld服务：mysqld
5. 停止mysqld服务：`mysqladmin -u root shutdown`，如果要密码加上-p
6. 执行mysql -hlocalhost -uroot -p连接数据库
7. 首次连接数据库后要更改初始化密码：ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';否则不能执行sql语句
8. 允许 root 从任意IP登录：`update mysql.user set host='%' where user='root';`,然后 `flush privileges`

## mysql 安装为服务
1. `mysqld --install-manual MySQL --defaults-file="D:\applications\Scoop\apps\mysql\current\my.ini"` 安装为手动启动的服务，`mysqld --install MySQL --defaults-file="D:\applications\Scoop\apps\mysql\current\my.ini"` 安装位自动启动的服务
2. 启动：`sc start mysql`，停止：`sc stop mysql`
2. 卸载服务:`sc delete MySQL`

### Linux Redhat 8  下安装 mysql
Mysql官网下载以下RPM包：
```
mysql-community-client-8.0.33-1.el8.x86_64.rpm
mysql-community-client-plugins-8.0.33-1.el8.x86_64.rpm
mysql-community-common-8.0.33-1.el8.x86_64.rpm
mysql-community-icu-data-files-8.0.33-1.el8.x86_64.rpm
mysql-community-libs-8.0.33-1.el8.x86_64.rpm
mysql-community-server-8.0.33-1.el8.x86_64.rpm
```
执行 `yum localinstall [rpm包]` 命令，提示缺少啥依赖包，就先装依赖。一半顺序如下：
```
yum localinstall mysql-community-common-8.0.33-1.el8.x86_64.rpm -y &&\
yum localinstall mysql-community-client-plugins-8.0.33-1.el8.x86_64.rpm -y &&\
yum localinstall mysql-community-libs-8.0.33-1.el8.x86_64.rpm -y &&\
yum localinstall mysql-community-client-8.0.33-1.el8.x86_64.rpm -y &&\
yum localinstall mysql-community-icu-data-files-8.0.33-1.el8.x86_64.rpm -y &&\
yum localinstall mysql-community-server-8.0.33-1.el8.x86_64.rpm -y
```


`systemctl start mysqld` 启动服务。
`cat /var/log/mysqld.log | grep temporary`: 查询首次安装的登录账号和密码
`ALTER USER 'root'@'%' IDENTIFIED BY 'Jhqqt0711!';` //修改密码
`update mysql.user set host='%' where user='root';` //修改允许登录的地址
`CREATE USER 'app'@'%' IDENTIFIED BY 'Jhqqt0711!'` //创建一个app用户
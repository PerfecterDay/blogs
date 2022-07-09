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

# linux 下安装 mysql
    shell> groupadd mysql
    shell> useradd -r -g mysql -s /bin/false mysql
    shell> cd /usr/local
    shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
    shell> ln -s full-path-to-mysql-VERSION-OS mysql
    shell> cd mysql
    shell> mkdir mysql-files
    shell> chown mysql:mysql mysql-files
    shell> chmod 750 mysql-files
    shell> bin/mysqld --initialize --user=mysql
    shell> bin/mysql_ssl_rsa_setup
    shell> bin/mysqld_safe --user=mysql &
    # Next command is optional
    shell> cp support-files/mysql.server /etc/init.d/mysql.server

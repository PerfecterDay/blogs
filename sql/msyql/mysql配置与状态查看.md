# Mysql配置与状态查看
{docsify-updated}

## 命令行上使用启动选项

`mysqld_safe --启动选项1[=值1] --启动选项2[=值2] ... --启动选项n[=值n]`

可以将各个启动选项写到一行中，各个启动选项之间使用空白字符隔开，在每一个启动选项名称前边添加-- 。对于不需要值的启动选项，比方说 skip-networking ，它们就不需要指定对应的值。对于需要指定值的启动选项，比如 default-storage-engine 我们在指定这个设置项的时候需要显式的指定它的值，比方说InnoDB 、 MyISAM 啦什么的～ 在命令行上指定有值的启动选项时需要注意，选项名、=、选项值之间不可以有空白字符。

每个MySQL程序都有许多不同的选项。大多数程序提供了一个 `--help` 选项，你可以查看该程序支持的全部启动选项以及它们的默认值。

+ `mysql --help` 可以看到 `mysql` 客户端程序支持的启动选项
+ `mysqld_safe --help` 可以看到 `mysqld_safe` 程序支持的启动选项。
+ `mysqld --verbose --help` 可以看到 `mysqld` 程序支持的启动选项。
+ `mysql.server --help` 可以看到 `mysql.server` 程序支持的启动选项。

## 配置文件
在命令行中设置启动选项只对当次启动生效，也就是说如果下一次重启程序的时候我们还想保留这些启动选项的话，还得重复把这些选项写到启动命令行中，这样很麻烦。把需要设置的启动选项都写在这个配置文件中，每次启动服务器的时候都从这个文件里加载相应的启动选项。由于这个配置文件可以长久的保存在计算机的硬盘里，所以只需我们配置一次，以后就都不用显式的把启动选项都写在启动命令行中了，所以我们推荐使用配置文件的方式来设置启动选项。

### 配置文件的路径
以 macos homebrew 安装的方式为例，相应的程序会在一下路径查找配置文件：

+ /etc/my.cnf
+ /etc/mysql/my.cnf
+ /opt/homebrew/etc/my.cnf
+ ~/.my.cnf

我们可以使用 `mysql --help | hrep my.cnf` 命令来查看 mysql 查找配置文件的路径。

### 配置文件的内容

与在命令行中指定启动选项不同的是，配置文件中的启动选项被划分为若干个组，每个组有一个组名，用中括号[] 扩起来：

```
[server]
option1 #这是option1，该选项不需要选项值
option2 = value2 #这是option2，该选项需要选项值
...

[mysqld]
option1 #这是option1，该选项不需要选项值
option2 = value2 #这是option2，该选项需要选项值
....

[mysqld_safe]
option1 #这是option1，该选项不需要选项值
option2 = value2 #这是option2，该选项需要选项值
...

[client]
option1 #这是option1，该选项不需要选项值
option2 = value2 #这是option2，该选项需要选项值
...

[mysql]
option1 #这是option1，该选项不需要选项值
option2 = value2 #这是option2，该选项需要选项值
...

[mysqladmin]
option1 #这是option1，该选项不需要选项值
option2 = value2 #这是option2，该选项需要选项值
...
```
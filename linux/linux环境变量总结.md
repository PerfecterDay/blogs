# Linux环境变量总结
{docsify-updated}

转自： https://www.jianshu.com/p/ac2bc0ad3d74
## Linux环境变量分类
一、按照生命周期来分，Linux环境变量可以分为两类：
1. 永久的：需要用户修改相关的配置文件，变量永久生效。
2. 临时的：用户利用export命令，在当前终端下声明环境变量，关闭Shell终端失效。

二、按照作用域来分，Linux环境变量可以分为：
1. 系统环境变量：系统环境变量对该系统中所有用户都有效。
2. 用户环境变量：顾名思义，这种类型的环境变量只对特定的用户有效。

## Linux设置环境变量的方法
一、在/etc/profile文件中添加变量 对所有用户生效（永久的）
用vim在文件/etc/profile文件中增加变量，该变量将会对Linux下所有用户有效，并且是“永久的”。修改文件后要想马上生效还要运行`source /etc/profile` 不然只能在下次重启时生效。

二、在用户目录下的 `.bash_profile` 文件中增加变量 【对单一用户生效（永久的）】
用 `vim ~/.bash_profile` 文件中增加变量，改变量仅会对当前用户有效，并且是“永久的”。修改文件后要想马上生效还要运行 `source ~/.bash_profile` 不然只能在下次重进此用户时生效。

三、直接运行 `export` 命令定义变量 【只对当前shell（BASH）有效（临时的）】
在shell的命令行下直接使用 `export 变量名=变量值`
定义变量，该变量只在当前的shell（BASH）或其子shell（BASH）下是有效的，shell关闭了，变量也就失效了，再打开新shell时就没有这个变量，需要使用的话还需要重新定义。

## Linux环境变量使用
一、Linux中常见的环境变量有：
+ PATH：指定命令的搜索路径
+ HOME：指定用户的主工作目录（即用户登陆到Linux系统中时，默认的目录）。
+ HISTSIZE：指保存历史命令记录的条数。
+ LOGNAME：指当前用户的登录名。
+ HOSTNAME：指主机的名称，许多应用程序如果要用到主机名的话，通常是从这个环境变量中来取得的
+ SHELL：指当前用户用的是哪种Shell。
+ LANG/LANGUGE：和语言相关的环境变量，使用多种语言的用户可以修改此环境变量。
+ MAIL：指当前用户的邮件存放目录。

二、Linux也提供了修改和查看环境变量的命令，下面通过几个实例来说明：
+ echo         显示某个环境变量值 echo $PATH
+ export   设置一个新的环境变量 export HELLO="hello" (可以无引号)
+ env      显示所有环境变量
+ set      显示本地定义的shell变量  
+ unset        清除环境变量 unset HELLO
+ readonly     设置只读环境变量 readonly HELLO

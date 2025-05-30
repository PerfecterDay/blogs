#  Systemd
{docsify-updated}

> https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html  
> https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html


1. 创建一个Systemd服务单元文件,即在 `/etc/systemd/system/` 目录下创建一个以 `gtja-thrift-proxy.service` 的文件
2. 修改 `/root/gtja-thrift-proxy/start.sh` ：加上 `cd /root/gtja-thrift-proxy` 。 systemd 默认的执行路径不是脚本所在的路径， `java -jar xxx.jar` 会找不到
3. 
```
[Unit]
Description=gtja-thrift-proxy

[Service]
Type=forking
ExecStart=/bin/bash /root/gtja-thrift-proxy./start.sh start 8080 prd5
Restart=always

[Install]
WantedBy=multi-user.target
```
4. `systemctl enable gtja-thrift-proxy`
5. `systemctl start gtja-thrift-proxy`
6. `systemctl status gtja-thrift-proxy`
7. 修改了 service 文件之后，要重载： `systemctl daemon-reload`
8. `systemctl restart gtja-thrift-proxy` ： 重启服务
9. `systemctl cat <service-name>/systemctl list-unit-files | grep <service-name>` ： 查看服务定义文件的目录及内容，比如我要查找 mysql 自启动服务的描述文件，可以 `systemctl cat mysqld`


查看日志： `journalctl -xe`
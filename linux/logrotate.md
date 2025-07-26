# logrotate
{docsify-updated}


## 命令行语法
```
logrotate [-dv] [-f|--force] [-s|--state file] config_file ..
```

+ `-d, --debug` : 开启调试模式，默认会使用 `-v`。在调试模式下，不会更改日志或 `logrotate` 状态文件。
+ `-f, --force` : 告诉 logrotate 强制轮换，即使它认为没有必要。有时，在向 logrotate 配置文件添加新条目后，或在手动删除旧日志文件后，这很有用，因为新文件将被创建，日志记录将继续正常进行。
+ `-s, --state <statefile>` : 告诉 logrotate 使用另一个状态文件。这在 logrotate 以不同用户身份运行不同日志文件集时非常有用。默认状态文件是 `/var/lib/logrotate.status` 。
+ `--usage` : 打印使用帮助
+ `--?, --help` : 打开帮助
+ `-v, --verbose` : 打开 verbose 

## 配置文件
`/etc/logrotate.conf`

```
# see "man logrotate" for details
# rotate log files weekly/daily/monthly/yearly
weekly

# use the syslog group by default, since this is the owning group
# of /var/log/syslog.
su root syslog

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# uncomment this if you want your log files compressed
compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

/var/log/messages {
    rotate 5
    weekly
    postrotate
        /usr/bin/killall -HUP syslogd
    endscript
}

"/var/log/httpd/access.log" /var/log/httpd/error.log {
    rotate 5
    mail www@my.org
    size 100k
    sharedscripts
    postrotate
        /usr/bin/killall -HUP httpd
    endscript
}

/var/log/news/* {
    monthly
    rotate 2
    olddir /var/log/news/old
    missingok
    postrotate
        kill -HUP 'cat /var/run/inn.pid'
    endscript
    nocompress
}
```


logrotate 需要状态文件控制频率（比如 daily）。默认，logrotate 会使用 `/var/lib/logrotate/status`（或 `/var/lib/logrotate/status.log` ）来记录上次执行时间。因此，如果你刚运行完一次，logrotate 不会再次执行 `daily` 配置，除非你：
+ 删除状态文件
+ 使用 `-f` 强制忽略状态文件

在大多数 Linux 系统中，logrotate 通过 cron 或 systemd timer 自动运行：
```
cat /etc/cron.daily/logrotate
systemctl list-timers --all | grep logrotate
```

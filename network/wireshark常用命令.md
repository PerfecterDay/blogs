# wireshark 常用命令
{docsify-updated}

- [wireshark 常用命令](#wireshark-常用命令)
  - [](#)


1. 过滤地址
    + `ip.addr==192.168.10.10`  或  `ip.addr eq 192.168.10.10`  #过滤地址
    + `ip.src==192.168.10.10`     #过滤源地址
    + `ip.dst==192.168.10.10`     #过滤目的地址

2. 过滤协议，直接输入协议名
    + `icmp `
    + `http`

3. 过滤协议和端口
    + `tcp.port==80`
    + `tcp.srcport==80`
    + `tcp.dstport==80`

4. 过滤http协议的请求方式
    + `http.request.method=="GET"`
    + `http.request.method=="POST"`
    + `http.request.uri contains admin   #url中包含admin的`
    + `http.request.code==404    #http请求状态码的`

5. 连接符
    + `&& `
    + `||`
    + `and`
    + `or`
  
通过连接符可以把上面的命令连接在一起，比如：
`ip.src==192.168.10.10 and http.request.method=="POST"`


## 
`dst host 198.18.1.227`
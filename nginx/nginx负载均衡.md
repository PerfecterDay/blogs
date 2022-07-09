
# nginx负载均衡
{docsify-updated}




# nginx负载均衡
-

## 负载均衡算法
nginx支持以下几种负载均衡算法：

+ round-robin轮询算法——轮流将请求发送至各个服务器，nginx的默认算法
+ least-connected最少连接算法——将请求转发至活动连接最少的服务器
+ ip-hash算法——基于客户端的请求IP，使用hash算法决定将下一次请求转发至哪台服务器

## round-robin轮询配置
当没有指定轮询算法时，nginx默认使用轮询调度算法。

    http {
        upstream myapp1 {
            server srv1.example.com;
            server srv2.example.com;
            server srv3.example.com;
        }

        server {
            listen 80;

            location / {
                proxy_pass http://myapp1;
            }
        }
    }
srv1、srv2、srv3组成的服务器组，所有匹配到上述server上下文的请求都被转发到myapp1服务器组上，nginx使用轮询算法在srv1-srv3上转发请求。

nginx反向代理支持HTTP, HTTPS, FastCGI, uwsgi, SCGI, memcached, and gRPC。如果要支持https，只要将http改成https即可。

要使用FastCGI, uwsgi, SCGI, memcached, 或者 gRPC, 请分别使用 fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass,  grpc_pass 指令。

## least-connected最少连接算法
要使用least-connected算法，只需在服务器组中使用least_conn指令即可：

    upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

## ip-hash算法、会话维持
上述的round-robin、least-connected算法无法保证统一客户端的后续请求被发送至同一台服务器。这样就会导致会话丢失，如果希望同一客户端的请求被发送至同一台服务器，可以使用ip-hash算法。该算法基于客户端ip进行hash计算以决定将请求发送至哪台服务器，同一ip客户端的请求计算的hash值一样，所以将会被发送至相同的服务器，除非该服务器不可用。

要使用ip-hash算法，只需在服务器组中使用 ip_hash指令即可：

    upstream myapp1 {
        ip_hash;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

## 支持权重的负载均衡

    upstream myapp1 {
        server srv1.example.com weight=3;
        server srv2.example.com;
        server srv3.example.com;
    }
如果有5个请求，3个被转发到srv1,srv2、srv3分别处理一个请求。

least-connected 和 ip-hash 算法的负载均衡也支持权重。

## 健康检查
nginx支持健康检查功能，即检测服务器组内的每台机器是否能正常的提供服务。将不会转发请求到不能正常提供服务的服务器。

max_fails指令设置在fail_timeout时间内，通讯失败的次数。如果被设置为0，表示关闭健康检查功能，默认值是1。如果设置为2，则表示在fail_timeout时间内，2次没有与服务器正常通信，则标记该服务器为非正常服务器。然后每隔fail_timeout时间就检测一次服务器状态，如果能正常通信，则标记为存活服务器。
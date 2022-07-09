
# nginx代理
{docsify-updated}




# nginx代理
--------------------
nginx一个最常用的功能就是作为一个代理服务器，即收到客户端的请求后，将请求转发到被代理的服务器上，收到被代理的服务器响应后，将其发送到客户端。

nginx转发指令是proxy_pass，参数转发的完整地址，包括协议、url和端口号。
如下配置：

    events {
        worker_connections  1024;
    }

    http{
        server {
        location / {
            proxy_pass http://localhost:8080/;
        }

        location ~ \.(gif|jpg|png)$ {
            root html/images;
        }
    }

    server {
        listen 8080;
        root html/proxy;

        location / {
        }
    }
    }

上述配置文件中，所有80端口的图片请求将被映射到html/images目录下,其他请求则会被转发到8080端口上，映射到html/proxy目录中。
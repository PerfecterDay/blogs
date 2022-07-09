
# 配置nginx处理静态内容请求
{docsify-updated}




# 配置nginx处理静态内容请求
--
通常来说，nginx配置文件包含若干个server上下文，不同的server上下文通过监听的端口和名字来区分。一旦nginx决定了用哪个server上下文来处理请求，nginx就会用请求URI与server块指令中配置的location指令的参数相比较，一旦匹配到某个location的参数，URI就会被添加到location块内的root指令参数后边，构成请求的静态内容的访问路径。如果有多个location匹配，nginx选取最长的匹配。
假设有配置文件如下：

    server {
        location / {
            root /data/www;
        }

        location /images/ {
            root /data;
        }
    }

则所有/images/的URI将会匹配到第二个location块，nginx将请求发送到/data/images目录中，如果有个请求是这样：http://localhost/images/example.png， nginx将返回/data/images/example.png的文件，如果文件不存在则报404错误。

其他的请求将会被映射到/data/www目录下。如http://localhost/some/example.html的请求，nginx将返回/data/www/some/example.html文件。

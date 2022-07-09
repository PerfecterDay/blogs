
# nginx的FastCGI代理
{docsify-updated}




# nginx的FastCGI代理
--------------------------
nginx的FastCGI代理可以将请求转发到FastCGI服务器上，比如PHP服务。

主要是用fastcgi_pass指令和fastcgi_param指令。

    server {
        location / {
            fastcgi_pass  localhost:9000;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param QUERY_STRING    $query_string;
        }

        location ~ \.(gif|jpg|png)$ {
            root html/images;
        }
    }
在PHP中，SCRIPT_FILENAME代表PHP脚本的名字，QUERY_STRING是请求的参数。
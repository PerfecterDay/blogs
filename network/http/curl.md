# curl 命令简介
{docsify-updated}

- [curl 命令简介](#curl-命令简介)
		- [检查Http服务是否支持gzip](#检查http服务是否支持gzip)


1. 不带有任何参数时，curl 就是发出 GET 请求。
	`curl http://www.baidu.com`
2. `-A` ：指定客户端的用户代理标头，即 `User-Agent` 。curl 的默认用户代理字符串是`curl/[version]`
3. `-b` ：用来向服务器发送 Cookie。
	`curl -b 'foo=bar' https://google.com`
	会生成一个http头`Cookie: foo=bar`，向服务器发送一个名为foo、值为bar的 Cookie
4. `-c` ：将服务器设置的 Cookie 写入一个文件。
	`curl -c cookies.txt https://www.google.com`
5. `-d` ：用于发送 POST 请求的数据体。
	`curl -d'login=emma＆password=123'-X POST https://google.com/login`
	还可以读取本地文件的数据发送给服务器。
	`curl -d '@data.txt' https://google.com/login`
6. `--data-urlencode`：等同于`-d`，但是会自动进行URL编码。
7. `--data-raw`： 发送原始数据体
	```
	curl --location --request POST 'https://10.187.144.42:8000/trade/' \
	--header 'App-Name: gtjagjapp' \
	--header 'App-ID: sdsdawdsads' \
	--header 'Content-Type: application/json' \
	--data-raw '[
		{
			"code": "1231",
			"params": {
				"username":"700210",
				"token":"ABCDEFGHIJKLMNOPQRST0123456789",
				"account_type":"",
				"market_code":"",
				"currency_code":"HKD",
				"balance_type":"HKD",
				"language":"1"
			},
			"params_ext":{}
		}
	]'
	```
8. `-e` ：用来设置 HTTP 的标头Referer，表示请求的来源。
9.  `-F` ：用来向服务器上传二进制文件。
	`curl -F 'file=@photo.png' https://google.com/profile`
	会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。
10. `-G` ：用来构造 URL 的get请求的查询字符串。
11. `-H` ：添加 HTTP 请求的标头。  `curl -H "X-First-Name: Joe" https://example.com`
12. `-i` ：打印出服务器回应的 HTTP 标头。
13. `-I` ：向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。
14. `-k` ：指定跳过 SSL 检测。
15. `-L` ：让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。
16. `--limit-rate` ：用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。
17. `-o` ：将服务器的回应保存成文件，等同于 `wget` 命令.
18. `-O` ：将服务器回应保存成文件，并将 URL 的最后部分当作文件名。
19. `-s` ：将不输出错误和进度信息。
20. `-S` ：指定只输出错误信息，通常与-s一起使用。
21. `-u` ：用来设置服务器认证的用户名和密码。
22. `-v` ：输出通信的整个过程，用于调试。
23. `-x` ：指定 HTTP 请求的代理。
24. `-X` ：指定 HTTP 请求的方法。
25. `--cert`: 指定证书
26. `--form`: 指定 form 参数， `--from 'UserID=67buu'`
27. `--http1.1/--http2/--http3`: 指定使用的http 版本 
28. `curl --compressed -vv 'https://www.baidu.com' -w '%{size_download}'` : 显示压缩数据及数据大小
29. `-w` : 按指定格式显示本次传输的信息，用字符串表示

### 检查Http服务是否支持gzip
```
curl https://www.baidu.com/ --silent --write-out "%{size_download}\n" --output /dev/null
curl https://www.baidu.com/ --silent -H "Accept-Encoding: gzip,deflate" --write-out "%{size_download}\n" --output /dev/null
```
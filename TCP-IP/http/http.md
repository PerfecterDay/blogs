## Http 概述
{docsify-updated}
>https://developer.mozilla.org/zh-CN/docs/Web/HTTP

### 统一资源标识符的语法 (URI)
```
http://www.example.com:80/path/to/myfile?key1=val1&key2=val2#somewhereInDocument
协议: http://
主机域名:www.example.com
端口: 80
路径: /path/to/myfile
查询参数: key1=val1&key2=val2
片段标识(锚): #somewhereInDocument,锚点代表资源内的一种“书签”，它给予浏览器显示位于该“加书签”点的内容的指示。例如，在 HTML 文档上，
浏览器将滚动到定义锚点的那个点上；在视频或音频文档上，浏览器将转到锚点代表的那个时间。值得注意的是 # 号后面的部分，也称为片段标识符，
永远不会与请求一起发送到服务器。
```

常见的协议有：
+ data：	Data URIs
+ file：	指定主机上文件的名称
+ ftp：	文件传输协议
+ http/https：	超文本传输 协议／安全的超文本传输协议,标准端口分别是80/443
+ mailto：	电子邮件地址
+ ssh：	安全 shell
+ tel：	电话
+ urn：	统一资源名称
+ view-source：	资源的源代码
+ ws/wss：	（加密的）WebSocket (en-US) 连接


### Http 消息格式
<center><img src="pics/httpmsgstructure2.png" width="80%"></center>

1. Http 请求
	1.  一个 HTTP 方法，一个动词（像 GET、PUT 或者 POST）或者一个名词（像 HEAD 或者 OPTIONS），描述要执行的动作。例如，GET 表示要获取资源，POST 表示向服务器推送数据（创建或修改资源，或者产生要返回的临时文件）。
	2. 请求目标（request target），通常是一个 URL，或者是协议、端口和域名的绝对路径，通常以请求的环境为特征。请求的格式因不同的 HTTP 方法而异。它可以是：
      	+ 一个绝对路径，末尾跟上一个 '?' 和查询字符串。这是最常见的形式，称为原始形式（origin form），被 GET、POST、HEAD 和 OPTIONS 方法所使用。
      		POST / HTTP/1.1
      		GET /background.png HTTP/1.0
      		HEAD /test.html?query=alibaba HTTP/1.1
      		OPTIONS /anypage.html HTTP/1.0
      	+ 一个完整的 URL，被称为绝对形式（absolute form），主要在使用 GET 方法连接到代理时使用。GET http://developer.mozilla.org/en-US/docs/Web/HTTP/Messages HTTP/1.1
      	+ 由域名和可选端口（以 ':' 为前缀）组成的 URL 的 authority 部分，称为 authority form。仅在使用 CONNECT 建立 HTTP 隧道时才使用。CONNECT developer.mozilla.org:80 HTTP/1.1
      	+ 星号形式（asterisk form），一个简单的星号（'*'），配合 OPTIONS 方法使用，代表整个服务器。OPTIONS * HTTP/1.1
	3. HTTP 版本（HTTP version），定义了剩余消息的结构，作为对期望的响应版本的指示符。

2. HTTP 响应
	HTTP 响应的起始行被称作状态行（status line），包含以下信息：

	1. 协议版本，通常为 HTTP/1.1。
	2. 状态码（status code），表明请求是成功或失败。常见的状态码是 200、404 或 302。
	3. 状态文本（status text）。一个简短的，纯粹的信息，通过状态码的文本描述，帮助人们理解该 HTTP 消息。 

	一个典型的状态行看起来像这样：`HTTP/1.1 404 Not Found`。


### Http 1.1 的连接管理（HTTP/2 新增了其他连接管理模型）
连接管理是一个 HTTP 的关键话题：打开和保持连接在很大程度上影响着网站和 Web 应用程序的性能。在 HTTP/1.x 里有多种模型：**短连接、长连接和 HTTP 流水线**。
<center><img src="pics/http1_x_connections.png" width="50%"></center>

1. 短连接  
	HTTP 的传输协议主要依赖于 TCP 来提供从客户端到服务器端之间的连接。在早期，HTTP 使用一个简单的模型来处理这样的连接。这些连接的生命周期是短暂的：每发起一个请求时都会创建一个新的连接，并在收到应答时立即关闭。这意味着发起每一个 HTTP 请求之前都会有一次 TCP 握手，而且是连续不断的。

	这个简单的模型对性能有先天的限制：打开每一个 TCP 连接都是相当耗费资源的操作。客户端和服务器端之间需要交换好些个消息。当请求发起时，网络延迟和带宽都会对性能造成影响。现代浏览器往往要发起很多次请求（十几个或者更多）才能拿到所需的完整信息，证明了这个早期模型的效率低下。

	这是 HTTP/1.0 的默认模型（如果没有指定 Connection 协议头，或者是值被设置为 close）。而在 HTTP/1.1 中，只有当 Connection 被设置为 close 时才会用到这个模型。除非是要兼容一个非常古老的，不支持长连接的系统，没有一个令人信服的理由继续使用这个模型。

2. 长连接  
	短连接有两个比较大的问题：创建新连接耗费的时间尤为明显，另外 TCP 连接的性能只有在该连接被使用一段时间后（热连接）才能得到改善。为了缓解这些问题，长连接的概念便被设计出来了，甚至在 HTTP/1.1 之前。或者，这被称之为一个 keep-alive 连接。

	一个长连接会保持一段时间，重复用于发送一系列请求，节省了新建 TCP 连接握手的时间，还可以利用 TCP 的性能增强能力。当然这个连接也不会一直保留着：连接在空闲一段时间后会被关闭（服务器可以使用 Keep-Alive 协议头来指定一个最小的连接保持时间）。

	长连接也还是有缺点的；就算是在空闲状态，它还是会消耗服务器资源，而且在重负载时，还有可能遭受 DoS 攻击。这种场景下，可以使用非长连接，即尽快关闭那些空闲的连接，也能对性能有所提升。

	HTTP/1.0 里默认并不使用长连接。把 Connection 设置成 close 以外的其他参数都可以让其保持长连接，通常会设置为 retry-after。

	在 HTTP/1.1 里，默认就是长连接的，不再需要标头（但我们还是会把它加上，万一某个时候因为某种原因要退回到 HTTP/1.0 呢）。
3. 流水线（注意：HTTP 流水线在现代浏览器中并不是默认被启用的。）  
	默认情况下，HTTP 请求是按顺序发出的。下一个请求只有在当前请求收到响应过后才会被发出。由于会受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

	流水线是在同一条长连接上发出连续的请求，而不用等待应答返回。这样可以避免连接延迟。理论上讲，性能还会因为两个 HTTP 请求有可能被打包到一个 TCP 消息包中而得到提升。就算 HTTP 请求不断的继续，尺寸会增加，但设置 TCP 的最大分段大小（MSS）选项，仍然足够包含一系列简单的请求。

	并不是所有类型的 HTTP 请求都能用到流水线：只有幂等方式，比如 GET、HEAD、PUT 和 DELETE 能够被安全地重试。如果有故障发生时，流水线的内容要能被轻易的重试。

	今天，所有遵循 HTTP/1.1 标准的代理和服务器都应该支持流水线，虽然实际情况中还是有很多限制：一个很重要的原因是，目前没有现代浏览器默认启用这个特性。

#### 域名分片
作为 HTTP/1.x 的连接，请求是序列化的，哪怕本来是无序的，在没有足够庞大可用的带宽时，也无从优化。一个解决方案是，浏览器为每个域名建立多个连接，以实现并发请求。曾经默认的连接数量为 2 到 3 个，现在比较常用的并发连接数已经增加到 6 条。如果尝试大于这个数字，就有触发服务器 DoS 保护的风险。

如果服务器端想要更快速的响应网站或应用程序的应答，它可以迫使客户端建立更多的连接。例如，不要在同一个域名下获取所有资源，假设有个域名是 `www.example.com`，我们可以把它拆分成好几个域名：`www1.example.com、www2.example.com、www3.example.com`。所有这些域名都指向同一台服务器，浏览器会同时为每个域名建立 6 条连接（在我们这个例子中，连接数会达到 18 条）。这一技术被称作**域名分片**。

除非你有紧急而迫切的需求，**不要使用这一过时的技术**；而是升级到 HTTP/2。在 HTTP/2 里，做域名分片就没必要了：HTTP/2 的连接可以很好的处理并发的无优先级的请求。域名分片甚至会影响性能。大多数 HTTP/2 的实现还会使用一种称作[连接聚合](https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/)的技术去尝试合并被分片的域名。

## Content-type
raw->json
	Content-Type: application/json
	Req-body
	{
		"a": "1"
	}

raw->text
	Content-Type: application/text
	Req-body
	{
		"a": "1"
	}

raw->xml
	Content-Type: application/xml
	Req-body
	{
		"a": "1"
	}


form
	Content-Type: multipart/form-data; boundary=--------------------------058363263533734731568425
	Req-body
	a: "1"


x-www-form-urlencoded
	Content-Type: application/x-www-form-urlencoded
	Req-body
	a: "1"
#  UDP协议
{docsify-updated}

- [UDP协议](#udp协议)
  - [UDP 概述](#udp-概述)
  - [UDP首部格式](#udp首部格式)

UDP是一个简单的面向数据报的运输层协议:进程的每个输出操作都正好产生一个UDP数据报，并组装成一份待发送的IP数据报。这与面向流字符的协议不同。如TCP，应用程序产生的全体数据与真正发送的单个IP数据报可能没有什么联系。

### UDP 概述

UDP的主要特点是：

1. UDP是无连接的，即发送数据之前不需要建立连接，因此减少了开销和发送数据之前的时延。
2. UDP使用尽最大努力交付，即不保证可靠交付，因此主机不需要维持复杂的连接状态。
3. UDP是面向报文的。发送方的UDP对应用程序交下来的报文，在添加首部后就向下交付给IP层，UDP既不会合并报文也不会拆分，而是保留这些报文的边界。接收方UDP收到报文后，去除首部后就上交给应用层程序
4. UDP没有拥塞控制,因此网络出现堵塞时不会是源主机的发送速率降低。
5. UDP支持一对一、一对多、多对一和多对多的交互通信。
6. UDP的首部开销小，只有8个字节，比TCP的20字节首部短。

### UDP首部格式
<center><img src="pics/udp.png" alt="" width=40%></center>

1. 源端口号：需要对方回信时使用，不需要时可用全0。
2. 目的端口号：必填字段
3. UDP长度：整个UDP报文的长度，包括首部和数据部分
4. 检验和：检测UDP在传输过程中是否有错。UDP检验和会把首部和数据部分一起检验。
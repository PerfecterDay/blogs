# java基础-NIO
{docsify-updated}

- [java基础-NIO](#java基础-nio)
  - [流与块的比较](#流与块的比较)
  - [通道](#通道)
  - [缓冲区](#缓冲区)
    - [缓冲区的状态变量](#缓冲区的状态变量)
      - [Capacity](#capacity)
      - [position](#position)
      - [limit](#limit)
      - [flip方法](#flip方法)
      - [clear方法](#clear方法)
      - [get() 方法](#get-方法)
      - [put()方法](#put方法)
      - [缓冲区分配和包装](#缓冲区分配和包装)
  - [NIO 网络编程的一般步骤](#nio-网络编程的一般步骤)
    - [服务器端打开一个 ServerSocketChannel](#服务器端打开一个-serversocketchannel)
    - [Selector](#selector)
    - [SelectionKey](#selectionkey)
    - [内部循环](#内部循环)
    - [监听新连接](#监听新连接)
    - [接受新的连接](#接受新的连接)
    - [删除处理过的 SelectionKey](#删除处理过的-selectionkey)
    - [处理传入的 I/O 数据](#处理传入的-io-数据)
    - [回到主循环](#回到主循环)


主要转自：https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html

## 流与块的比较
面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节流的数据，一个输出流消费一个字节流的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是，面向流的 I/O 通常相当慢。

一个 面向块 的 I/O 系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按(流式的)字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

## 通道
实际上 ChanneI 可以分为两大类 ： 分别是用于网络读写的 `SelectableChannel` 和用于文件操作的 `FileChannel` 。

Channel 是一个对象，可以通过它读取和写入数据。拿 NIO 与原来的 I/O 做个比较，通道就像是流。 所有数据都通过 Buffer 对象来处理。永远不会将字节直接写入通道中，相反，是将数据写入包含一个或者多个字节的缓冲区。同样，也不会直接从通道中读取字节，而是将数据从通道读入缓冲区，再从缓冲区获取这个字节。 
通道与流的不同之处在于通道是双向的。而流只是在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)， 而通道可以用于读、写或者同时用于读写。 

常见的Channel有： `FileChannel` 、 `SelectableChannel` 、 `ServerSocketChannel` 、 `SocketChannel` 、 `DatagramChannel` 、 `Pipe.SinkChannel` 、 `Pipe.SourceChannel` 。所有 Channel 都不应该使用构造器来直接创建，而是：
1. 通过相应 Channel 的静态方法 `open` 打开
2. 或者通过传统的节点流 `InputStream/OutputStream` 的 `getChannel` 方法来返回对应的 Channel。不同的节点流获取的 Channel 不一样， `FileInputStream/FileOutputStream->FileChannel; PipedInputStream->Pipe.SinkChannel; PipedOutputStream->Pipe.SourceChannel` 。

Channel中最常用的方法：
+ map: 将Channel对应的部分或全部数据映射到 ByteBuffer
+ read: 有很多重载方法，从Channel中读取数据到给定 buffer
+ write: 有很多重载方法，将 buffer 中的数据写入到 Channel

## 缓冲区
Buffer 是一个对象， 它包含一些要写入或者刚读出的数据。 在 NIO 中加入 Buffer 对象，体现了新库与原 I/O 的一个重要区别。在面向流的 I/O 中，您将数据直接写入或者将数据直接读到 Stream 对象中。

在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的。在写入数据时，它是写入到缓冲区中的。任何时候访问 NIO 中的数据，都是将它放到缓冲区中。

缓冲区实质上是一个数组。通常它是一个字节数组，但是也可以使用其他种类的数组。但是一个缓冲区不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

### 缓冲区的状态变量
可以用三个值指定缓冲区在任意时刻的状态：
+ capacity
+ position
+ limit

这三个变量一起可以跟踪缓冲区的状态和它所包含的数据。

#### Capacity
缓冲区的 capacity 表明可以储存在缓冲区中的最大数据容量。实际上，它指定了底层数组的大小 ― 或者至少是指定了准许我们使用的底层数组的容量。 
limit 决不能大于 capacity。

#### position
position 变量跟踪已经写了多少数据。更准确地说，它指定了下一个字节将放到数组的哪一个元素中。因此，如果您从通道中读三个字节到缓冲区中，那么缓冲区的 position 将会设置为3，指向数组中第四个元素。  
同样，在写入通道时，您是从缓冲区中获取数据。 position 值跟踪从缓冲区中获取了多少数据。更准确地说，它指定下一个字节来自数组的哪一个元素。因此如果从缓冲区写了5个字节到通道中，那么缓冲区的 position 将被设置为5，指向数组的第六个元素。

#### limit
limit 变量表明还有多少数据需要取出。(在从缓冲区写入通道时)limit等于缓冲区中数据的数量，或者还有多少空间可以放入数据(在从通道读入缓冲区时)，limit应该等于capacity。  
position 总是小于或者等于 limit。

#### flip方法
现在我们要将数据写到输出通道中。在这之前，我们必须调用 flip() 方法。这个方法做两件非常重要的事：
+ 它将 limit 设置为当前 position。
+ 它将 position 设置为 0。

#### clear方法
从通道读入数据到缓冲区之前，这个方法重设缓冲区以便接收更多的字节。 Clear 做两件非常重要的事情：
+ 它将 limit 设置为与 capacity 相同。
+ 它设置 position 为 0。

#### get() 方法
ByteBuffer 类中有四个 get() 方法：
+ byte get(): 获取单个字节
+ ByteBuffer get(byte[] dst): 将一组字节读到一个数组中
+ ByteBuffer get(byte[] dst, int offset, int length): 将一组字节读到一个数组中，从 offset 开始的 length 个字节
+ byte get( int index ): 从缓冲区中的特定位置获取字节

此外，我们认为前三个 get() 方法是相对的，而最后一个方法是绝对的。 相对意味着 get() 操作服从 limit 和 position 值 ― 更明确地说，字节是从当前 position 读取的，而 position 在 get 之后会增加。另一方面，一个 绝对方法会忽略 limit 和 position 值，也不会影响它们。事实上，它完全绕过了缓冲区的统计方法。  
上面列出的方法对应于 ByteBuffer 类。其他类有等价的 get() 方法，这些方法除了不是处理字节外，其它方面是是完全一样的，它们处理的是与该缓冲区类相适应的类型。

#### put()方法
ByteBuffer 类中有五个 put() 方法：
+ ByteBuffer put( byte b );
+ ByteBuffer put( byte src[] );
+ ByteBuffer put( byte src[], int offset, int length );
+ ByteBuffer put( ByteBuffer src );
+ ByteBuffer put( int index, byte b );
第一个方法 写入（put） 单个字节。第二和第三个方法写入来自一个数组的一组字节。第四个方法将数据从一个给定的源 ByteBuffer 写入这个 ByteBuffer。第五个方法将字节写入缓冲区中特定的 位置 。那些返回 ByteBuffer 的方法只是返回调用它们的缓冲区的 this 值。

与 get() 方法一样，我们将把 put() 方法划分为 相对 或者 绝对 的。前四个方法是相对的，而第五个方法是绝对的。

上面显示的方法对应于 ByteBuffer 类。其他类有等价的 put() 方法，这些方法除了不是处理字节之外，其它方面是完全一样的。它们处理的是与该缓冲区类相适应的类型。

#### 缓冲区分配和包装
在能够读和写之前，必须有一个缓冲区。要创建缓冲区;
1. 分配它。我们使用静态方法 allocate() 来分配缓冲区：

    ByteBuffer buffer = ByteBuffer.allocate( 1024 );
2. 还可以将一个现有的数组转换为缓冲区，如下所示：

    byte array[] = new byte[1024];
    ByteBuffer buffer = ByteBuffer.wrap( array );

## NIO 网络编程的一般步骤

### 服务器端打开一个 ServerSocketChannel
为了接收连接，我们需要一个 `ServerSocketChannel` 。事实上，我们要监听的每一个端口都需要有一个 `ServerSocketChannel` 。对于每一个端口，我们打开一个 `ServerSocketChannel` ，不能直接使用 `ServerScocket` 的 getChannel 方法来获得 ServerSocketChannel 对象，也不能直接绑定到端口，必须使用 socket() 方法获得关联的 ServerScocket 对象，然后再用该 ServerScocket 对象来绑定端口。如下所示：
```
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.configureBlocking( false );
ServerSocket ss = ssc.socket();
InetSocketAddress address = new InetSocketAddress( ports[i] );
ss.bind( address );
```
第一行创建一个新的 `ServerSocketChannel` ，最后三行将它绑定到给定的端口。第二行将 `ServerSocketChannel` 设置为*非阻塞*的 。我们必须对每一个要使用的套接字通道调用这个方法，否则异步 I/O 就不能工作。

### Selector
`Selector` 就是您注册对各种 I/O 事件的兴趣的地方，而且当那些事件发生时，就是这个对象告诉您所发生的事件。
我们关心某个channel是否发生了读写或者Accept事件，就把这个channel和相应的事件通过channel的register方法注册到selector对象上，这样selector会在这些channel上发生了你感兴趣的事件时通知你。

所以，我们需要做的第一件事就是创建一个 Selector，通过 Selector 类的静态方法 opeb() 来获得系统默认的 Selector：
    
    Selector selector = Selector.open();
然后，我们将对不同的*通道对象*调用 `register()` 方法，以便注册我们对这些对象中发生的 I/O 事件的兴趣。`register()` 的第一个参数总是这个 Selector。Selector可以同时监控注册到其上的多个 SelectableChannel 的IO状况，是非阻塞IO的核心。一个 Selector 有三个SelectionKey的集合：
1. 所有SelectionKey的集合：代表了注册在改Selector上的Channel，可以通过 keys() 获取该集合
2. 所有选择的SelectionKey集合：代表了所有监测到的、需要进行IO处理的Channel，调用select()方法会检测哪些Channel需要IO处理，然后通过 selectedKeys()方法可以获取该集合
3. 被取消的SelectionKey的集合：代表了所有被取消了注册关系的Channel，程序通常不需要直接访问该集合。
同时，Selector 还提供了系列和 select() 相关的方法：
+ int select(): 监控检测所有注册的Channel，将那些需要进行IO处理的Channel所对应的 SelectionKey 添加到所有选择的SelectionKey集合中，返回需要IO处理的Channel的数量。如果没有需要IO处理的Channel，该方法将阻塞线程
+ int select(long timeout): 可以设置超时时长的select
+ int selectNow(): 立即返回的select，不会阻塞线程
+ Selector wakeup(): 使一个未返回的 select() 方法立即返回。

### SelectionKey
SelectionKey代表的是 SelectableChannel 和 Selector 之间的注册关系。通过它可以或得对应的 SelectableChannel 和 Selector:
+ Selector selector(): 获取 Selector
+ SelectableChannel channel(): 获取 SelectableChannel
  

所以，下一步是将新打开的 `ServerSocketChannel` 注册到 `Selector` 上。为此我们使用 `SelectableChannel.register()` 方法，如下所示：
```
SelectionKey key = ssc.register( selector, SelectionKey.OP_ACCEPT );
```
`register()` 的第一个参数总是这个 `Selector` 。第二个参数是 `OP_ACCEPT` ，这里它指定我们想要监听 accept 事件，也就是在新的连接建立时所发生的事件。这是适用于 `ServerSocketChannel` 的唯一事件类型。 
请注意对 `register()` 的调用的返回值。 `SelectionKey` 代表这个通道在此 `Selector` 上的这个注册。当某个 `Selector` 通知您某个传入事件时，它是通过提供对应于该事件的 `SelectionKey` 来进行的。`SelectionKey` 还可以用于取消通道的注册。

### 内部循环
现在已经注册了我们对一些 I/O 事件的兴趣，下面将进入主循环。使用 `Selector` 的几乎每个程序都像下面这样使用内部循环：
```
int num = selector.select();
Set selectedKeys = selector.selectedKeys();
Iterator it = selectedKeys.iterator();
while (it.hasNext()) {
     SelectionKey key = (SelectionKey)it.next();
     // ... deal with I/O event ...
}
```
首先，我们调用 `Selector` 的 `select()` 方法。这个方法会阻塞，直到至少有一个已注册的事件发生。当一个或者更多的事件发生时， `select()` 方法将返回所发生的事件的数量。

接下来，我们调用 `Selector` 的 `selectedKeys()` 方法，它返回发生了事件的 `SelectionKey` 对象的一个集合 。

我们通过迭代 `SelectionKeys` 并依次处理每个 `SelectionKey` 来处理事件。对于每一个 `SelectionKey` ，您必须确定发生的是什么 I/O 事件，以及这个事件影响哪些 I/O 对象。

### 监听新连接
程序执行到这里，我们仅注册了 `ServerSocketChannel` ，并且仅注册它们“接收”事件。为确认这一点，我们对 SelectionKey 调用 `readyOps()` 方法，并检查发生了什么类型的事件：
```
if ((key.readyOps() & SelectionKey.OP_ACCEPT)
     == SelectionKey.OP_ACCEPT) {
     // Accept the new connection
     // ...
}
```
可以肯定地说， `readOps()` 方法告诉我们该事件是新的连接。

### 接受新的连接
因为我们知道这个服务器套接字上有一个传入连接在等待，所以可以安全地接受它；也就是说，不用担心 accept() 操作会阻塞：
```
ServerSocketChannel ssc = (ServerSocketChannel)key.channel();
SocketChannel sc = ssc.accept();
```
下一步是将新连接的 SocketChannel 配置为非阻塞的。而且由于接受这个连接的目的是为了读取来自套接字的数据，所以我们还必须将 SocketChannel 注册到 Selector上，如下所示：
```
sc.configureBlocking( false );
SelectionKey newKey = sc.register( selector, SelectionKey.OP_READ );
```
注意我们使用 register() 的 OP_READ 参数，将 SocketChannel 注册用于 **读取** 而不是 **接受** 新连接。

### 删除处理过的 SelectionKey
在处理 SelectionKey 之后，我们几乎可以返回主循环了。但是我们必须首先将处理过的 SelectionKey 从选定的键集合中删除。如果我们没有删除处理过的键，那么它仍然会在主集合中以一个激活的键出现，这会导致我们尝试再次处理它。我们调用迭代器的 `remove()` 方法来删除处理过的 SelectionKey：

    it.remove();
现在我们可以返回主循环并接受从一个套接字中传入的数据(或者一个传入的 I/O 事件)了。

### 处理传入的 I/O 数据
当来自一个套接字的数据到达时，它会触发一个 I/O 事件。这会导致在主循环中调用 Selector.select()，并返回一个或者多个 I/O 事件。这一次， SelectionKey 将被标记为 OP_READ 事件，如下所示：
```
if ((key.readyOps() & SelectionKey.OP_READ)
     == SelectionKey.OP_READ) {
     // Read the data
     SocketChannel sc = (SocketChannel)key.channel();
     // ...
}
```
与以前一样，我们取得发生 I/O 事件的通道并处理它。在本例中，由于这是一个 echo server，我们只希望从套接字中读取数据并马上将它发送回去。

### 回到主循环
每次返回主循环，我们都要调用 select 的 Selector()方法，并取得一组 SelectionKey。每个键代表一个 I/O 事件。我们处理事件，从选定的键集中删除 SelectionKey，然后返回主循环的顶部。

在现实的应用程序中，您需要通过将通道从 Selector 中删除来处理关闭的通道。而且您可能要使用多个线程。创建一个线程池来负责 I/O 事件处理中的耗时部分会更有意义。
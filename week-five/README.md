# 网络编程基础

## 网络编程

### 网络通信协议

#### 通信

指人与人或人与自然之间通过某种行为或媒介进行的信息交流与传递，从广义上指需要信息的双方或多方采用任意方法，任意媒质，将信息从某方准确安全地传送到另一方。

#### 网络

网络是由节点和连线构成，表示诸多对象及其相互的联系。网络是信息传输、接收、共享的虚拟平台，通过它把各个点、面、体的信息联系到一起，从而实现这些资源的共享。

#### 网络通信

网络通信是通过网络将各个孤立的设备进行连接，通过信息交换实现人与人，人与计算机，计算机与计算机之间的通信。

#### 网络通信协议

主要是对信息传输的速率、传输代码、代码结构、传输控制步骤、出错控制等作出规定并制定出标准。

常见的网络通信协议有：TCP/IP协议、IPX/SPX协议、NetBEUI协议等。

#### TCP/IP协议

互联网相关联的协议集合起来总称为TCP/IP。

#### 计算机网络

计算机网络是通信技术与计算机技术相结合的产物。计算机网络是按照网络协议，将地球上分散的、独立的计算机相互连接的集合。连接介质可以是电缆、双绞线、光纤、微波、载波或通信卫星。计算机网络具有共享硬件、软件和数据资源的功能，具有对共享数据资源集中处理及管理和维护的能力。

计算机网络通常被分为：局域网(LAN)、城域网(MAN)、广域网(WAN)

### TCP与UDP协议

TCP：传输控制协议 (Transmission Control Protocol)。TCP协议是面向连接的通信协议，即传输数据之前，在发送端和接收端建立逻辑连接，然后再传输数据，它提供了两台计算机之间可靠无差错的数据传输。建立一次连接完成三次握手后，客户端和服务器就可以开始进行数据传输了。传输完成之后，通过四次挥手关闭连接。由于这种面向连接的特性，TCP协议可以保证传输数据的安全，所以应用十分广泛，例如下载文件、浏览网页等。

UDP：用户数据报送协议(User Datagram Protocol)。UDP协议是一个面向无连接的协议。传输数据时，不需要建立连接，不管对方端服务是否启动，直接将数据、数据源和目的地都封装在数据包中，直接发送。每个数据包的大小限制在64k以内。它是不可靠协议，因为无连接，所以传输速度快，但是容易丢失数据。日常应用中,例如视频会议、直播...等。

### TCP的三次握手

三次握手：TCP协议中，在发送数据的准备阶段，客户端与服务器之间的三次交互，以保证连接的可靠。

客户端–发送带有 SYN 标志的数据包-1次握手-服务端

服务端–发送带有 SYN/ACK 标志的数据包–2次握⼿–客户端

客户端–发送带有带有 ACK 标志的数据包–3次握⼿–服务端

![三次握手](.\note\三次握手.png)

三次握手的目的是建立可靠的通信信道，说到通讯，简单来说就是数据的发送与接收，而三次握手最主要的目的就是双方确认自己与对方的发送与接收是正常的。

- 第一次握手： Client 什么都不能确认； Server 确认了对方发送正常，自己接收正常
- 第二次握手： Client 确认了：自己发送、接收正常，对方发送、接收正常； Server 确认了：对方发送正常，自己接收正常
- 第三次握手： Client 确认了：自己发送、接收正常，对方发送、接收正常； Server 确认了：自己发送、接收正常，对方发送、接收正常

![四次挥手](.\note\四次挥手.png)

- 客户端-发送⼀个 FIN，用来关闭客户端到服务器的数据传送
- 服务器-收到这个 FIN，它发回一个 ACK，确认序号为收到的序号加1 。和 SYN ⼀样，⼀个FIN 将占用⼀个序号
- 服务器-关闭与客户端的连接，发送⼀个FIN给客户端
- 客户端-发回 ACK 报文确认，并将确认序号设置为收到序号加1

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接。

### 输入URL地址到显示网页经历了哪些过程，会使用到哪些协议？

![url传输协议](.\note\url传输协议.png)

过程：
1. 浏览器查找域名对应IP地址
DNS查找过程：浏览器缓存 ==> 主机HOST ==> 路由器缓存 ==> DNS缓存 ==> 根域名解析服
务器
2. 浏览器想Web服务器发送一个HTTP请求，Cookies会随着请求发送给服务器
3. 服务器处理请求：获取参数和Cookies，处理请求生成一个html响应体
4. 服务器返回响应HTML
5. 浏览器渲染HTML

使用到的协议：

- DNS协议：获取域名对应ip
- HTTP协议-应用层：使用HTTP协议访问网页，建立可靠连接与传输数据需要依靠TCP协议与IP协议
- TCP协议-传输层：与服务器建立TCP连接，并传输数据
- IP协议-网络层：建立TCP协议之后，发送数据在网络层依靠IP协议

### HTTP1.0与HTTP1.1的区别

#### 长连接

在HTTP/1.0中，默认使用的是短连接，也就是说每次请求都要重新建立一次连接。HTTP是基于TCP/IP协议的，每一次建立或者断开连接都需要三次握手四次挥手的开销，如果每次请求都要这样的话，开销会比较大。因此最好能维持一个长连接，可以用个长连接来发多个请求。

HTTP1.1起，默认使用长连接，默认开启 Connection：keep-alive 。HTTP/1.1的持续连接有非流水线方式和流水线（pipelining）方式。

- 流水线方式：客户在收到HTTP的响应报文之前就能接着发送新的请求报文，其实就是并行发送请求。
- 非流水线方式：客户在收到前一人响应后才能发送下一个请求，其实就是串行发送请求。

#### 错误状态响应码

在HTTP1.1中新增了24个错误状态响应码

#### 缓存处理

在HTTP1.0中缓存判断标准单一，HTTP1.1则引入了更多的缓存控制策略

#### 带宽优化及网络连接的使用

HTTP1.0不支持断点续传功能，存在一些浪费带宽的现象。HTTP1.1加入了断点续传的支持，允许只请求资源的某个部分，充分利用带宽和连接，避免浪费带宽。

### HTTP与HTTPS的区别

- **端口**：HTTP的URL由“http:/"起始且默认使用端口80，而HTTPS的URL由“https://"起始且默认使用端口443。
- **安全性和资源消耗**：HTTP协议运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。HTTPS是运行在SSL/TLS之上的HTTP协议，SSL/TLS运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。所以说，HTTP安全性没有HTTPS高，但是HTTPS比HTTP耗费更多服务器资源
  - HTTPS = HTTP + 加密 + 认证 + 完整性保护
  - HTTPS是身披SSL（Secure Socket Layer）外壳的HTTP。通常，HTTP协议直接与TCP协议通信，当使用了SSL之后，就演变为HTTP先于SSL通信，然后再由SSL与TCP通信
- **对称加密：**密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的对称加密算法有DES、AES等；
- **非对称加密：**密钥成对出现，且根据公钥（public key）无法推知私钥，根据私钥也无法推知公钥，加密解密使用不同密钥。公钥加密需要私钥解密，私钥加密需要公钥解密，相对对称加密速度较慢，典型的非对称加密算法有RSA、DSA等。

![HTTP与HTTPS的区别](.\note\HTTP与HTTPS的区别.png)

### URI与URL的区别

- URI（Uniform Resource Identifier）统一资源标志符，资源抽象的定义，不管用什么方法表示，只要能定位一个资源，就叫URI。
- URL（Uniform Resource Locator）统一资源定位符，是一种具体的URI，在用地址定位
- URN（Uniform Resource Name）统一资源名称，也是一种具体的URI，在用名称定位

URL详解：

- URL应用广泛，作用是为了告诉使用者 某个资源在 Web 上的地址
- 这个资源可以是HTML文档，JS脚本，CSS样式，font字体，图片，视频等等。

![URL地址的构成](.\note\URL地址的构成.png)

- 协议：使用 http: 或 https: 等协议方案名获取访问资源时要指定协议类型。不区分字母大小写，最后附一个冒号（:）
- 登陆信息（认证）：指定用户名和密码作为从服务器端获取资源时必要的登录信息（身份认证），可省略
- 服务器地址：必须指定访问的服务器地址。地址可以是域名，可以是IPv4地址192.168.1.1或172.17.187.80，也可以IPv6地址[0:0:0:0:0:0:0:1] 用方括号括起来
- 端口号：指定服务器端口号，也可省略，若用户省略则使用默认端口号80
- 带层次的文件路径：指定服务器上的文件路径来定位特指的资源
- 查询参数：针对已指定的文件路径内的资源，可以使用查询字符串传入任意参数，可省略
- 片段标识符：使用片段标识符通常可标记出已获取资源中的子资源，也就是文档内的某位置，可省略

## BIO与NIO

### BIO

BIO 有的称之为 basic(基本) IO，有的称之为 block(阻塞) IO，主要应用于文件 IO 和网络 IO。

![BIO](.\note\BIO.png)

### NIO

**java.nio 全称 Java Non-Blocking IO，是指 JDK 提供的新 API。**

从 JDK1.4 开始，Java 提供了一系列改进的输入/输出的新特性，被统称为 NIO(即 New IO)。新增了许多用于处理输入输出的类，这些类都被放在 java.nio 包及子包下，并且对原 java.io 包中的很多类进行改写，新增了满足 NIO 的功能。

NIO 和 BIO 有着相同的目的和作用，但是它们的实现方式完全不同；

- BIO 以流的方式处理数据，而 NIO 以块的方式处理数据，块 IO 的效率比流 IO 高很多。
- NIO 是非阻塞式的，这一点跟 BIO 也很不相同，使用它可以提供非阻塞式的高伸缩性网络。

**NIO 主要有三大核心部分：**

- Channel通道
- Buffer缓冲区
- Selector选择器

传统的 BIO 基于字节流和字符流进行操作，而 NIO 基于 Channel和 Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道。

### 文件 IO

#### 概述和核心 API

缓冲区（Buffer）：实际上是一个容器，是一个特殊的数组，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。

Channel 提供从文件、网络读取数据的渠道， 但是读取或写入的数据都必须经由 Buffer

在 NIO 中，Buffer 是一个顶层父类，它是一个抽象类，常用的 Buffer 子类有：

- ByteBuffer，存储字节数据到缓冲区
- ShortBuffer，存储短整型数据到缓冲区
- CharBuffer，存储字符数据到缓冲区
- IntBuffer，存储整数数据到缓冲区
- LongBuffer，存储长整型数据到缓冲区
- DoubleBuffer，存储小数到缓冲区
- FloatBuffer，存储小数到缓冲区

对于 Java 中的基本数据类型， 都有一个 Buffer 类型与之相对应，最常用的自然是ByteBuffer 类（字节缓冲），该类的主要方法如下所示：

- public abstract ByteBuffer put(byte[] b); 存储字节数据到缓冲区
- public abstract byte[] get(); 从缓冲区获得字节数据
- public final byte[] array(); 把缓冲区数据转换成字节数组
- public static ByteBuffer allocate(int capacity); 设置缓冲区的初始容量
- public static ByteBuffer wrap(byte[] array); 把一个现成数组放到缓冲区中使用
- public final Buffer flip(); 翻转缓冲区，重置位置到初始位置（缓冲区有一个指针从头开始读取数据，读到缓冲区尾部时，可以使用这个方法，将指针重新定位到头）

**Channel**：类似于 BIO 中的 stream，例如 FileInputStream 对象，用来建立到目标（文件，网络套接字，硬件设备等）的一个连接，但是需要注意：BIO 中的 stream 是单向的，例如 FileInputStream 对象只能进行读取数据的操作，而 NIO 中的通道(Channel)是双向的， 既可以用来进行读操作，也可以用来进行写操作。

常用的 Channel 类有：FileChannel、DatagramChannel、ServerSocketChannel 和SocketChannel。

- FileChannel 用于文件的数据读写
- DatagramChannel 用于 UDP 的数据读写
- ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写。

**FileChannel** 类，该类主要用来对本地文件进行 IO 操作，主要方法如下所示：

- public int read(ByteBuffer dst) ，从通道读取数据并放到缓冲区中
- public int write(ByteBuffer src) ，把缓冲区的数据写到通道中
- public long transferFrom(ReadableByteChannel src, long position, long count)，从目标通道中复制数据到当前通道
- public long transferTo(long position, long count, WritableByteChannel target)，把数据从当前通道复制给目标通道

### 网络 IO

Java NIO 中的网络通道是非阻塞 IO 的实现，基于事件驱动，非常适用于服务器需要维持大量连接，但是数据交换量不大的情况

在 Java 中编写 Socket 服务器，通常有以下几种模式：

一个客户端连接用一个线程

- 优点：程序编写简单
- 缺点：如果连接非常多，分配的线程也会非常多，服务器可能会因为资源耗尽而崩溃。

把每个客户端连接交给一个拥有固定数量线程的连接池

- 优点：程序编写相对简单， 可以处理大量的连接。
- 缺点：线程的开销非常大，连接如果非常多，排队现象会比较严重。

使用 Java 的 NIO，用非阻塞的 IO 方式处理

- 优点：这种模式可以用一个线程，处理大量的客户端连接
- 缺点：代码复杂度较高，不易理解

#### Selector选择器

![Selector](.\note\Selector.png)

该类的常用方法如下所示：

- public static Selector open()，得到一个选择器对象
- public int select(long timeout)，监控所有注册的通道，当其中有 IO 操作可以进行时，将对应的SelectionKey 加入到内部集合中并返回，参数用来设置超时时间
- public Set selectedKeys()，从内部集合中得到所有的 SelectionKey

#### SelectionKey

代表了 Selector 和网络通道的注册关系

一共四种（就是连接事件）

- int OP_ACCEPT：有新的网络连接可以 accept，值为 16
- int OP_CONNECT：代表连接已经建立，值为 8
- int OP_READ 和 int OP_WRITE：代表了读、写操作，值为 1 和 4

该类的常用方法如下所示：

- public abstract Selector selector()，得到与之关联的 Selector 对象
- public abstract SelectableChannel channel()，得到与之关联的通道
- public final Object attachment()，得到与之关联的共享数据
- public abstract SelectionKey interestOps(int ops)，设置或改变监听事件
- public final boolean isAcceptable()，是否可以 accept
- public final boolean isReadable()，是否可以读
- public final boolean isWritable()，是否可以写

#### ServerSocketChannel

- public static ServerSocketChannel open()，得到一个 ServerSocketChannel 通道
- public final ServerSocketChannel bind(SocketAddress local)，设置服务器端端口号
- public final SelectableChannel configureBlocking(boolean block)，设置阻塞或非阻塞模式， 取值 false 表示采用非阻塞模式
- public SocketChannel accept()，接受一个连接，返回代表这个连接的通道对象
- public final SelectionKey register(Selector sel, int ops)，注册一个选择器并设置监听事件

#### SocketChannel

网络 IO 通道，具体负责进行读写操作

NIO 总是把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。

常用方法如下所示：

- public static SocketChannel open()，得到一个 SocketChannel 通道
- public final SelectableChannel configureBlocking(boolean block)，设置阻塞或非阻塞模式， 取值 false 表示采用非阻塞模式
- public boolean connect(SocketAddress remote)，连接服务器
- public boolean finishConnect()，如果上面的方法连接失败，接下来就要通过该方法完成连接操作
- public int write(ByteBuffer src)，往通道里写数据
- public int read(ByteBuffer dst)，从通道里读数据
- public final SelectionKey register(Selector sel, int ops, Object att)，注册一个选择器并设置监听事件，最后一个参数可以设置共享数据
- public final void close()，关闭通道

## Netty核心技术

### 概述

- Netty是一个被广泛使用的Java网络应用编程框架
- Nettv框架帮助开发者快速、简单的实现一个客户端/服务端的网络应用程序。
- Netty利用Java 语言的NIO网络编程的能力，并隐藏其背后的复杂性从而提供了简单易用的 API

### 特点

- API简单易用: 支持阻塞和非阻塞式的socket
- 基于事件模型: 可扩展性和灵活性更强
- 高度定制化的线程模型: 支持单线程和多线程
- 高通吐、低延迟、资源占用率低
- 完整支持SSL和TLS

### 应用场景

互联网行业：分布式系统远程过程调用，高性能的RPC框架

游戏行业：大型网络游戏高性能通信

大数据：Hadoop的高性能通信和序列化组件 Avro 的 RPC 框架

![线程池模型](.\note\线程池模型.png)

![Netty 线程模型](.\note\Netty 线程模型.png)

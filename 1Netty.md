# 基础知识

## BIO、NIO、AIO有什么区别

- BIO：同步阻塞IO，实现方式为一个连接对应一个线程，数据的读取与写入必须阻塞在一个线程内等待完成。
- NIO：同步非阻塞IO，在一个线程内**不断的轮询数据的状态**，看看是否有数据准备好发生了改变，从而进行下一步的操作。NIO在读取数据是仍然是同步的，也就是将数据从内核缓冲区复制到用户缓冲区时，**用户线程仍然是要阻塞的**
- AIO：异步非阻塞IO，异步非阻塞IO基于事件与回调机制实现，**无需一个线程去轮询所有IO操作的状态改变**，由系统完成数据读取至用户缓冲区的操作，完成之后，系统会通知对应的线程来处理。

## Netty是什么

1. Netty是一个基于NIO的client-server框架，使用它可以简单快速地开发网络应用程序
2. 它极大地简化并优化了TCP和UDP套接字服务器等网络编程，并且性能与安全性等方面做得更好
3. 支持FTP、HTTP以及各种二进制与基于文本的传统协议

总而言之：Netty成功地找到了一种在不妥协可维护性与性能的情况下，实现易于开发、高性能、稳定、灵活的方案

## 为什么不直接用NIO

Netty基于NIO开发，原生NIO的编程模型比较复杂，且容易出现Bug，对编程功底要求较高，且大量代码为重复的格式化代码，如果用NIO的话，用起来会比较麻烦。

而且，NIO在面对断线重连、包丢失、粘包等问题时的处理过程非常复杂，而Netty解决了这些问题

## 为什么要用Netty

Netty具有以下优点：
- 提供了同一的API，支持多种传输类型（包含阻塞的与非阻塞的）
- 基于简单而强大的线程模型
- 自带编码器，解决了TCP粘包/拆包的问题
- 自带各种协议栈
- 实现了真正的无连接数据包套接字支持
- 比直接使用Java核心API的吞吐量更高、延迟更低、资源消耗更少、内存复制更少
- 安全性不错，有完整的SSL/TLS支持

## Netty的应用场景

Netty是对NIO的封装，所以其主要任务和NIO相同，即网络通信：
- 作为RPC框架的网络通信工具
- 用于实现HTTP服务器


## Netty的核心组件

![](./../images/Netty/Netty架构.png)

### Bytebuf（字节容器）

> 网络通信中，最终都是通过字节流来进行传输的

ByteBuf是Netty提供的一个**字节容器**，其内部是一个**数组**，用于Netty数据传输。

ByteBuf可以看作是Netty对Java NIO中的ByteBuffer字节容器的封装与抽象，由于ByteBuffer使用起来比较繁琐，所以用ByteBuf替代

ByteBuf通过维护读索引与写索引，实现了读取与写入的位置标识，从而分别控制读访问与写访问。

ByteBuf共有三种模式：
- 堆缓冲区模式：
  - 又称为支撑数组，将数据存放在JVM的堆空间，通过将数据存储在数组中实现。
- 直接缓冲区模式：
  - 使用的是堆外分配的**直接内存**，**不会占用堆的空间**。
- 复合缓冲区模式：
  - 本质上类似于提供一个或多个ByteBuf的组合视图，可以根据需要添加和删除不同类型的ByteBuf。

### Bootstrap和ServerBootstrap（启动引导类）

Bootstrap是客户端的启动引导类，ServerBootstrap是服务端的启动引导类

1. Bootstrap通常使用`connect()`方法连接到远程主机的端口，作为一个Netty TCP协议通信中的客户端。另外，Bootstrap也可以通过`bind()`方法绑定本地的一个端口，作为UDP协议通信中的一端
2. ServerBootstrap通常使用`bind()`方法绑定在本地的端口上，然后等待客户端的连接
3. Bootstrap只需要配置一个线程组EventLoopGroup，而`ServerBootstrap`需要配置**两个线程组**，**分别用于接受连接与具体的IO处理**

```java
EventLoopGroup group = new NioEventLoopGroup();
try{
    //创建客户端启动引导类
    Bootstrap b = new Bootstrap();
    //指定线程模型
    b.group(group).
        ...
    //尝试建立连接
    ChannelFuture f = b.connect(host, port).sync();
    f.channel().closeFuture().sync();
}finally{
    group.shutdownGracefully();
}
```

### Channel（网络操作抽象类）

**Channel接口是Netty对网络操作的抽象类，通过Channel可以进行IO操作**。

一旦客户端成功连接服务器，就会新建一个Channel同该用户端进行绑定。

Channel的常用实现类为：
- NioServerSocketChannel：用于服务端
- NioSocketChannel：用于客户端

> 这两个Channel可以和BIO编程模型中的ServerSocket与Socket两个概念对应上

### EventLoop（事件循环）

EventLoop定义了Netty的核心抽象，**用于处理连接的生命周期中所发生的的事件**。
其主要作用就是**监听网络事件，并调用事件处理器进行相关IO操作的处理**

- Channel与EventLoop的关系
  - Channel为Netty网络操作的抽象类，EventLoop负责处理注册到其上的Channel的IO操作，两者配合执行IO操作
  - 且一个Channel在其生命周期内，只能注册于一个EventLoop
  - 而一个EventLoop中可以包含多个Channel
- EventloopGroup与EventLoop的关系
  - EventLoopGroup包含多个EventLoop（每一个EventLoop内部通常包含一个线程），它管理着所有EventLoop的生命周期

![](./../images/nginx/EventLoopGroup.png)



### ChannelHandler和ChannelPipeline

```java
b.group(eventLoopGroup)
    .handler(new ChannelInitializer<SocketChannel>(){
        @Override
        protected void initChannel(SocketChannel ch){
            ch.pipeline().addLast(new NettyKryoDecoder(kryoSerializer, RpcResponse.class));
            ch.pipeline().addLast(new NettyKryoEncoder(kryoSerializer, RpcRequest.class));
            ch.pipeline().addLast(new KryoClientHandler());
        }
    });
```

上面的代码就是指定序列化编码器以及自定义的ChannelHandler来处理消息的过程

可知，**ChannelHandler是消息的具体处理器，主要负责处理客户端/服务端接受和发送的数据**

ChannelPipeline为由`ChannelHandler`组成的链表，负责处理Channel上的入站和出站事件的代码

当Channel被创建时，会被自动地分配到专属的`ChannelPipeline`，且一个`Channel`包含一个`ChannelPipeline`，可以在ChannelPipeline上通过`addLast()`方法添加`ChannelHandler`。

> 应该就是**责任链模式**吧，和spring、Nginx的父子容器链的差不多

当`ChannelHandler`被添加到`ChannelPipeline`时，它得到一个`ChannelHandlerContext`，它代表一个ChannelHandler和ChannelPipeline之间的绑定。
`ChannelPipeline`通过`ChannelContext`来间接管理`ChannelHandler`

**常用方法：**
- write：将数据写到远程节点。这个数据将被传递给ChannelPipeline，并且排队，直到它被flush
- flush：将之前已写的数据冲刷到底层，进行

**典型用途：**
- 将数据从一种格式转换成另一种格式 ----- 编码器/解码器
- 异常通知 ----- exceptionCaught事件
- 提供Channel变为活动或者非活动的通知 ----- channelActive/channelInactive
- 提供用户自定义事件的通知 ---- fireUserEventTriggered。

注意：`isActive`方法对于`TCP`与`UDP`是不同的：
- TCP**只有与远程建立连接后，isActive才会被触发**
- UDP是无连接的协议，Channel一旦被打开，便激活
  - 因此，无法通过`isActive`判断UDP的另一端是否正常工作

**Channel的线程安全问题**

1. Netty中的Channel是线程安全的，因为单个Channel在其生命周期中，任何IO事件都交由EventLoop绑定的线程处理，而一个EventLoop只绑定一个线程
2. 多个线程同时获得一个Channel，都调用`writeAndFlush()`方法，此时仍然为线程安全的
   - 由于Netty的操作都是异步的，多个线程调用`writeAndFlush()`后，函数立即返回。
   - 真正开始读写操作时，一定是由单个线程执行的
3. Netty保证：多个线程消息，消息会保证按顺序发送

#### 编码器和解码器

解码器添加在**入站事件的头部**，编码器添加在出站事件的头部。天然的解决了网络数据的编解码，非常优秀的设计

编码器/解码器中覆写了`channelRead()`方法，在方法里调用`encode()`/`decode()`方法。再传递给下一个`ChannelHandler`处理

### ChannelFuture（操作执行结果）

ChannelFuture是`Future`的子类，自然可以看出，它是用于获取异步操作的结果的。
而Netty中，所有IO操作均为异步的，可以通过`ChannelFuture`接口的`addListener()`方法注册一个`ChannelFutureListener`，当操作成功或失败时，监听器自动触发，返回结果。

除此之外，还可以通过ChannelFuture的`sync()`方法**让异步操作变为同步的**

## NioEventLoopGroup默认的构造函数会起多少线程

NioEventLoopGroup的构造函数中，最终都会调用父类`MultithreadEventLoopGroup`的构造函数，其中：
- 若调用的是无参构造器，则会从`1`、`CPU核心数*2`中选出最大的，作为默认线程数
- 若在构造器中指定了线程数，则线程数为传入的int值

此外，每个`NioEventLoopGroup`对象内部都会分配一组`NioEventLoop`，其大小为`nThreads`，这样就构成了一个**线程池**，一个`NioEventLoop`与一个线程对应


## Reactor网络模型

Reactor是一种经典的线程模型，Reactor模式基于**事件驱动**，特别适合处理海量IO事件

- 单Reactor单线程（单线程Reactor）
  - 所有IO操作都由同一个NIO线程处理
  - 对系统资源的消耗小，但是无法支撑大量请求
- 单Reactor多线程（多线程Reactor）
  - Reactor负责处理网络请求，一组NIO线程（处理资源池）负责处理IO操作
- 多Reactor多线程（主从Reactor）
  - 一组NIO线程负责接受请求，另一组NIO线程负责处理IO操作

## Netty线程模型

### 单线程模型

一个线程需要执行处理所有`accept`、`read`、`decode`、`process`、`encode`、`send`事件，不适用于高并发场景

```java
//1.eventGroup既用于处理客户端连接，又负责具体的处理
EventLoopGroup eventGroup = new NioEventLoopGroup(1);
//不重要
ServerBootstrap b = new ServerBootstrap();
bootstrap.group(eventGroup, eventGroup);
```

### 多线程模型

一个Acceptor线程来负责监听客户端的连接`accept`，一个NIO线程池负责处理剩下的事件

> 和单Reactor多线程差不多嘛


```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
try{
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
    //...
}
```


### 主从多线程模型

从一个**主线程NIO线程池**中选择一个线程作为Acceptor线程，绑定监听端口，接收客户端的连接请求，其他线程负责后续的接入认证等工作。

连接建立完成后，Sub NIO线程池负责具体处理IO读写。

> 也是和主从Reactor模型差不多，主Reactor负责监听端口，通过accept方法建立连接后，把连接交给从Reactor，让其进行后续的工作

```java
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
try{
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
    //...
}
```

## Netty客户端和服务端的启动过程

### 服务端

```java
    // 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
    EventLoopGroup bossGroup = new NioEventLoopGroup(1);
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
        //2.创建服务端启动引导/辅助类：ServerBootstrap
        ServerBootstrap b = new ServerBootstrap();
        //3.给引导类配置两大线程组,确定了线程模型
        b.group(bossGroup, workerGroup)
                // (非必备)打印日志
                .handler(new LoggingHandler(LogLevel.INFO))
                // 4.指定 IO 模型
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) {
                        ChannelPipeline p = ch.pipeline();
                        //5.可以自定义客户端消息的业务处理逻辑
                        p.addLast(new HelloServerHandler());
                    }
                });
        // 6.绑定端口,调用 sync 方法阻塞知道绑定完成
        ChannelFuture f = b.bind(port).sync();
        // 7.阻塞等待直到服务器Channel关闭(closeFuture()方法获取Channel 的CloseFuture对象,然后调用sync()方法)
        f.channel().closeFuture().sync();
    } finally {
        //8.优雅关闭相关线程组资源
        bossGroup.shutdownGracefully();
        workerGroup.shutdownGracefully();
    }
```

1. 创建两个`NioEventLoop`对象实例：
   - bossGroup：用于处理客户端的TCP连接请求
   - workerGroup ：负责每一条连接的具体读写数据的处理逻辑，真正负责IO读写操作，并交由对应的Handler处理
     - 一般，bossGroup的线程数为1，workerGroup的线程数为`2*CPU核心数`，即单Reactor多线程模型

2. 创建一个服务端启动引导类ServerBootstrap，用于引导服务端的启动
3. 通过`.group()`方法给引导类ServerBootstrap配置两大线程组，确定了线程模型
4. 通过`channel()`方法给引导类指定IO模型为NIO
   - NioServerSocketChannel：指定服务端的IO模型为NIO
   - NioSocketChannel：指定客户端的IO模型为NIO 

5. 通过`.childHandler()`为引导类创建一个`ChannelInitializer`，然后制定了服务端消息的业务处理逻辑`HelloServerHandler`对象（这里是用自定义的Handler演示）
6. 调用`ServetBootstrap`的`bind()`方法绑定端口

### 客户端
 
```java
    //1.创建一个 NioEventLoopGroup 对象实例
    EventLoopGroup group = new NioEventLoopGroup();
    try {
        //2.创建客户端启动引导/辅助类：Bootstrap
        Bootstrap b = new Bootstrap();
        //3.指定线程组
        b.group(group)
                //4.指定 IO 模型
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        ChannelPipeline p = ch.pipeline();
                        // 5.这里可以自定义消息的业务处理逻辑
                        p.addLast(new HelloClientHandler(message));
                    }
                });
        // 6.尝试建立连接
        ChannelFuture f = b.connect(host, port).sync();
        // 7.等待连接关闭（阻塞，直到Channel关闭）
        f.channel().closeFuture().sync();
    } finally {
        group.shutdownGracefully();
    }
```

大体上与服务端的创建过程类似，只是最后调用的是`Bootstrap`的`connect()`方法，且指定了IP地址与端口号作为参数。
其返回值是一个`Future`类型的对象，说明这个方法是异步的，可以通过`addListener`方法监听连接是否成功，进而打印出连接信息

## 什么是TCP粘包/拆包，有什么解决方案

TCP粘包/拆包即基于TCP发送数据是，出现了多个字符串“粘”在了一起，或者一个长字符串被“拆”开来的情况。

粘包/拆包的原因：
- 应用程序写入的数据**大于套接字缓冲区的大小，导致拆包**
- 应用程序写入的数据**小于套接字缓冲区的大小**，网卡会**将应用多次写入的数据一起发到网络上，导致粘包**
- 进行MSS（最大报文长度）大小的TCP分段，当**TCP报文长度 - TCP首部长度 > MSS时，将发生拆包**
- **接收方法不及时读取套接字缓冲区数据，这将发生粘包**

本质原因：
- 发送方的原因：
  - TCP默认会使用`Nagle`算法，其主要任务为：
    1. 只有上一个分组得到确认，才会发送下一个分组；
    2. **收集多个小分组，在一个确认到来时一起发送**。
    
  - 所以，正是Nagle算法造成了发送方有可能造成粘包现象。

- 接收方的原因：
  - TCP接收方采用**缓存**方式读取数据包，一次性读取多个缓存中的数据包。自然出现前一个数据包的尾和后一个收据包的头粘到一起。

解决方案：（以使用Netty自带的解码器为例）
 1. ==定长消息==（`FixedLengthFrameDecoder`）
   - 基于固定长度的数据包实现，**按照指定的长度，对消息进行相应的拆包**，如果不够指定的长度，则进行补全
   > 如果包内容超过指定字节数，则又需要进行拆包，并在接收方组装，需要增加额外的处理逻辑
   
 2. ==利用特殊标志作为数据包的结束标志==（`LineBasedFrameDecoder`）
   - 发送端发送数据包时，每个数据包之间**以特殊标志（例如换行符）作为分隔符**。
   - 解码时通过遍历`ByteBuf` 中的可读字节，当读取到指定的特殊标志时，说明到了包尾
   > 需要对结束标志符进行转义或其他特殊处理，防止协议数据包部分出现了设定的包结束标志，导致错误的判断
 3. ==在包头设定长度字段==（`LengthFieldBasedFrameDecoder`）
   - 基于长度字段的解码器，**在消息头中定义长度字段，来标识消息的总长度**，即==包头+包体的方式==
   - 接收方解包时，首先尝试获取包头以及其中的长度字段，如果连包头都被拆分了，就继续拼接后续的数据，并根据包头的数据向后取包体的数据


## LengthFieldBasedFrameDecoder的参数

1. maxFrameLength - 发送的数据帧的最大长度
2. lengthFieldOffset - 定义长度域位于发送的字节数组中的下标。
   - 换句话说：发送的字节数组中下标为`${lengthFieldOffset}`的地方，就是长度域的开始地方
3. lengthFieldLength - 用于描述定义的长度域的长度。
   - 换句话说：发送字节数组`bytes`时, 字节数组`bytes[lengthFieldOffset, lengthFieldOffset+lengthFieldLength]`域对应于的定义长度域部分
4. lengthAdjustment - 满足公式: 发送的字节数组`bytes.length - lengthFieldLength = bytes[lengthFieldOffset, lengthFieldOffset+lengthFieldLength] + lengthFieldOffset + lengthAdjustment`
5. initialBytesToStrip - 接收到的发送数据包，去除前`initialBytesToStrip`位
6. failFast - 
   - true: 读取到长度域超过maxFrameLength，就抛出一个 TooLongFrameException。
   - false: 只有真正读取完长度域的值表示的字节之后，才会抛出 TooLongFrameException
   - 默认情况下设置为true，建议不要修改，否则可能会造成内存溢出 
7. ByteOrder - 数据存储采用大端模式或小端模式

## Netty长连接、心跳机制

TCP在进行读写前，需要通过三步握手建立连接，释放/关闭连接时，通过四步挥手释放连接，这个过程是比较消耗网络资源且由时间延迟的

> HTTP针对这个问题，在HTTP1.1实现了长连接和请求的流水线处理机制

- 短连接：Server和Client建立连接后，读写完成后就关闭连接，如果下一次要互相发送消息，就重新连接
  - 优点：易于管理
  - 缺点：每次读写都会浪费大量网络资源，用于建立、关闭连接

- 长连接：Server和Client建立连接后，即使完成了一次读写，也不主动关闭连接，后续的读写操作继续使用这个连接。
  - 省去了很多TCP建立与关闭的操作，降低了对网络资源的依赖，适用于频繁请求资源的客户

**心跳机制**：在TCP的长连接中，Client不会主动断开连接，如果出现断网等网络异常，Client和Server没有交互的话，就不会发现对方已经断线，导致占用了网络资源，也无法成功连接。
为了解决这个问题，引入了心跳机制。

- 心跳机制的工作原理：
  - Server和Client之间，在一定时间内都没有数据交互，Client或Server会发送一个**特殊的数据包**给对方。
  - 当对方接收到这个数据包之后，也**立即发送一个对应的数据包**，视为回应发送方，此即为一个`PING-PONG`交互。
  - **当某一端收到心跳消息后，就知道了对方仍然在线，确保了TCP连接的有效性**

> 实际上，TCP自身就带有长连接选项，本身也是有心跳包机制的，即TCP选项中的`SO_KEEPALIVE`，但是TCP协议层面的长连接灵活性较差，所以通常都是在应用层协议层面，实现自定义的心跳机制。

## Netty零拷贝

零拷贝的介绍在消息队列里详细说了，主要指的就是在进行IO时，避免了CPU的调用，实现0CPU调用的IO，即避免数据在**用户态**与**内核态**之间的来回拷贝。
在Netty层面，零拷贝主要体现在对数据操作的优化：

1. 使用Netty提供的CompositeByteBuf类，可以将多个`ByteBuf`合并为一个逻辑上的`ByteBuf`，避免了各个`ByteBuf`之间的拷贝
2. `ByteBuf`支持slice操作，可以将`ByteBuf`分解为多个**共享一个存储区域**的ByteBuf，避免了**内存的拷贝**
3. 通过`FileRegion`包装的`FileChannel.transferTo`实现文件传输，可以直接将文件缓冲区的数据发送到目标`Channel`，避免了传统的通过while循环的方式导致的**内存拷贝**



## Netty 内存管理是怎么做的？

Netty底层通过使用Direct Memory，减少了内核态与用户态之间的内存拷贝，加快了IO速率。
但是频繁的向系统申请Direct Memory，并在使用完成后释放本身就是一件影响性能的事情。
为此，Netty内部实现了一套自己的内存管理机制：
   - 在申请时，Netty会一次性向操作系统申请较大的一块内存，然后再将大内存进行管理，按需拆分成小块分配。
   - 释放时，Netty并不着急直接释放内存，而是将内存回收以待下次使用。





## Nety三大组件

**Buffer缓冲区**

**Buffer最终的实现是一个内存中的字节数组，所以一个Buffer本质上是内存中的一块区域**，我们可以将数据写入这块内存，之后从这块内存获取数据。

> Buffer与Channel交互，Channel向Buffer写入数据，从Buffer读取数据

> 传统IO模式中，是从Stream流中逐字节地读取数据的，且只能顺序读取，不能回溯

**Channel通道**

Channel是对传统IO包中的Stream的模拟，通过Channel这个对象，我们可以读取和写入数据，并且通过Channel的所有发送数据都要读到缓冲区Buffer中，所有要接收的数据都要写到缓冲区Buffer里。

Channel与Stream的区别：
- 流是单向的，读写分离，通道是双向的，可读可写
- 流读写是阻塞的，通道可以异步读写
- 流中的数据可以选择性的先读到缓存中，通道的数据总是要先读到一个缓存中，或从缓存中写入

**Selector选择器**

它做到了一个线程管理多个Channel：可以向Selector注册感兴趣的事件，当事件就绪，通过 `Selector.select()`方法获取注册的事件，进行相应的操作。

Selector通过**事件循环**的机制来处理读写事件：
1. 首先你需要把你的连接，也就是 `Channel` 绑定到 Selector 上
2. 在接收数据的线程来调用 Selector.select() 方法来等待数据到来
   - select是阻塞方法，会阻塞线程
3. Selector.select()方法的返回值是一个迭代器，可以从这个迭代器里面获取所有 Channel 收到的数据，然后执行业务逻辑 

> Select的底层是基于epoll()实现的，自然是IO多路复用

# Nginx

## Nginx是啥

Nginx和tomcat是一个WEB服务器，它是一个轻量级、占用内存少、启动快、并发能力强的服务器。

> WEB服务器：负责处理和响应用户请求，也称为HTTP服务器

## Nginx的特点

- 内存占用小
  - 10000万HTTP Keep-Alive连接仅占用2.5MB内存
- 高并发
  - 支持10万以上的并发
- 跨平台
- 扩展性好
- 安装简单
- 开源

## Nginx的用途

### 静态资源服务器

Nginx是一个HTTP服务器，可以将服务器上的静态文件（HTML、图片等）通过HTTP协议展现给客户端，即作为静态资源Web服务器。

```nginx
server {
	// 监听的端口号
	listen 80;
	// server 名称
	server_name  localhost;

	// 匹配 api，将所有 :80/api 的请求指到指定文件夹
	location /api {
		root   /mnt/web/;
		// 默认打开 index.html
		index  index.html index.htm;
	}
}
```


### 动静分离

将**动态请求**与**静态请求**分离，而不是把动态页面与静态页面分离。
可以理解为**Nginx处理静态页面，Tomcat或其他web服务器处理动态页面**，从而减轻Web服务器的压力，提高网页响应速度

- 静态页面：一个页面对应一个内容，也就是一对一的关系，在互联网架构中，页面几乎为不变的或者是页面发生变化频率较低的。
  - 静态页面的特点：
    - 每个网页都有一个固定的 URL，且网页URL以.htm、.html、.shtml等常见形式为后缀，而不含有 `？`
    - 网页内容发布到网站服务器上。无论是否有用户访问，每个静态网页的内容都将保存在网站服务器上，也就是说，保存在服务器上的文件，每个网页都是一个独立的文件
    - 内容相对稳定，容易被搜索引擎所检索；
    - 没数据库的支持，网站制作和维护方面工作量大，当网站信息量很大时，完全依靠静态网页制作方式较困难；
    - 交互性较差，功能方面有较大的限制；
    - 运行数据快；
- 动态页面：是一对多访问，通过一个页面可以根据若干参数返回其不同的数据，在互联网架构中，不同的用户访问不同的动态场景页面请求，都可能是不一样的页面
  - 以数据库技术为基础，可大大降低网站维护的工作量；
  - 采用动态网页技术的网站可以实现更多的功能；
  - 不是独立存在于服务器上的网页文件，只有当用户请求时服务器才返回一个完整的网页


### 反向代理

反向代理的原理：客户端将请求发送到反向代理服务器，**由反向代理服务器去选择目标服务器**，获取数据后再返回给客户端。
对外暴露的是反向代理服务器的地址，隐藏了真实服务器的IP地址，代理的是目标服务器。从而使得真实服务器对客户端不可见。

同时，可以通过反向代理实现网站的负载均衡

> 一般在处理跨域请求时比较常用。

### 正向代理

> 反向代理，代理的是目标服务器
> 正向代理，代理的是客户端

客户端通过正向代理服务器重定向请求，从而访问目标服务器，即目标服务端不知道客户端是谁，**客户端对目标服务器的这次访问是透明的**

VPN就相当于一个正向代理

### 负载均衡

在作为反向代理服务器时，Nginx可以将接收到的客户端请求以一定的规则，均匀地分配到这个服务器集群中的所有服务器上。

此外，Nginx还带有健康检查功能(服务器心跳检查)，会定期轮询向集群里的所有服务器发送健康检查请求，来检查集群中是否有服务器处于异常状态。


## Nginx负载均衡策略有哪些

- 轮询（默认）
  - 每个请求按照时间顺序，**逐一分配到不同的后端服务器**。如果后端服务器挂掉了，可以自动剔除
  - 特点：适用于服务器性能相近的集群情况，其中每个服务器承载相同的负载，但容易造成资源分配不合理的问题
- 加权随机
  - 权重越高的服务器被访问的概率越大
  - 可以应用于服务器性能不等的集群，使资源分配更合理
- 最小连接数
  - 有新的请求出现时，遍历服务器节点列表，**选取其中连接数最小的一台服务器来响应请求**。连接数可以理解为当前处理的请求数。
- IP哈希
  - 根据发出的请求和ip的hash值分配服务器，可以**保证同IP发出的请求映射到同一服务器**，或者**具有相同hash值的不同IP映射到同一服务器**
  - 在一定程度上解决了集群部署环境下，Session不共享的问题
- URL哈希
  - 根据url的hash值分配服务器，**使每个url定向到同一个后端服务器**
  - 能够提高后端缓存服务器的效率，但后端服务器宕机的时候，url_hash不会自动跳转的其他缓存服务器，而是返回给用户一个503错误
- 一致性哈希
  - 一致性哈希算法和redis cluster的哈希算法类似
  - 根据请求中某一些数据（例如MAC地址、IP地址或其他协议中的信息）作为特征值来计算需要落在的节点上，算法一般会保证同一个特征值每次都一定落在相同的服务器上。
> 如果有接受请求的服务器挂了，并且请求超过最大失败次数，则在失效时间的窗口内，不会再将请求转发到该节点

Nginx的**平滑加权轮询算法**:

![](https://images-shf.oss-cn-beijing.aliyuncs.com/dubbo/%E5%B9%B3%E6%BB%91%E8%BD%AE%E8%AF%A2%E7%AE%97%E6%B3%95.png?versionId=CAEQHhiBgICXjaCE.RciIDYwMDAyNTlmODliODQzYTdhZjUxMDk0NWIyYTc2MDVm)

如果不使用平滑算法，对${A:3, B:2, C:1}$的权重设置，会出现AAABBC的调用方式，调用过于集中。而在平滑算法整个过程中，节点的流量是平衡的，会得到类似${ABACBA}$的方式，而不是${AAABBC}$


## Nginx性能优化的常见方式

- 设置Nginx**运行工作进程个数**：一般设置为CPU的核心数或2*CPU核心数
  - Nginx是单Reactor多进程的模型
- 开启**静态资源压缩**：对网页的静态资源进行压缩，提高访问速度，优化Nginx性能
- 设置**本地缓存**：像图片、CSS、Js等基本不会修改的文件，完全可以将其设置在浏览器本地缓存，从而降低Nginx的压力
- 设置单个worker进程允许的客户端最大连接数
- 设置超时时间设置：避免在无用连接上消耗太多资源

## Nginx怎么实现后端服务健康检查

可以通过第三方插件`upstream_check_module`来检测后端服务的健康状态，如果后端服务器不可用，则所有的请求不转发到这台服务器。


## 如何保证Nginx服务的高可用

可以结合Keepalived来实现高可用。

> Keepalived是一个开源路由软件，是一个轻量级的负载均衡与高可用解决方案。
> Keepalived的负载均衡依赖于Linux虚拟服务器内核模块
> 且实现了一组检查器，用于根据服务器节点的健康状况动态维护和管理服务器集群

实现方法：
1. 准备2台安装并配置了Keepalived的Nginx服务器，分别为主与备
2. 为两台Nginx服务器绑定同一个虚拟IP
3. 编写Nginx检查脚本，用于通过Keepalived检查Nginx主服务器的状态是否正常

## Nginx总体架构

> 对于传统的 HTTP 和反向代理服务器而言，在处理并发请求的时候会使用单进程或单线程的模式处理，同时会止网络或输入/输出操作。
> 这种方式会消耗大量的内存和 CPU 资源。因为每产生一个单独的进程或线程需要准备一套新的运行时环境，包括分配堆和堆栈内存，以及创建新的执行上下文。
> 
> 由于上面的原因，Nginx 在设计之初就使用了模块化、事件驱动、异步处理，非阻塞的架构。

![](./../images/nginx/总体架构.png)

在Nginx中，包含两类进程：
- Master
  - 负责加载分析配置文件、启动/管理 Worker 进程以及平滑升级
  - 一个Master管理多个Worker
- Worker
  - 负责处理并响应用户的HTTP/HTTPS请求
  - 因此，Nginx是一个**多Reactor多进程**的系统

其中，Worker线程通过IO多路复用的技术处理网络请求（即IO请求），包含kevent/epoll/select等手段；
而对于与本地数据磁盘的通信，Worker采用的是`Advanced IO`、`AIO`、`sendfile`等机制

Worker实现了模块化的设计，其内部包含核心模块与功能性模块：
- 核心模块：负责维持一个运行循环（`run-loop`），执行网络请求处理的不同阶段的模块功能
  - 如网络读写、存储读写、内容传输、外出过滤，以及将请求发往上游服务器等
- 功能性模块：围绕核心模块构建，负责实现具体的请求处理功能
  

可见， **Nginx 不会为每个连接生成一个进程或线程，而是通过 Worker 进程使用多路复用的方式处理多个请求**。
在每个Worker内部都维持了一个高效的运行循环run-loop，通过共享监听套接字的方式接受新请求并处理。
run-loop很大程度上依赖于异步任务处理，通过模块化，事件通知，回调函数和定时器来支撑异步操作的实现。其目的是为了实现高并发请求下的不阻塞（尽可能不阻赛）。 


## Nginx进程模型

- Master 进程：它作为父进程会在 Nginx 初始化的时候生成并启动，其他的进程都是它子进程，**Master 进程对其他进行进行创建和管理**。
- Worker 进程：是 Master 的子进程，负责**处理网络请求**。
  - 这里需要说明一下 Nginx 为什么采用了多进程而不是多线程的结构，其原因是为了保证**高可用性**，进程不像线程那样共享地址空间，也**避免了当一个线程中的第三方模块出错引而影响其他其他线程的情况发生**。
- `Cache Manager` 和 `Cache Loader` 进程：Cache Loader 负责**缓存载入**，Cache Manager 负责**缓存管理**。每一个请求所使用的缓存还是由 Worker 来决定的，而**进程间通信都是通过共享内存来实现的**。

> 注意，在上述进程中，**只有Worker进程是多进程**，这是因为 Nginx 采用了事件驱动的模型。
> Worker的进程数取决于应用场景：
> - CPU密集型请求：Worker数等于CPU核心数
> - IO密集型请求：Worker数是CPU核心数的1-2倍

## Nginx怎么处理HTTP请求

![](./../images/nginx/HTTP处理.png)

如图所示，描绘了 Master 创建 Listen 以及 fork 出 Worker 的过程，以及客户端请求和 Worker 响应请求的过程。
1. Master 进程创建以后，通过 `socket` 方法创建 socket 的 IO 通道。（master的红色箭头），并通过`bind()`方法将master进程与监听器 `listen` 进行绑定，然后通过 `fork` 方法 `fork` 出多个 `Worker` 进程（绿色虚线）。 
2. 在每个 Worker 进程中，通过`accept`方法监听 `socket` 请求，一旦 `listen` 监听到 `socket` 请求，Worker 进程就可以通过 `accept` 接受请求。
3. 在最下面的client模块中，当 client 通过 `connect` 方法与 Nginx 发生连接时，所有执行 `accept` 方法的 Worker 进程都会接受到来自 listen 的通知，但是**只有一个 Worker 进程能够成功 accept 到，其他的进程则会失败**。
   - ==这里 Nginx 提供了一把互斥锁 `accept_mutex` 来保证同一时刻只有一个 Worker 进程在 accept 连接，从而解决惊群问题==
4. 当 Worker 进程 `accept` 到 socket 请求后，client 会通过 `send` 方法发送请求（绿色虚线）给 Worker。
  而 Worker 使用 `recv` 方法接受请求，同时通过 `parse（解析）、process（处理）、generate（生成响应）`几个步骤将返回的响应通过 `send` 方法传送给 client，而 client 会使用 `recv` 方法接受响应。
5. 最后 Worker 调用 `close` 方法断开和 client 的连接，同时在时间循环中继续accept，等待下一次请求


> 惊群问题：在多线程（或多进程）场景下，有多个线程在等待某一资源可用，一旦这个资源可用，那么所有等待这个资源的线程都会被唤醒，但是资源只有一份，那么只有一个线程获得这个资源，其它线程都获取失败。
> **导致了不必要的线程唤醒，因为只有一个线程能获得资源，理想情况下，只要有一个线程被唤醒即可，而唤醒多个线程导致了不必要的调度开销**
> 
> 在老版本的`accept`方法中，存在惊群问题，但现在已经解决了，但`epoll`上还存在着惊群问题。
> - 考虑一个场景，父进程创建了一个监听描述符，通过`epoll_ctl`将该监听套接字加入到epoll中，然后fork出多个子进程，每个子进程都在epoll_wait中阻塞等待监听描述符的可读事件（新连接到来），当有新连接到来时，所有子进程都会被唤醒。
> 
> Nginx通过互斥锁实现同一时刻只有一个Worker进程能监听Web端口，即通过`accept`尝试获取连接

## 怎么解析一个域名，需要哪些信息

Nginx的域名解析是基于其配置文件Nginx.conf实现的

为了不阻塞当前线程，Nginx采用了异步的方式进行域名查询。整个查询过程主要分为三个步骤，这点在各种异步处理时都是一样的：

1. 准备函数调用需要的信息，并设置回调方法
2. 调用函数
3. 处理结束后回调方法被调用

另外，为了尽量减少查询花费的时间，Nginx还对查询结果做了本地缓存。为了初始化DNS Server地址和本地缓存等信息，需要在真正查询前需要先进行一些全局的初始化操作。

**域名解析的具体流程如下：**

1. 初始化域名查询所需要的的全局信息
   需要初始化的全局信息包括：
    1. DNS服务器的地址，如果指定了多个服务器，nginx会采用`Round Robin`的方式轮流查询每个服务器
       - DNS服务器的信息需要在配置文件中明确指出（`resolver 8.8.8.8`） 
    2. 对查询结果的缓存，采用红黑树的数据结构，以要查询名字的`Hash`作为Key, 节点信息存放在 `struct ngx_resolver_node_t`中。 
2. 准备本次查询
   对要查询的域名进行解析，如果已经是IP地址了，就不用发DNS请求了
3. 根据域名进行IP地址查询
   Nginx会先检查本地缓存，命中则更新过期时间并回调Handler，完成查询；如果未命中，则发送请求给DNS Server
4. 查询完成后回调Handler
   真正的DNS查询完成后，不管成功，失败或是超时，nginx会回调相应查询的handler

![](./../images/nginx/域名解析.png)
其中，颜色越深则花费时间越长

总而言之，域名解析流程可以划分为三步：
1. 判断传入的是否为IPV4地址，是则直接调用Handler
2. 检查是否在本地缓存，是则直接调用Handler
3. 发送DNS请求，收到回复后调用Handler

## 有一大批流量总是打到一个实例，这个实例的兄弟实例分到的很少，怎么办

可以调整负载均衡策略，如果是基于hash的负载均衡，可以设置为基于权重的负载均衡，或轮询的负载均衡

# Tomcat

## Http服务器是啥

HTTP Server本质上也是一种应用程序——它通常运行在服务器之上，**绑定服务器的IP地址并监听某一个tcp端口来接收并处理HTTP请求**，这样客户端就能够通过HTTP协议来获取服务器上的网页（HTML格式）、文档（PDF格式）、音频（MP4格式）、视频（MOV格式）等等资源

### Application Server

Application Server 是一个应用执行的服务器，它首先需要支持开发语言的 Runtime，保证应用能够在应用服务器上正常运行需要支持应用相关的规范，例如类库、安全方面的特性。
与HTTP Server相比，Application Server能够动态的生成资源并返回到客户端。

## 介绍一下Tomcat

Tomcat就是一个Application Servr，需要提供 JSP/Sevlet 运行需要的标准类库、Interface等。但为了方便，应用服务器往往也会集成 HTTP Server 的功能。

Tomcat运行在JVM之上，它和HTTP服务器一样，绑定IP地址并监听TCP端口，同时还包含以下指责：
1. 管理`Servlet`程序的生命周期；
2. 将URL映射到指定的`Servlet`进行处理；
3. 与Servlet程序合作，处理HTTP请求：根据HTTP请求生成`HttpServletRequest/Response`对象并传递给`Servlet`进行处理，将`Servlet`中的`HttpServletResponse`对象生成的内容返回给浏览器；


## Tomcat的结构

![](./../images/nginx/tomcat结构.png)

Tomcat的核心组件：
- 连接器Connector：接受浏览器发来的TCP连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，并产生一个线程，来处理这个请求，并把产生的 Request 和 Response 对象传给处理这个请求的线程，即Container的线程
  - Tomcat默认的是`HttpConnector`，负责根据收到的`Http`请求报文生成`Request`对象和`Response`对象

- 容器Container：Container是容器的父接口，所有子容器都必须实现这个接口，简单来说就是服务器部署的项目是运行在Container中的。
  - Container里面的项目获取到`Connector`传递过来对应的的`Request`对象和`Response`对象进行相应的操作。

> Container 容器的设计用的是典型的责任链的设计模式

## Tomcat是如何提供服务的？

1. 创建一个`request`对象，并填充那些**有可能被所引用的Servlet使用的信息**，如参数，头部、cookies、查询字符串等。
2. 创建一个`response`对象，所引用的`servlet`使用它来给客户端发送响应。
3. 调用`servlet`的`service`方法，并传入request和response对象。
  这里servlet会**从request对象取值，给response写值**。
4. **根据servlet返回的response生成相应的HTTP响应报文**。

## Tomcat架构

![](./../images/nginx/tomcat架构.png)

- Server(服务器)是Tomcat构成的顶级构成元素，所有一切均包含在Server中，Server的实现类StandardServer可以包含一个到多个Services;
- 次顶级元素`Service`的实现类为`StandardService`，调用容器(Container)的接口，实际上是调用了`Servlet Engine`(引擎)；
- 接下来次级的构成元素就是`容器(Container)`。
  - `主机(Host)`、`上下文(Context)`和`引擎(Engine)`均继承自Container接口，所以它们都是容器。
  - 但是，它们是有父子关系的，其中，引擎是顶级容器，其直接包含主机容器，而主机容器又包含上下文容器
- 连接器(Connector)将`Service`和`Container`连接起来，首先它需要注册到一个Service，它的作用就是**把来自客户端的请求转发到Container(容器)**

## Tomcat是怎样处理HTTP请求的

假设来自客户的请求为：`http://localhost:8080/test/index.jsp`

1. Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应；

2. `Engine`获得请求`localhost:8080/test/index.jsp`，匹配它的所有虚拟主机`Host`；

3. Engine匹配到名为`localhost`的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）；

4. `localhost Host`获得请求`/test/index.jsp`，匹配它所拥有的所有`Context`；

5. Host匹配到路径为`/test`的`Context`（如果匹配不到就把该请求交给路径名为""的Context去处理）；

6. `path="/test"`的`Context`获得请求`/index.jsp`，在它的`mapping table`中寻找对应的servlet；
  
7. Context匹配到`URL PATTERN`为`*.jsp`的`servlet`，对应于JspServlet类；

8. 构造`HttpServletRequest`对象和`HttpServletResponse`对象，作为参数调用`JspServlet`的`doGet`或`doPost`方法；

9. Context把执行完了之后的HttpServletResponse对象返回给Host；

10. Host把HttpServletResponse对象返回给Engine；

11. Engine把HttpServletResponse对象返回给Connector；

12. Connector把HttpServletResponse对象返回给客户browser；




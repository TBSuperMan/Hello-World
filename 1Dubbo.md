# Dubbo基础知识

## 既然有HTTP了，为什么还要设计RPC

> 首先要明确一点，使用RPC并不是是因为HTTP的连接建立与断开的过程会耗费更多的资源。
> HTTP协议是支持连接池复用的，也就是建立一定数量的连接不断开，并不会频繁地创建与销毁连接，而且HTTP也可以使用protobuf这种二进制编码协议对传输内容进行编码，因此二者最大的区别还是在传输协议上

通用定义的http1.1协议的tcp报文包含太多废信息，即时传输的内容使用了二进制编码，但HTTP Head的数据也是必须用文本编码的，非常占用字节数（不过这个在HTTP2.0被优化了）

除此之外，HTTP2.0也对HTTP首部的信息进行了优化，但还是会有传输首部的开销

> HTTP2.0在客户端和服务器端使用“**首部表**”来跟踪和存储之前发送的键值对，对于相同的数据，不再通过每次请求和响应发送；

除此之外，**良好的rpc调用是面向服务的封装，针对服务的可用性和效率等都做了优化。单纯使用http调用则缺少了这些特性**

> RPC是一个任务，而HTTP是一种具体的协议，两者不应当相提并论

## 为什么要用Dubbo

dubbo是⼀个分布式服务框架，提供⾼性能和透明化的RPC远程服务调⽤⽅案，以及SOA服务治理方案。说白了其实dubbo就是一个远程调用的分布式框架。

Dubbo是一款优秀的RPC框架，除了核心的RPC服务外，它还提供了以下服务：
- 负载均衡
- 服务调用链路生成：随着系统的发展，服务器之间的调用会变得非常复杂，而Dubbo可以描述服务间是如何调用的
- 服务访问压力以及时长统计、资源调度与治理

## Dubbo架构的核心角色

![](./../images/dubbo/核心角色.jpg)

图中：蓝色的块表示业务有交互，绿色的块表示只是Dubbo的内部交互

- Container：服务运行的容器，负责加载、运行服务提供者，是必不可少的
- Provider：服务提供方，向注册中心注册自己的服务
- Consumer：服务的调用方，向注册中心订阅自己的服务
- Registry：注册中心
- Monitor：统计服务的调用次数和调用时间的监控中心。服务消费者和提供者会定时发送统计数据到监控中心。 非必须

## Dubbo的Invoker的原理

**Invoker 就是 Dubbo 对远程调用的抽象**

![](./../images/dubbo/invoker.jpg)

Invoker可以划分为：服务提供Invoker、服务消费Invoker

## Dubbo的工作原理

Dubbo的整体设计，可以划分为十层，各层为单向依赖

![](./../images/dubbo/十层架构.jpg)

1. Service服务接口层：
   - 该层是与实际业务逻辑相关的，根据服务提供方和服务消费方的业务设计对应的接口和实现。
2. Config配置层：
   - 对外的配置接口，支持代码配置，同时也支持基于 Spring 来做配置，
   > 以 ServiceConfig, ReferenceConfig 为中心
3. Proxy服务代理层：
   - 服务接口的透明代理，**是调用远程方法像调用本地的方法一样简单的关键角色**，真实调用过程依赖代理类
   > 以 ServiceProxy 为中心
4. registry注册中心层：
   - 封装服务地址的注册与发现
   > 以服务URL为中心，扩展接口为`RegistryFactory`、`Registry`和`RegistryService`。
   - 如果没有服务注册中心，此时服务提供方直接暴露服务。
5. Cluster路由层
   - 封装多个提供者的路由及负载均衡，并桥接注册中心。
    > 以 `Invoker` 为中心，扩展接口为`Cluster`、`Directory`、`Router`和`LoadBalance`。
   - **将多个服务提供方组合为一个服务提供方，实现对服务消费方来透明**，只需要与一个服务提供方进行交互
6. Monitor监控层
   - 进行RPC 调用次数和调用时间监控
    > 以 Statistics 为中心，扩展接口为MonitorFactory、Monitor和MonitorService。
7. Protocol远程调用层
   - 封装 RPC 调用，
   > 以 Invocation, Result 为中心，扩展接口为Protocol、Invoker、Exporter
   - Protocol是**服务域**，它是Invoker暴露和引用的主功能入口，它负责Invoker的生命周期管理。
   - Invoker是**实体域**，它是Dubbo的核心模型，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起invoke调用，它有可能是一个本地的实现，也可能是一个远程的实现，也可能一个集群实现。
8. Exchange信息交换层
   - 封装请求响应模式，同步转异。
   > 以 Request, Response 为中心，扩展接口为Exchanger、ExchangeChannel、ExchangeClient和ExchangeServer
9. Transport网络传输层
   - 抽象 mina 和 netty 为统一接口
   > 以 Message 为中心，扩展接口为Channel、Transporter、Client、Server和Codec。
10. Serialize数据序列化层
   - 对需要在网络传输的数据进行序列化，包含可复用的一些工具
   > 扩展接口为Serialization、 ObjectInput、ObjectOutput和ThreadPool。 

对于上述各层的关系，Dubbo官方的描述如下：
- 在RPC中，`Protocol`是核心层，也就是只要有`Protocol + Invoker + Exporter`，就可以完成非透明的RPC调用，然后在Invoker的主过程上Filter拦截点
- Cluster是一个外围概念，其目的为将多个Invoker伪装为一个Invoker，从而使得其他人**只要关注Protocol层Invoker即可**
  - 加上Cluster或者去掉Cluster对其它层都不会造成影响，因为只有一个提供者时，是不需要Cluster的
- Proxy层封装了所有接口的透明化代理，而在其它层都以`Invoker`为中心，只有到了暴露给用户使用时，才用`Proxy`将`Invoker`转成`对外暴露的接口`，或将`接口`实现转成`Invoker`
  - 也就是去掉Proxy层RPC是可以Run的，只是不那么透明，不那么看起来像调本地服务一样调远程服务。
- Registry和Monitor实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在一起。


## Dubbo工作流程

1. Provider向注册中心注册自己的服务
2. Consumer向注册中心订阅服务，注册中心会通知Consumer注册好的服务
3. Consumer调用Provider提供的服务
4. Consumer与Provider异步地通知监控中心

## Dubbo服务暴露的流程

服务暴露的本质：
1. Dubbo的启动
   - 由监听器`DubboBootstrapApplicationListener`对`ApplicationContext`的`refresh()`方法的监听触发，当`refresh()`完成后，会调用`dubboBootstrap.start()`，启动dubbo
2. 对Dubbo配置文件进行校验与自动更新，并进行服务暴露的前置Check（例如注册中心是否有效之类的）
   - 主要是对ServiceConfig对象进行配置的校验与更新 
4. 参数解析（注册中心URL、协议、host、port）
   1. 加载配置中的注册中心的URL，获取当前服务需要注册到的注册中心的列表，并解析服务需要通过哪些协议进行暴露。
      - 服务暴露需要用到各种参数，用于构建后续的服务暴露URL，这里会使用`HashMap`存储。 
      - 参数组装完毕，解析出服务暴露的host和port，然后构建URL
5. **本地暴露**
   - Provider启动，首先将想要提供的服务先暴露在本地（也可以通过配置文件指定是暴露到远程还是暴露到本地，或同时暴露），Dubbo将服务注册到本地的`ServiceRepository`，从而在自身引用自己的服务时，直接引用本地的服务
   > 本地暴露虽然不走网络传输，但是还是会被Filter和Listener处理 
6. **远程暴露**（生成Invoker）
   1. 远程暴露前，首先将服务实现类封装为`Invoker`，Invoker会根据methodName调用具体方法
      - 调用方法的方式有两种：1. 利用Java的**反射**调用；2.利用字节码技术**动态生成代理对象**；
      - Dubbo默认选择第二种，即动态生成代理对象的方式，根据方法名和参数直接调用对应方法，避免反射带来的性能问题
   2. 根据之前解析得到的协议来暴露服务，例如将服务注册到注册中心
      - 需要从URL中分离得到注册中心的URL与服务暴露的真实URL 
7. 通过`doOpen()`开启服务，并通过Netty实现本地端口的监听，即监听网络请求
8. 完成服务暴露后，将服务注册到注册中心，让Consumer可以感知到
    - 暴露是指开始监听本地端口，对外提供服务，而注册指的是将服务注册到注册中心

> 在上述过程中，还会完成Filter链与服务取消监听器的创建

## Dubbo服务引用的流程

服务引用的大致流程：
- 首先会对相关的配置进行检查和更新，然后引用服务创建Invoker，最终基于Invoker创建代理对象，从而利用Invoker调用指定服务
- 如果引用的是注册于注册中心的远程服务，会去订阅相关服务，然后得到对应的Invoker，例如DubboInvoker。
- DubboInvoker创建时会自动创建`NettyClient`和服务端建立连接，`DubboInvoker`在执行invoke调用时，会**轮训NettyClient发送网络请求**

## Dubbo服务调用的流程


## 关于 Dubbo 服务保障手段的问题（服务方和消费方如何优雅退出、RPC 的熔断机制、降级、限流）



## Dubbo的注册中心只能是ZK吗？（redis理论上也行/）

不一定，也可以是Eureka、Nacos


## Dubbo的SPI机制，如何扩展Dubbo中的默认实现

`SPI（Service Provider Interface）`机制被大量用在开源项目中，它可以帮助我们**动态寻找服务/功能（比如负载均衡策略）的实现**。

具体原理：我们**将接口的实现类放在配置文件中**，我们在程序运行过程中**读取配置文件**，通过==反射==加载实现类。这样，我们可以在运行的时候，动态替换接口的实现类。
和 IoC 的解耦思想是类似的。

> Dubbo没有使用Java原生的SPI机制，而是对其进行了增强

如果说我们想要实现自己的负载均衡策略，可以创建对应的实现类 `XxxLoadBalance` 实现 `LoadBalance` 接口或者 `AbstractLoadBalance` 类。
并将实现类的路径写入dubbo的对应的配置文件中即可。

## Dubbo为什么不适用Java原生的SPI机制


## Dubbo 为什么默认用 Javassist

因为如果要在Invoker通过反射调用原始服务的方法，性能会比较差

## Dubbo的微内核机制

Dubbo 采用 微内核（Microkernel） + 插件（Plugin） 模式，简单来说就是微内核架构。微内核只负责组装插件。


> 什么是微内核架构？
> 微内核架构模式（有时被称为**插件架构模式**）是实现基于产品应用程序的一种自然模式。
> 基于产品的应用程序是已经打包好并且拥有不同版本，可作为第三方插件下载的。
> 然后，很多公司也在开发、发布自己内部商业应用像有版本号、说明及可加载插件式的应用软件（这也是这种模式的特征）。
> **微内核系统可让用户添加额外的应用如插件，到核心应用，继而提供了可扩展性和功能分离的用法**。

微内核架构包含两类组件：**核心系统（core system）** 和 **插件模块（plug-in modules）**

![](./../images/dubbo/微内核架构示意图.png)

通常情况下，微核心都会采用 Factory、IoC、OSGi 等方式管理插件生命周期。Dubbo 不想依赖 Spring 等 IoC 容器，也不想自己造一个小的 IoC 容器（过度设计），因此采用了一种最简单的 Factory 方式管理插件 ：JDK 标准的 SPI 扩展机制 `（java.util.ServiceLoader）`

## Dubbo的负载均衡策略

Dubbo默认使用`Random`随机调用，也可以自行扩展负载均衡策略

在 Dubbo 中，所有负载均衡实现类均继承自 `AbstractLoadBalance`，该类实现了 `LoadBalance` 接口，并封装了一些公共的逻辑。

通过`@Service(weight = 100)`配置权重

| 类名 | 策略名 | 具体行为 |
|--|--|--|
| `RandomLoadBalance` | 加权随机 | 默认算法，根据权重随机分配，默认权重相同。存在慢的提供者累计请求的问题 |
| `RoundRobinLoadBalance` | 加权轮询 | 借鉴于Nginx的**平滑加权轮询法**，默认权重相同 |
| `LeastActiveLoadBalance` | 最小活跃数优先 | 背后是能者多劳的思想。 |
| `ShortestResponseLoadBalance` | 最短响应优先 | 在最近的一个滑动窗口中，响应时间越短，越优先调用。使得响应速度快的服务提供者，处理更多请求 |
| `ConsistentHashLoadBalance` | 一致性Hash | 根据请求参数的hash值（第一个参数）确定服务提供者，相同参数的请求发送到同一提供者 |

> **活跃数是啥**
> 初始状态下所有服务提供者的活跃数均为 0
> 每个服务提供者的中，特定方法都对应一个活跃数，**每收到一个请求后，对应的服务提供者的活跃数 +1，当这个请求处理完之后，活跃数 -1**。
> 因此，活跃数越小的服务器，处理性能也就可以视为越强
> 如果多个服务提供者的活跃数相同，则回到`RandomLoadBalance`
> 
> 活跃数是通过 RpcStatus 中的一个 ConcurrentMap 保存的，根据 **URL 以及服务提供者被调用的方法的名称**，我们便可以获取到对应的活跃数
> 也就是说，**服务提供者中的每一个方法的活跃数都是互相独立的**


## Dubbo服务降级



## Dubbo集群容错策略

集群容错策略：

| 策略 | 名称 | 备注 |
|--|--|--|
| Failover Cluster | 失败重试（缺省值） | 当出现失败，重试其他服务器，默认重试两次，使用retries配置。**一般用于幂等的读操作** |
| Failfast Cluster | 快速失败 | 只发起一次调用，失败立即报错。**通常用于非幂等的写操作** |
| Failsafe Cluster | 失败安全 | 出现异常时直接忽略，返回一个空结果 |
| Failback Cluster | 失败自动恢复 | 后台记录失败请求，定时重发。**常用语消息通知** |
| Forking Cluster | 并行调用多服务器 | 并行调用多个服务器，只要一个成功即返回。**常用于实时性要求较高的读操作，浪费服务资源** |
| Broadcast Cluster | 广播调用 | 广播调用所有提供者，逐个调用，任意一台报错则报错。**用于通知所有提供者更新缓存或日志等本地资源信息** |

## Dubbo序列化协议

Dubbo 默认使用的序列化方式是 hession2

## Dubbo线程模型

![](./../images/dubbo/线程模型.jpg)

在Dubbo的线程模型中，如果事件处理较慢，需要派发到线程池，与JDK中的线程池类似：

派发策略（Dispatcher）
- `all`：所有消息都派发到线程池，包括请求、响应等
- `direct`：所有消息都不派发到线程池，直接在IO线程上执行
- `message`：只有请求响应消息派发到线程池，其他连接断开事件、心跳等消息，直接在IO线程执行
- `execution`：只有请求消息派发到线程池，不含响应，响应和其他连接断开事件、心跳等消息，直接在IO线程上运行
- `connectino`：在IO线程上，将链接断开事件放入队列，有序逐个运行，其他消息放入线程池

线程池（ThreadPool）
- `fixed`：固定大小的线程池，饿汉式
- `cached`：缓存线程池，懒汉式，空闲一分钟则删除
- `limited`：可伸缩线程池，但池中的线程数只会增长，不回收缩（避免收缩时突然来了大流量的性能问题）
- `eager`：优先创建`worker`线程池。
  - 在任务数大于`corePoolSize`但小于`maximumPoolSize`时，优先创建`Worker`来处理任务。
  - 当任务数量大于`maximumPoolSize`时，将任务放入阻塞队列中，阻塞队列满则抛出`RejectedExecutionException`
  - 相较于`cached`，`cached`在任务数量超过`maximumPoolSize`时直接抛出异常


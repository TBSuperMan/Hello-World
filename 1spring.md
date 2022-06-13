# Spring

## 区分 BeanFactory 和 ApplicationContext？

| BeanFactory                | ApplicationContext       |
| -------------------------- | ------------------------ |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化               | 支持国际化               |
| 不支持基于依赖的注解       | 支持基于依赖的注解       |

BeanFactory和ApplicationContext的优缺点分析：

BeanFactory的优缺点：

- 优点：应用启动的时候占用资源很少，对资源要求较高的应用，比较有优势；
- 缺点：运行速度会相对来说慢一些。而且有可能会出现空指针异常的错误，而且通过Bean工厂创建的Bean生命周期会简单一些。

ApplicationContext的优缺点：

- 优点：所有的Bean在启动的时候都进行了加载，系统运行的速度快；在系统启动的时候，可以发现系统中的配置问题。
- 缺点：把费时的操作放到系统启动中完成，所有的对象都可以预加载，缺点就是内存占用较大。

> ApplicationContext实际上是BeanFactor的子类：ListableBeanFactor的子类

## 介绍一下IOC

IOC：控制反转，由**spring来负责控制对象的生命周期和对象间的关系**

IOC是Spring针对解决**程序耦合**而存在的。

所有的类都会在spring容器中注册，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。
所有的类的创建、销毁都由 spring来控制，也就是说**控制对象生存周期的不再是引用它的对象，而是spring**。以前创建对象的主动权和创建时机是由自己把控的，而现在这种权力转移到spring。

## 介绍一下DI

DI：依赖注入，组件之间依赖关系由容器在运行期决定，形象的说，即**由容器动态的将某个依赖关系注入到组件之中，也就是==动态的向某个对象提供它所需要的其他对象==**。

依赖注入的目的是**提升组件重用的频率**。

通过依赖注入机制，我们只需要通过简单的配置描述如何创建对象，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

而DI可以说是IOC的另一种解释

通常，依赖注入可以通过三种方式完成，即：

- 构造函数注入
- setter 注入
- 接口注入

在 Spring Framework 中，仅使用构造函数和 setter 注入。


## Spring 中的 bean 的作用域有哪些?

- `singleton` : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- `prototype` : 每次请求都会创建一个新的 bean 实例。
- `request` : 每一次HTTP请求都会产生一个新的bean，**该bean仅在当前HTTP request内有效**。
- `session` ：在一个HTTP Session中，一个Bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
- `global-session`： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

## 将一个类声明为Spring的 bean 的注解有哪些?

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 @Autowired 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 Spring 组件。如果一个Bean不知道属于哪个层，可以使用@Component 注解标注。 
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。
- 在返回Object的方法上使用`@Bean`注解

使用注解的目的是让它们被Spring扫描到，从而让Spring利用它们加载`BeanDefinition`，并用于后续的Bean创建

## Spring 中的 bean 生命周期?

Bean的生命周期是由容器来管理的。主要在创建和销毁两个时期。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318172844873.png)

![](http://blog-img.coolsen.cn/img/1583675090641_51.png)

## Bean的创建过程

> bean初始化时先进行BeanNameAware、BeanClassLoaderAware和BeanFactoryAware感知注入
> 再执行前置处理
> Bean的初始化 -> 执行InitializingBean、initMethod、@PostConstruct对应的方法
> 最后执行后置处理   

**BeanDefinition的创建阶段**
1. 加载XML配置文件并解析，将解析结果封装为BDF;
2. 解析`@Component`、`@PropertySource`、`@ComponentScan`等注解，并进行**包扫描**，解析配置类中的注解，将解析好的信息封装为BDF
3. 进行BeanDefinition的编程式构建
4. 进行BDF的后处理

**Bean的实例化阶段**
1. 按照Ordered接口的顺序，创建BeanPostProcessor
2. 注册一个BeanPostProcessorChecker，检查BeanPostProcessor实例化期间，是否有Bean意外创建
    （检查BeanFactory中的后置处理器的数量是否少于预期）
3. 将之前得到的BeanPostProcessor注册到BeanFactory中
4. 通过beanFactory，初始化非延迟加载的单例Bean。流程如下：
   1. 如果是单例Bean，则尝试从单例缓存中直接获得对象 
   3. 根据BeanName获取BeanDefinition，将其与父BDF合并，作为最终的BeanDefinition
   4. 配置Bean的scope，默认为singleton
   5. 调用后置处理器拦截Bean的创建，给后置处理器一个机会来创建代理Bean而不是目标Bean实例
   6. 真正创建Bean对象。包含：
      1. Bean对象的实例化（此时，Bean中所有属性为空）
         1. 首先进行Bean的类型的解析，然后进行校验，若Bean无法访问，则抛出异常
         2. 进行Bean的创建：根据BDF的信息，反射调用工厂方法创建Bean对象，或直接通过构造器创建Bean对象（如果是Prototype，则会将创建Bean需要的信息保存起来，以后直接取，不用再解析BDF）
         3. 得到Bean对象的引用
      2. 属性赋值与依赖注入
         1. 回调InstantiationAwareBeanPostProcessor的`postProcessAfterInstantiation`方法，在属性赋值之前，对Bean的属性进行干预（其方法返回false，则不再执行后面的属性赋值+依赖注入）
         2. 对@AutoWired进行解析，从BDF中找到@Autowired的Bean，先对它们进行创建
         3. 进行PropertyValues的封装
         4. 在已封装好的PropertyValues的基础上，进行属性赋值与依赖注入
      3. Bean对象的初始化（该步骤后，Bean已经完整）
         1. 执行Aware回调
         2. 依次执行容器中所有`BeanPostProcessor`的前置回调方法
         3. 执行生命周期回调`invokeInitMethods`，执行xml中的`init`方法，进行Bean的初始化
         4. 执行`BeanPostProcessor`的后置回调
   7. 注册销毁时的回调钩子

完成非抽象、非延迟加载的单例Bean的初始化后，执行`finishRefresh()`方法，清空之前使用的资源缓存，并将事件进行广播

## Bean的销毁过程：

当Bean不再用到，便要销毁 
1，若实现了`DisposableBean`接口，则会调用destroy方法； 
2，若配置了`destry-method`属性，则会调用其配置的销毁方法；

关闭ApplicationContext有以下几步：

1. 广播容器关闭事件
2. 通知所有实现了Licecycle接口的Bean回调`close`方法，多用于关闭连接、释放资源
3. 销毁所有Bean
4. 关闭BeanFactory
5. 标记本身为不可用

## IOC容器是怎么初始化的

IOC容器的初始化流程同样体现在`refresh()`方法中：
1. 容器预先准备，记录容器启动的时间和标记
2. 创建`BeanFactory`，如果有则销毁，没有则创建
3. 装载`BeanDefinition`
4. 配置`BeanFactory`的标准上下文特性，例如`ClassLoader`、`PostProcessor`等
7. 注册用于拦截Bean创建的`BeanPostProcessor`
8. 初始化MessageResource、上下文事件广播，注册监听器
9. 完成对非延迟初始化的单例对象的初始化
10. 完成容器的初始化
11. finishRefresh，

## IOC的原理

IOC的核心技术是==反射==，反射的主要用途即是根据给出的类名（字符串方式）来动态地生成对象。

由于IOC/DI是**在运行时动态地将依赖注入到类**中，自然要使用**可以动态获取类的信息与动态调用对象方法的反射机制**
spring是在将标有相关注解的类**实例化之后**，通过反射读取其成员变量，并**通过反射为其成员变量赋值**，从而实现依赖注入

而在完成Bean的初始化之后，spring会为其注册用于销毁的方法

Spring是通过BeanFactory创建Bean的，并被保存在上下文的map中，也就相当于被引用，而当spring的容器关闭时，会调用Bean的用于销毁的方法，让Bean被送去垃圾回收

## 怎样才能知道IOC容器已经完全初始化了

在`refresh()`方法中，
在完成Bean的初始化之前，会初始化事件广播器`ApplicationEventMulticaster`，并根据硬编码、配置文件、依赖注入注册监听器`Listener`
而在最后的`finishRefresh`方法中，也会**将上下文刷新完毕事件推送给事件广播器**
我们只需要监听`ContextRefreshedEvent`，就可以知道IOC容器已经完成初始化了。

> 在广播事件时，会遍历所有监听器，一一判断和当前事件是否匹配

## 为什么需要知道IOC容器的初始化进度


## ApplicationContext初始化中的扩展点有哪些
1. BeanFactory创建完成后，可以增加或修改BDF
   - `BeanFactoryProcessor` 
2. Bean实例化前后
   - `InstantiationAwareBeanPostProcessor` 
3. Bean初始化前后
   - `BeanPostProcessor`

## Spring后置处理器的接口名叫什么

最上层的接口为：`BeanPostProcessor`
BeanPostProcessor在bean实例上运行，它作用于bean对象创建后，如果在一个容器中定义了Processor，它将只作用于当前容器的bean

https://blog.csdn.net/qq_25215821/article/details/106480780


## 自己也可以实现Bean的前置后置处理器，为什么要用Spring提供的

1. Spring提供的前置后置处理器是按顺序执行的，它们包含@Order注解，用于标识执行后置处理器的顺序，如果自己实现的话，可能导致乱序，出现奇怪的问题
2. 自己实现Bean的后置处理器，会导致耦合严重，不符合Spring的初衷，而且要实现对Bean的初始化的监听，比较麻烦


## 实际开发中，在哪里使用了Bean的前置后置处理器

可以基于BeanPostProcessor进行一些统计信息的收集，或收集日志之类的，具体和AOP差不多，一般都是用AOP代替

## Spring为啥要使用BeanPostProcessorChecker

BeanPostProcessorChecker用于检查是否存在不会被所有后置处理器处理的bean，如果存在，会输出一条INFO级别的提示

其本身也是一个PostProcessor，负责检查BeanPostProcessor实例化期间，**是否有Bean意外创建**，如果Bean提前创建了，它就不能被所有PostProcessor处理时（因为还有BeanPostProcessor没创建）

> 如果有BeanPostProcessor依赖于Bean，则在实例化BeanPostProcessor前，会首先实例化Bean，而实例化Bean时，又会执行后置处理器，但是此时的后置处理器并没有全部注册完成，所以这个依赖bean不会被还没实例化的后置处理器处理到

## Spring为啥要使用ApplicationListenerDetector

ApplicationListenerDetector主要用于管理监听器：
1. bean实例化时，如果这个bean是监听器且是单例模式的，则在广播中注册这个监听器
2. bean销毁时，判断如果是监听器，则在广播中移除这个监听器

## 事件订阅的接口名字叫什么

`ApplicationEventPublisher`：事件发布器，接受事件，交给事件广播器处理
`ApplicationEventMulticaster`：事件广播器，拿到事件发布器发布的事件，广播给**监听器**
`ApplicationListener`：事件监听器，监听事件，并设定相应的业务


## Spring中出现同名bean怎么办？

- 同一个配置文件内同名的Bean，**最上面定义**的优先
- 不同配置文件中存在同名Bean，**后解析的配置文件**优先
- 同文件中`@ComponentScan`和`@Bean`出现同名Bean。
  同文件下==1@Bean优先==
  通过`@ComponentScan`扫描进来的优先级是最低的，原因就是**它扫描进来的Bean定义是最先被注册的** 

Spring**在解析并注册BDF**时，会判断是否存在Bean覆盖的情况：
1. 若**同一个配置类**中的`@Bean`名字相同，**以先加载的@Bean方法为准**
2. 若**不同的配置类**中的`@Bean`名字相同，则先加载的`@Bean`可以被覆盖，**以后加载的@Bean方法为准**

**当然，如果配置中要求不允许Bean覆盖，则会直接抛出异常**

## Spring 怎么解决循环依赖问题？

spring对循环依赖的处理有三种情况： 
- **构造器的循环依赖**：处理不了的，直接抛出BeanCurrentlylnCreationException异常。
  - 通过构造函数注入，无法处理。因为构造器的调用是在Bean的实例化之前，一二三级缓存中都没有Bean的信息
- **单例模式下的setter循环依赖**：通过**三级缓存**处理循环依赖。 
  - set方法注入表示的是**弱的依赖关系：聚合关系**，即依赖的对象只是当前对象的一个组件，不影响当前对象整体
- **protoType类的循环依赖**：无法处理。
  - 因为**spring不会缓存‘prototype’作用域的bean**，而spring中循环依赖的解决正是通过缓存来实现的。


Spring的单例对象的初始化主要分为三步：
1. createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象

2. populateBean：填充属性，这一步主要是多bean的依赖属性进行填充

3. initializeBean：调用spring xml中的init 方法。

Spring中，针对单例对象，主要包含以下三类缓存：
1. 一级缓存：singletonObjects：**完全初始化好的单例Bean的缓存**
   - 普通的IOC容器 
2. 二级缓存：earlySingletonObjects：**完成了实例化，但未完成属性赋值与依赖注入的单例Bean** 
3. 三级缓存：singletonFactories：**单例Bean的工厂对象**
   - 在`doCreateBeanBean()`方法中，**Bean实例化之后，属性赋值之前**，通过`addSingletonFactory()`方法添加到缓存中
   - 该工厂通过匿名内部类，创建存放于二级缓存中的对象 

> 在二级缓存中：
> - 如果Bean被AOP切面代理，则保存的是Bean的代理对象
> - Bean没有被代理，则保存原始Bean对象
> 
> 三级缓存同理，如果被代理，则工厂返回Bean的代理对象

![](http://blog-img.coolsen.cn/img/1584758309616_10.png)

> 如果是单例类，在执行`doGetBean()`方法，进行类的实例化、属性赋值与依赖注入、初始化时，**会首先从ApplicationContext的`getSingleton`中尝试获取单例对象**，并直接利用该单例类作为返回结果

假设循环依赖情况为：
- A的某个field或者setter依赖了B的实例对象，
- B的某个field或者setter依赖了A的实例对象

则在spring初始化Bean时，流程如下：
1. A的实例化
    A首先完成实例化，并且将自己提前曝光到`singletonFactories`中；
2. A的依赖注入
    A发现自己依赖对象B，此时就尝试去`get(B)`，发现B还没有被create，所以走B的create流程：
3. B的实例化与依赖注入
    B发现自己依赖了对象A，于是尝试`get(A)`
4. 首先尝试一级缓存`singletonObjects`(肯定没有，因为A还没初始化完全)
5. 尝试二级缓存`earlySingletonObjects`（也没有）
6. 尝试三级缓存singletonFactories，由于A通过`ObjectFactory`将自己提前曝光了，所以B能够通过`ObjectFactory.getObject`拿到未完全初始化的A对象   
> B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存`singletonObjects`中。
> 此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中
> 而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。



## 为什么要用三级缓存解决循环依赖，而不是二级缓存？

对于普通的Bean，实际上两级缓存就可以解决循环依赖的问题
但当Bean被代理时，两级缓存无法解决问题：

假设AService的testAopProxy被AOP代理了，此时`singletonFactory.getObject()`方法返回的是一个AService的代理对象。
然而，当我们再次调用`singletonFactory.getObject()`方法，**返回的是另一个新的代理对象**，这就违反了单例的规定。

需要借助二级缓存来解决这个问题：
在此处调用`singletonFactory.getObject()`，实际上会调用`getEarlyBeanReference()`方法，它会将`singletonFactory`从三级缓存中移除，并将生成的Bean放入二级缓存。
这样一来，之后就会直接从**二级缓存**中获取Bean对象，而不是继续通过**三级缓存**生成对象，维护了单例性


## XML的两种解析策略有什么区别

1. DOM解析：
   - 需要xml完全加载进内存才可以操作，可以方便进行增删改查操作；
   - 耗费内存 
   - 是平台无关的官方解析方式
2. SAX解析
   - 逐渐扫描xml文件，当遇到标签时触发解析处理器，不需要加载进内存；
   - 只能读取，无法进行增删改查，且难以同时访问同一个XML的多处数据
   - 是基于事件驱动的解析方式
3. JDOM方法：基于SAX解析发展得到，利用了大量Collection的类
4. DOM4J：是JDOM的一种智能分支，合并了许多基本XML文档表示的功能
   - DOM4J使用接口和抽象基本类方法，是一个优秀的JavaXMLAPI，具有性能优异、灵活性好、功能强大和极端易使用的特点 


## BeanPostProcessor和InstantiationAwareBeanPostProcessor的区别

BeanPostProcessor是**后置处理器的基础接口**，而InstantiationAwareBeanPostProcessor是BeanPostProcessor的一个子接口

- BeanPostProcessor提供的方法为`postProcessBefore/AfterInitialization`，其执行的时机为**Bean初始化的前后**
- InstantiationAwareBeanPostProcessor提供的方法为`postProcessBefore/AfterInstantiation`，其执行的时机为**Bean实例化前后**，除此之外，还提供了对Properties、PropertieValues的后置处理方法

## 什么是 AOP？

AOP(Aspect-Oriented Programming), 即 **面向切面编程**, 它与 OOP相辅相成, 提供了与 OOP 不同的抽象软件结构的视角。
在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 **Aspect(切面)**

## AOP 有哪些实现方式？

实现 AOP 的技术，主要分为两大类：

- AspectJ的静态代理：指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强；
  - **编译时编织（特殊编译器实现）**
  - **类加载时编织（特殊的类加载器实现）**

  - 静态代理 ：**需要代理对象和目标对象实现一样的接口**，调用代理的方法，而代理的方法中调用了目标对象的方法
    - 优点：对客户隐藏了被代理类的具体实现逻辑在一定程度上实现了解耦合，同时提高了安全性
    - 缺点：
      1. 静态代理类**需要实现目标类（被代理类）的接口**，并实现其方法，造成了代码的大量冗余
      2. 静态代理只能对某个固定接口的实现类进行代理服务，其灵活性不强
- 动态代理：在运行时在内存中动态生成 AOP 动态代理类，因此也被称为运行时增强。
  - `JDK` 动态代理：通过==反射==来接收被代理的类，并且**要求被代理的类必须实现一个接口** 。
    - JDK 动态代理的核心是 `InvocationHandler` 接口和 `Proxy` 类，因为需要最终生成的代理类会**继承Proxy类**，所以只能通过实现接口的方式实现动态代理
  - `CGLIB`动态代理： 如果目标类没有实现接口，那么 `Spring AOP` 会选择使用 `CGLIB` 来动态代理目标类
     - `CGLIB`通过==动态修改字节码的方式，动态生成代理类的子类实现代理功能==，**生成的子类重写了被代理类的所有方法**（不包括final方法、private方法），cglib生成的子类中通过**方法拦截**的方式实现对代理类方法的代码织入
     > 注意， `CGLIB` 是通过==继承==的方式做的动态代理，因此如果某个类或方法被标记为 `final` ，那么它是无法使用 `CGLIB` 做动态代理的。
  - 优点：
    - 只需要将被代理对象作为参数传入代理类就可以获取代理类对象，从而实现类代理，具有较强的灵活性
    - 动态代理的服务内容不需要像静态代理一样写在每个代码块中，只需要写在invoke()方法中即可，降低了代码的冗余度
  - 缺点：
    - JDK动态代理仍需实现接口
    - 通过接口代理对象，会导致执行效率降低


## AOP是怎样实现动态代理的

上面已经说了，SpringAOP是怎么实现代理对象的生成的，但并未解释AOP是怎样实现切面织入的

AOP的核心原理就是：**在目标业务bean创建+初始化过程中spring利用动态代理机制对原始的业务bean进行增强**

1. 开启AOP
   想要开启AOP功能，就必须在配置类中加上`@EnableAspectJAutoProxy`注解，而该注解内部使用了`@Import(AspectJAutoProxyRegistrar.class)`，将`AspectJAutoProxyRegistrar`作为Bean注册入IOC容器。
2. AOP核心类的BDF的注册
   在该类中，主要任务即`AnnotationAwareAspectJAutoProxyCreator`的`RootBeanDefinition`的注册，从而令Spring在启动时将对象注入IOC容器
3. AOP核心后置处理器的创建
   AOP的核心逻辑位于`AnnotationAwareAspectJAutoProxyCreator`，而这个类继承了`BeanPostProcessor`接口与`BeanFactoryAware`接口，因此，AOP的核心后置处理器`AnnotationAwareAspectJAutoProxyCreator`会在创建BeanPostProcessor时一起创建。
4. 切入点表达式的解析
   同时，在初始化AOP核心后置处理器时，会到BeanFactory中解析容器中的所有切面类的切入点表达式，从而**在实例化Bean之前，告知BeanPostProcessor哪些Bean需要切入**
5. AOP代理对象的创建与初始化
   - AOP是通过BeanPostProcessor实现的，AOP目标对象自然也是在**BeanPostProcessor在初始化之后的方法**中初始化的，在该方法中，调用动态代理对Bean增强的核心方法为`warpIfNecessary`，其主要思路为：
     1. **获得当前Bean对应的通知(拦截器Advisor)** 
     2. 获取拦截器成功后，**创建Bean代理对象，对Bean进行增强**，并在其中获取适用于当前Bean的通知方法，并对其进行排序
        - 通知的先后顺序为：AfterThrowing -> AfterReturning -> After -> Around -> Before
     3. 通过代理对象，对当前Bean的方法进行增强：
        - 如果目标对象**实现了接口或者说指定了使用JDK的动态代理**，调用JDK动态代理进行目标对象的增强
        - **否则调用CGLIB进行目标对象的增强----一般都会走这个方法进行对象的增强**


**当目标Bean被AOP通过动态代理增强后，注入到IOC容器中的就不再是目标Bean本身的对象了，而是Bean的代理对象**
当然，之后再从IOC容器里取目标Bean，取到的也将是的代理对象，而不是目标Bean本身

当我们调用目标Bean的方法时，使用的也是代理对象的方法。
注意：AOP会将织入的通知按顺序转换为**方法拦截器**，并组成**方法拦截器链**，在真正执行方法时，通过**责任链模式+递归**的方法，进行逐个调用。因此，最先执行的是责任链中最后的`Before`通知

## Spring AOP and AspectJ AOP 有什么区别？

Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。
Spring AOP 仅支持方法级别的 PointCut；AspectJ 提供了完全的 AOP 支持，它还支持属性级别的 PointCut。

## CGLib可不可以增强接口

不能增强接口，基于JDK的动态代理必须增强接口，而CGLib没有这个限制，因为CGLIB是通过==继承==的方式实现的动态代理，只要类没有被标注为`final`，都可以实现增强，但接口是不存在子类这一概念的，只存在子接口或实现类的概念

## CGLib是怎样操作字节码的

CGLIB包的底层是通过使用一个小而快的 字节码处理框架ASM，来转换字节码并生成新的类。

## ASM操作字节码的原理

ASM是一个java字节码操纵框架，它能被用来动态生成类或者增强既有类的功能。ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。
ASM从`.class`文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。

其核心类包含：
1. ClassReader：字节码的读取与分析引擎，用于解析编译过的class字节码文件。
    它采用类似SAX的事件读取机制，每当有事件发生时，调用注册的ClassVisitor、AnnotationVisitor、FieldVisitor、MethodVisitor做相应的处理。
   - 在调用ClassReader的accept方法时，它解析字节码中常量池之后的所有元素。
2. ClassVisitor：定义在读取Class字节码时会触发的事件，如类头解析完成、注解解析、字段解析、方法解析等；
   除此之外，还有AnnotationVisitor、FieldVisitor、MethodVisitor，分别对应解析注解时触发的事件、解析域时触发的事件、解析方法时触发的事件 

3. ClassWriter：是ClassVisitor的子类，用于拼接字节码，比如说修改类名、属性以及方法，甚至可以生成新的类的字节码文件。
   其他Writer也是对应Visitor的子类 

4. ClassAdapter:该类也实现了ClassVisitor接口，它将对它的方法调用委托给另一个ClassVisitor对象。


## Spring 中的单例 bean 的线程安全问题？

当多个用户同时请求一个服务时，**容器会给每一个请求分配一个线程**，这时多个线程会并发执行该请求对应的业务逻辑（成员方法）。
此时就要注意了，如果该处理逻辑中有**对单例状态的修改**（体现为该单例的成员属性），则必须考虑线程同步问题。
 
**线程安全问题都是由全局变量及静态变量引起的。** 
若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则就可能影响线程安全.

**无状态bean和有状态bean**

- 有状态就是有数据存储功能。
  - 有状态对象(Stateful Bean)，就是**有实例变量的对象**，可以保存数据，是非线程安全的。在不同方法调用间不保留任何状态。
- 无状态就是一次操作，不能保存数据。
  - 无状态对象(Stateless Bean)，就是**没有实例变量的对象**.不能保存数据，是不变类，是线程安全的。

- 无状态的Bean适合用单例模式，这样可以共享实例提高性能。
- 有状态的Bean在多线程环境下不安全，适合用`Prototype`原型模式。 

Spring使用`ThreadLocal`解决线程安全问题（例如JDBCTemplate内部使用ThreadLocal）
如果你的Bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。

## Spring 框架中用到了哪些设计模式？

**工厂模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。

**代理模式** : Spring AOP 功能的实现

**单例模式** : Spring 中的 Bean 默认都是单例的。

**模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。

**包装器模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

**观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。

**适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

## Spring 事务实现方式有哪些？

- 编程式事务管理：可以通过编程的方式管理事务，这种方式带来了很大的灵活性，但很难维护。是侵入性事务管理
- 声明式事务管理：可以将事务管理和业务代码分离。你只需要通过注解或者XML配置管理事务。
  - 建立在**AOP**之上，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，执行完目标方法之后根据执行的情况提交或者回滚。
  - 不足：粒度是方法级别，而编程式事务管理是可以到代码块的

## spring事务定义的传播规则

事务的传播性一般用在**事务嵌套**的场景，比如一个事务方法里面调用了另外一个事务方法，那么两个方法是各自作为独立的方法提交，还是内层的事务合并到外层的事务一起提交，这就是需要事务传播机制的配置来确定怎么样执行。

> 传播规则回答了这样一个问题：**一个新的事务应该被启动还是被挂起，或者是一个方法是否应该在事务性上下文中运行**

常见的事务传播规则如下：
- REQUIRED: 
  - 默认的传播规则，如果外层有事务，则**当前事务加入到外层事务，一块提交，一块回滚**。
  - 如果外层没有事务，新建一个事务执行。
- REQUIRES_NEW: 
  - 每次都会新开启一个事务，同时**把外层事务挂起**，当当前事务执行完毕，恢复上层事务的执行。
  - 如果外层没有事务，新建一个事务执行。
- SUPPORTS: （**完全依赖外层的事务**）
  - 如果外层有事务，则加入外层事务。
  - 如果外层没有事务，则直接使用**非事务方式**执行。
- NOT_SUPPORTED: （**完全不支持事务**）
  - **以非事务方式执行操作**
  - 外层有事务，则挂起外层事务，执行完当前代码才恢复，无论是否异常都不会回滚当前的代码
- NEVER: 
  - 不支持外层事务，即**如果外层有事务就抛出异常**
- MANDATORY: 
  - 与NEVER相反，**如果外层没有事务，就抛出异常**
- NESTED:
  - 外层存在事务，则开启一个**嵌套事务**，它是外层事务的子事务。嵌套事务开始执行时，保存**状态保存点**，如果这个嵌套事务失败，则回滚到此**状态保存点**
    - 嵌套事务是外部事物的一部分，只有外部事务结束后，它才会被提交
  - 外层不存在事务，则新建一个事务执行
  > 也就是说，如果事务B为事务A的嵌套事务，当事务B回滚，只会回滚到状态保存点，不会导致事务A一起回滚
  > 注意：上述情况的要求是B捕获了异常且不往上抛出，否则事务A和嵌套事务B一起回滚



## Spring怎么实现事务？

Spring事务的使用方式为将`@Transactional`注解在方法上，表明该方法需要事务的支持，而Spring事务是基于AOP实现的。

声明式事务是利用AOP中的环绕增强实现的，其一个通过`final`方法判断当前是否有活跃的事务对象，并通过`ThreadLocal`对象在当前线程记录了当前事务的事务资源。

## Spring事务的隔离级别

和mysql相同

## @Transactional注解事务的特性

1. 在service类标签(一般不建议在接口上)上添加@Transactional，可以将整个类纳入spring事务管理，在每个业务方法执行时都会开启一个事务，不过这些事务采用相同的管理方式。
2. @Transactional 注解只能应用到 `public` 可见度的方法上。 如果应用在protected、private或者 package可见度的方法上，也不会报错，不过事务设置不会起作用。
3. 默认情况下，Spring会对`unchecked`异常进行事务回滚；如果是`checked`异常则不回滚。
   - 空指针等异常，会被回滚，文件读写，网络出问题，spring就没法回滚了 

## 事务注解在什么情况会失效


常见情况：
1. 调用的方法不是`public`的
   - 因为Spring事务是通过AOP实现的，而**CGLib不能重写private、protected与final的方法**，JDK动态代理也不能代理private的方法
2. 异常类型是`checked`异常，不会执行事务回滚
   - 如果想让checked异常也回滚，需要在注解上面写明异常的`class`即可
3. 数据库引擎要支持事务，如果是Mysql，注意表要使用支持事务的引擎，比如innodb，如果是myisam，事务是不起作用的
4. 是否开启了对注解的解析
5. spring是否扫描到你使用注解事务的这个类所在的包
6. 异常是不是被catch住了

其他情况：
1. 原始的SSM开发中，父子容器一起包扫描，会导致子容器先扫描到service并注册到子容器中，但不加载事务。
   之后父容器也会扫描到service，但因为子容器汇总的controller已经注入了没有事务代理的service，会导致事务失效
    - 应当在`spring-mvc.xml`中，设置只扫描`@Controller`注解，从而保证子容器仍然能从父容器中获取到service、dao等，而父容器开启事务控制，使得service是被事务增强过的
    - 如果不慎在`spring-mvc.xml`中忘记配置，扫描`Controller`的同时一并扫描了`service`，会导致**父子容器中同时存在service，此时controller会直接拿子容器中没有被事务增强过的service**

2. **声明式事务的配置必须由父 IOC 容器加载，SpringWebMvc 的子 IOC 容器加载不生效**
    与1.类似，声明式事务控制放在子容器，父容器感知不到子容器的配置

3.  如果 `@Transactional` 注解标注在接口上，但实现类使用 `Cglib` 代理，则事务会失效

4.  同一个类中，一个方法调用了自身另一个带有事务控制的方法，则直接调用时也会导致事务失效
    此时不能使用this调用（因为**实际起作用的是代理对象，而this指的是增强前的对象本身**）
    应当使用`AopContext`进行调用（需要在`@EnableAspectJAutoProxy`上设置`exposeProxy=true`，即暴露代理对象）

# springMVC

## SpringMVC 工作原理了解吗?

**原理如下图所示：**

MVC架构：
![](./../images/MVC.png)


JavaEE三层架构：
![](./../images/JavaEE三层架构.png)


![](./../images/springMVC.png)

**流程说明（重要）：**

1. 发送HTTP请求。
   客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. 寻找处理器
   `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 调用处理器
   解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. 进行业务处理
   `HandlerAdapter` 会根据 `Handler`来调用真正的处理器`Controller`开处理请求，并处理相应的业务逻辑。
5. 返回处理结果
   处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. 处理视图映射并返回模型
   `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
   `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7. HTTP响应
   把 `View` 返回给请求者（浏览器）


### DispatcherServlet

DispatcherServlet是SpringMVC的核心，统一接收客户端的所有请求，根据请求的uri，转发给匹配的Controller。配置于`web.xml`中，

主要任务：
1. 截获符合特定格式的URL请求
2. 初始化DispatcherServlet上下文对应的`WebApplicationContext`，并将其与业务层、持久化层的`WebApplicationContext`建立关联
3. 初始化Spring MVC的各个组成部件，并装配到DispatcherServlet中

此处的匹配寻找、请求转发、返回视图、相应json都不是DispatcherServlet干的，而是委托给其他组件干的

拦截格式：
1. 拦截*.do、*.htm
   最传统的方式，最简单也最实用，不会导致静态文件(jpg\js\css)被拦截
2. 拦截`/`，例如`/user/add`
   可以实现REST风格，但会导致静态文件（jpg,js,css）被拦截后不能正常显示，需要额外处理
   > 通过配置`<mvc:resources mapping="/images/**" location="/images/" /> `访问静态文件
3. 拦截/*，不可行，会导致jsp被拦截   

### Handler

在Controller中，标注了`@RequestMapping`的方法，就相当于一个`Handler`，任务即处理客户端发来的请求，并响应

### HandlerMapping

非常重要
处理器映射器，**负责根据`uri`与`@RequestMapping`寻找合适的`Handler`，组合可能匹配的拦截器后，包装为一个`HandlerExecutionChain`的对象，交给`DispatcherServlet`**

```java
public interface HandlerMapping {
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```
通常使用`RequestMappingHandlerMapping`作为落地实现，是支持`@RequestMapping`注解的处理器映射器

### HandlerAdapter

处理器适配器，蕴含了适配器模式

在`DispatcherServlet`拿到`HandlerExecutionChain`后，将拦截器与Handler的执行交给`HandlerAdapter`。
得到`Handler`的执行结果后，将返回的`ModelAndView`返回给`DispatcherServlet`

由于Handler包含`@Controller`+`@RequestMapping`的方式与实现`Controller`接口的方式（可能还有别的），想要用一个接口同时兼顾这两种方式，就需要使用适配器模式，先将Object接收，然后针对不同的Handler便携方式，找合适的HandlerAdapter即可

通常使用`RequestMappingHandlerAdapter`作为实现类，是基于`@RequestMapping`的处理器适配器，底层使用反射调用Handler的方法

### ViewResolver

`DispatcherServlet`得到`ModelAndView`后，需要响应视图，这个任务交给`ViewResolver`。`ViewResolver`会根据`ModelAndView`中存放的视图名称，去预先配置好的位置找对应的视图文件，进行实际的视图渲染，渲染完成后将视图响应给`DispatcherServlet`

当Handler执行完毕后，SpringWebMVC会根据当前Handler及其返回值，决定使用何种方式进行响应，如果标注了`@RestController`或`@ResponseBody`，则`ViewResolver`不工作，而是选择`jackson`进行解析

## Controller的原理

1. Controller实际上也是IOC容器中的Bean，通过`@Controller`标识其为SpringMVC的控制器类
2. 通过`@RequestMapping`来根据URL映射request请求到控制器类或Controller的方法上。

## Controller是单例的吗？单例的好处是什么？与多线程有什么关系

**Controller默认为单例**的，单例的好处是**可以在多线程共享实例，不需要多次创建，降低了开销**；

但Controller能够在多线程下应用单例模式的条件是：**Controller中没有需要进行写入的共享状态**。
一旦有了共享状态，Controller就从无状态的类变为有状态的类，在多线程环境下会出现竞争，导致出现线程安全问题。

因此如果要使用有状态的Controller，就需要通过注解，将其设置为Prototype

## 常用注解

- `@RequestMapping`：用于：
  1. 请求路径的拼接(`value="/list"`)
  2. 请求方式的限定(`method=RequestMethod.GET`，如果此时用POST请求，会报405)
  3. 注解扩展(拥有以下别名：GetMapping、PostMapping、PutMapping、DeleteMapping)

- `@RequestParam`：当**类的属性名与请求的参数名对不上**时，通过`@RequestParam`，可以通过标注在指定的属性上，使类的属性名与请求参数名对应，相当于给类的属性名一个别名
    -required：表示请求参数中是否必须携带对应的参数
    -defaultValue：指定参数不传递时的默认值


- `@RequestBody`：将请求的json数据转换为对象（可用于请求参数，例如：`@RequestBody Department department`）
  - 该注解用于读取`Request`请求的`body`部分数据，使用系统默认配置的`HttpMessageConverter`进行解析，然后**把相应的数据绑定到要返回的对象上**；
   再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。

- `@ResponseBody`：将返回的java对象转换为json格式的数据，写入response对象的body区（**使用此注解之后不会再走视图处理器，而是直接将数据写入到流中**）
  - 不加`@ResponseBody`，方法返回值通常为跳转路径，将其作为视图名称，并自动匹配视图去显示，
  - 加上`@ResponseBody`，就仅仅是将方法返回值直接写入HTTP响应正文(ResponseBody)，当作内容直接返回到客户端，并且会自适应响应头的`content-type`。
    - 返回的字符串符合`json`，那么`content-type`就是`application/json`，如果是普通字符串，就是`text/plain`，但是加上注解属性`produces=application/json`，那么不管内容是什么格式，响应头的`content-type`就一直是`application/json`，不再去做自适应，至于内容是不是json都不重要了
  - 当返回的数据不是html标签的页面，而是其他各种格式的数据时使用
  - 且一般在异步获取数据时使用(AJAX)

- `@RestController`：如果需要在一个Controller的==所有方法==上都注明`@ResponseBody`，即该Controller中没有任何视图跳转，那么可以直接用`@RestController`代替

- `@ModelAttribute`：用于**在参数中标注快速存入需要添加到request域中的数据，可以用于数据的回显**

- `@ControllerAdvice`：与下面这个注解一同使用，表示**当前类用于增强其他Controller**，它不是@Controller，而是一个普通的@Component；可以用于创建全局异常处理器
    可以用value/basePackages来指定要增强的包，也可以用assignableTypes来指定要增强的@Controller
    - `@ExceptionHandler`：声明式的捕获指定的异常，参数为需要捕获的异常的class
    - `@InitBinder`：使用共用方法初始化控制器方法调用使用的数据绑定器
      - 被`@InitBinder`标注的方法，会**在每一次`Controller方法执行时都触发**，所以不太实用
    - `@ModelAttribute`：可用于公共数据暴露与请求数据处理。
      - Target为参数与方法，
        - 标注在方法时，可以用来暴露一些特定的数据（通过返回值暴露或传入Model/ModelMap/HttpServletRequest手动set）
      - **同样会在每次Controller方法执行前执行，所以在Controller的任意位置，都能得到暴露的数据**
      - 如果需要让暴露的属性在Controller方法中能够拿到，就需要在`Controller`的方法参数前标注`@ModelAttribute`

- `@SessionAttribute`：从session中取数据，与`@ModelAttribute`在功能上相似，`@ModelAttribute`在`request域`工作，而`@SessionAttribute`在`session域`工作
    - `@SessionAttributes`：向session中存数据，只能注解在类上，而`@SessionAttribute`注解在方法参数上

- `@CrossOrigin`：用于跨域资源共享，可以标注在类或方法上


> 同源策略：
> 在浏览器上对资源的一种保护策略，规定了三个相同：**协议相同、域名/主机相同、端口相同**，当这三个因素相同时，浏览器会认为访问的资源是一个来源
> 主要是为了保护`cookie`

## RESTful风格是啥

REST描述了一个架构样式的网络系统，指的是一组架构约束条件和原则
如果一个程序架构的设计符合REST原则，则称它为RESTful架构

REST即`表现层状态转化`，而其中的表现层指的是`资源的表现层`，我们把"资源"具体呈现出来的形式，叫做它的"表现层"。
而资源表现层的状态转化体现的就是客户端与服务器的一个交互过程，客户端与服务端的互动体现为其中的资源的状态转化，而这种转化是建立在表现层之上的，即建立在资源的载体上。
客户端用到的协议是HTTP协议，其中的四个操作方式的动词分别对应四种基本操作：
- GET：获取资源/数据
- POST：新建
- PUT：更新
- DELETE：删除

因此，RESTful架构即：
1. ==每一个URI代表一种资源==；
2. 客户端和服务器之间，传递这种资源的某种表现层；
3. 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

RESTful的设计误区：
1. URI包含动词：资源时一种实体，应该是名词
   例如对`/posts/show/1`，其中show为动词，如果要符合RESTful，就应该设计为`/post/1`，然后用GET方法表示show
   如果有某种动作是四种基本动作表示不了的，例如转账，则应该将动作视作一种资源或一种服务： 

```java
错误：
POST /accounts/1/transfer/500/to/2

正确：
POST /transaction HTTP/1.1
Host: 127.0.0.1
from=1&to=2&amount=500.00
```
2. 在URI中加入版本号
   资源的不同版本可以理解为同一个资源的不同表现形式，应当采用同一个URI
   版本号可以在HTTP请求头信息的Accept字段中进行区分
   > 应当将API的版本号放入URL，而不是URI


## 请求乱码的原因

- POST请求乱码：浏览器提交表单的时候，字符编码集与后端 Tomcat 不一致。
  通常情况下，浏览器并不会发送带 content-type 请求头的字符编码标识符，而是会把字符编码的决定留在读取 HTTP 请求的时候。
  而Web容器（Tomcat）默认的编码为`ISO-8859-1`

- GET请求乱码：Web容器出问题了，可能是server.xml配置错误，没有加上`useBodyEncodingForURI=true`
   如果还是有乱码，可能是发ajax请求时出错，将上面的改为`URIEncoding=UTF-8`

## 静态资源的解析配置

当`<servlet-mapping>`中的`<url-pattern>`标签配置为`/`时，会对静态资源进行拦截，但SpringWebMVC并没有解析静态资源的手段，所以直接丢出404。此时就需要配置静态资源的解析

XML配置文件：`<mvc:resources mapping="/js/**" location="/js/" />`
如果是放在`/WEB-INF`下，则对location进行修改：`location="/WEB-INF/js/"`

注解方式：可以直接重写MVC注解配置类的`addResourceHandlers`方法（因为它已经实现了`WebMvcConfigurer`接口）

## ServletAPI的获取和使用

request与response的获取：
1. 在Controller的方法参数上声明`HttpServletRequest`与`HttpServletResponse`
2. 在Controller类中，通过`@Autowired`注入request与response
3. 通过静态类`RequestContextHolder`获取，同时，可以通过自定义工具类的静态方法，对其进行一些封装

## 介绍一下拦截器interceptor

拦截器包含主要用于拦截用户请求并作相应的处理。例如通过拦截器可以进行权限验证、记录请求信息的日志、判断用户是否登录等。可以通过实现`HandlerInterceptor`接口或实现`WebRequestInterceptor`接口来定义拦截器

自定义拦截器主要有三个方法：
- `preHandle()`：在**控制器方法前**执行，返回值表示是否中断后续操作
  - 当返回true，表示继续向下执行
  - 当返回false，则中断后续所有操作，直接返回（包含拦截器链中的其他方法与Controller中的方法）
- `postHandle()`：在**控制器方法调用后，且解析视图前**执行。
  - 可以通过此方法对请求域中的模型和视图做出进一步的修改。
- `afterCompletion()`：在整个请求完成，即视图渲染结束后执行。
  - 可以通过此方法实现资源清理、日志记录的工作

![](./../images/spring/拦截器执行流程.png)

因此，可以利用HandlerInterceptor实现User的获取：
- 根据token（UUID）获取Session，并进而获取User，并将其存入`ThreadLocal`
- 并将UserId与URI拼接在一起，从而让后面的请求可以通过`@PathVariable`直接获取UserId


## filter、intercepter的区别，用到了什么设计模式（责任链）

拦截器interceptor与过滤器Filter的区别：
1. 拦截器是Spring框架的概念，而过滤器是Servlet的概念
2. 拦截器是在`DispatcherServlet`(SpringWebMVC的核心)**接收到请求之后才有机会工作**的，不能处理没有接收到的请求；但过滤器是在`web.xml`或者借助Servlet3.0规范来注册的，可以拦截几乎所有请求
3. 拦截器可以借助依赖注入获取所需要的Bean，而过滤器无法使用正常手段获取（拦截器是被Spring的IOC统一管理的，它就是一个个普通的Bean）

### 拦截器的拦截时机

拦截器的核心接口为`HandlerInterceptor`，其中三个方法如下：

```java
public interface HandlerInterceptor {
    //在执行Controller的方法之前触发，可用于编码、权限检验拦截等
	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true;//如果这里为false，则Controller与下面的两个拦截方法都不会执行
	}

    //在执行完Controller方法后，跳转页面/返回json数据之前触发
    //参数中有一个ModelAndView，说明该时机下还没有确定好数据的封装和视图的返回，此时可以进行修改
	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

    //在完全执行完Controller方法后触发，可用于异常处理、性能监控等
    //参数中少了ModelAndView，但是多了一个Exception
	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}
}
```

## 拦截器的实现原理

首先，查看DispatcherServlet的源码，这是SpringMVC的核心类，查看其`doDispatch()`方法

在其中，分别按顺序执行了自定义拦截器的`preHandle`方法、`Controller`中的`RequestMapping`映射处理方法、自定义拦截器的`postHandle`方法、自定义拦截器的`afterCompletion`方法

其中，`preHandle`和`postHandle`是通过正向与反向遍历`interceptorList`，并执行其中的方法实现的


## @Autowire的原理

自动注入取代了手动为Bean进行配置的过程，可以说是DI依赖注入的核心

@Autowire这个注解是基于一个BeanPostProcessor实现的，在对Bean进行属性赋值之后，会遍历Bean中的域，并收集标注有`@Autowire`的域，Spring会遍历IOC容器，根据对应的规则，找到待注入的对象，并通过**反射**注入，从而实现依赖注入

@Autowire注解和@Resource一样，同样也可以标注在**字段或属性的setter方法**上，但它默认==按类型装配==。
@Autowire用于标识需要依赖注入的地方，并进行自动的注入。
@Autowire的注入方式有两种：setter方法注入 与 构造函数注入

注入规则：
- 默认按照**类型**进行注入，如果IOC容器中存在两个及以上的相同类型的bean时，根据**bean的名称**进行注入，如果没有指定名称的bean，则会报错

**根据注解的位置不同，@Autowire也会有不同的作用**：
- 注解于变量上：扫描IOC容器，寻找符合要求的Bean，进行注入
- 注解于方法上：会首先拿到方法的参数列表，然后根据上面所说的注入规则去Spring IOC中找相应的bean，注入参数
- 注解于构造器上：
   在A的创建过程中，会到IOC中寻找需要注入的Bean，完成A的创建
```java
   @Autowired
   public A(B b, C c) {
      System.out.println("b=" + b + ", c=" + c);
      this.b = b;
      this.c = c;
```

## SpringMVC的启动过程

SpringMVC的启动是基于配置文件web.xml的：
1. 读取与解析`web.xml`文件，并读取其中指定的`applicationContext.xml`文件，作为全局变量
1. 根据配置文件中的`<listener>`，创建和初始化指定的事件监听器
   - 通常为`ContextLoaderListener`，它实现了`ServletContextListener`接口，这个接口里的函数会结合web容器的生命周期被调用
2. 创建`ServletContext`，`ServletContextListener`的`contextInitialized()`方法被调用，并进一步调用`initWebApplicationContext()`方法，创建`root WebApplicationContext`对象，即**根IoC容器**
     - 同时，对根IOC容器进行配置与刷新，根据获取到的`applicationContext.xml`，进行Bean的初始化
     > 其中一般包含DAO层、Service层等相关Bean 
2. Filter的初始化
   初始化用于编码用户请求和响应的过滤器
3. Servlet的初始化
   - 常用的Servlet即`DispatcherServlet`，调用Servlet的`init()`方法，开始了`DispatcherServlet`的初始化

## DispatcherServlet的初始化

https://www.cnblogs.com/tengyunhao/p/7518481.html

1. `HttpServletBean` 的 `init()` 方法
   - 即DispatcherServlet的父类的`init()`方法，其中调用了后续的`initServletBean()`方法
     - 主要作用是加载 `web.xml` 中 `DispatcherServlet` 的 `<init-param>` 配置，并调用子类的初始化 
2. `FrameworkServlet` 的 `initServletBean()`
   - 在 HttpServletBean 的 init() 方法中调用了 `initServletBean()` 这个方法，其中又调用了`onRefresh()`方法
     - 主要作用是**建立 DispatcherServlet的 子ApplicationContext容器，并加载 web.xml 配置文件中定义的 Bean 到该容器中，最后将该容器添加到 ServletContext 中**
3. `DispatcherServlet` 的 `onRefresh()` 方法
   - 建立好 DispatcherServlet的 子ApplicationContext容器后，通过 `onRefresh()` 方法回调，进入 `DispatcherServlet` 类中。主要任务是**进行SpringMVC的初始化**，而`onRefresh()`方法就是调用了`initStrategies()`
4. `initStrategies()`
   - 进行SpringMVC的各个组件的初始化，其中包含`HandlerMapping`、`HandlerAdapter`、`HandlerExceptionResolver`等核心组件
   - `initHandlerMappings()`
     -  从 SpringMVC 的容器及 Spring 的容器中查找所有的 HandlerMapping 实例，并把它们放入到 handlerMappings 这个 list 中
     -  并不是进行HandlerMapping 实例的创建，它在之前的`onRefresh()`方法中就完成创建了

## 父子IOC容器的访问特性

- 根IoC容器作为**全局共享的IoC容器，放入Web应用需要共享的Bean**
- 子IoC容器**根据需求的不同，放入不同的Bean**，这样能够做到隔离，保证系统的安全性。

## DispatcherServlet处理请求的原理

DispatcherServlet通过其父类FrameworkServlet的`doGet()`、`doPost()`等方法处理请求

这些方法又调用了`processRequest()`方法，其中又调用了`DispatcherServlet`的`doService()`方法，并进一步调用`doDispatch()`方法，进行分发与处理。

`doDispatch()` 方法的主要过程：
1. 通过 `HandlerMapping` 获取 `Handler`
2. 找到用于执行它的 `HandlerAdapter`
3. 遍历拦截器，执行它们的`preHandle()`方法
4. 执行 `Handler`，得到 `ModelAndView` ，ModelAndView 是连接“业务逻辑层”与“视图展示层”的桥梁，
5. 遍历拦截器，执行它们的 `postHandle()` 方法
6. 通过 `ModelAndView` 获得 View，再通过它的 Model 对 View 进行渲染。
7. 遍历拦截器，执行它们的 `afterHandle()` 方法

## ExceptionHandler介绍

从字面上看，就是 **异常处理器** 的意思，其实际作用也是：若在某个Controller类定义一个异常处理方法，并在方法上添加该注解，那么**当出现指定的异常时，会执行该处理异常的方法**。其可以使用springmvc提供的数据绑定，比如注入HttpServletRequest等，还可以接受一个当前抛出的Throwable对象。

通常与`@ControllerAdvice`配合使用，该注解可以把异常处理器应用到所有Controller，而不是单个Controller

## HandlerAdaptor是啥

Handler(也就是Controller)返回啥的都用，如果直接调用，非常麻烦，就遵循适配器模式，搞了个HandlerAdaptor，从而通过一个接口，调用各种Controller

## SpringMVC的全局异常处理机制

除了自定义异常处理器之外，SpringMVC提供了四种异常处理器：
- DefaultHandlerExceptionResolver，默认的异常处理器，根据各个不同类型的异常，返回不同的异常视图
- SimpleMappingExceptionResolver，简单映射异常处理器。通过配置异常类和view的关系来解析异常
  - 直接显示异常的堆栈信息，没啥用
- ResponseStatusExceptionResolver，状态码异常处理器。解析带有`@ResponseStatus`注释类型的异常
  - 会显示HTTP的错误代码，以及异常的类型
- `ExceptionHandlerExceptionResolver`，注解形式的异常处理器。对`@ExceptionHandler`注解的方法进行异常解析
  - **也就是`@ControllerAdvice`+`@ExceptionHandler`的方式**
  - 非常灵活，可以跳转页面，也可以实现其他的一些操作

异常会在`doDispatch()`方法中，在try..catch块的catch块中被`this.processDispatchResult()`方法处理，在判断异常类型不是`ModelAndViewDefiningException`之后，通过`processHandlerException()`处理异常，其主要逻辑就是：**通过责任链模式，遍历异常处理器的集合，并尝试处理，不能处理则交给下一个异常处理器**

而异常处理器的加载则是在初始化`DispatchServlet`时，扫描带有`@ControllerAdvice`、`@ExceptionHandler`、`@ResponseStatus`这三个注解的方法，以及根据xml文件扫描，从而注册并初始化异常处理器。

通常将`ControllerAdvice`与`@ExceptionHandler`的方法配合使用，作为全局的异常处理器

https://blog.csdn.net/yehongzhi1994/article/details/106797360

# SpringBoot

## 为什么会有SpringBoot的出现

SpringBoot是建立在Spring的基础之上的，Spring是一个非常轻量级的容器，但Spring需要非常复杂的XML配置，在使用过程中很痛苦，因此推出了更简便的SpringBoot。

在Spring中，是通过maven管理依赖的包，并通过XML配置文件来管理对象的生命周期的

**版本控制** 

在SpringBoot中，SpringBoot会依赖`spring-boot-starter-parent`这个包，而其又依赖于`spring-boot-dependencies`这个POM工程，其中用`<properties>`标签统一了大量第三方包的版本，**从而实现对版本的统一控制**

除此之外，SpringBoot将所有的常见开发功能，分成了一个个**场景启动器（starter）**，这样我们需要开发什么功能，就导入什么场景启动器依赖即可。
> 例如spring-boot-starter-web，其中包含了spring-web和spring-webmvc，以及其他web应用的包

**自动装配**

SpringBoot通过`@EnableAutoConfiguration`启动自动配置，从而省去XML配置文件

这个注解中又包含其他的注解：
- `@AutoConfigurationPackage`
  - 其中又引入了`@Import(AutoConfigurationPackages.Register.class)`
    - 即==自动配置包注册器==
    - 主要执行的是**将启动类所在包及子包中的所有类注册到IOC容器中**
- `@Import(AutoConfigurationImportSelector.class)`
  - **自动装配的核心类**，==自动配置导入选择器==
  - 其中，**默认加载了大量配置类**，**这些配置类会自动地给我们加载每个场景需要的所有组件，并将它们配置好，从而省去XML配置文件**
    - 配置类的位置：`META-INF/spring.factories`文件
    > 实际上，SpringBoot并不会每次都把所有类都加载进来，这样会占用极大的内存
    > SpringBoot采用`@ConditionOnClass`等注解，检测配置的相关类是否存在，如果不存在，则不进行加载 

https://zhuanlan.zhihu.com/p/95217578

## SpringBoot是怎么处理高并发场景的

SpringBoot可以通过在方法上使用`@Async`注解，并通过`EnableAsync`开启异步，就可以异步地执行任务，并通过回调获得结果

## SpringBoot是怎样启动的

SpringBoot的启动类如下：
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

用于启动自动装配与包扫描的`@SpringBootApplication`在上面已经分析过了，这里着重分析`main()`方法中的内容

可见，使用过`SpringApplication`这个类的静态方法`run()`启动的，而其中使用的是`SpringApplication`实例的`run()`方法。
在其实例化时，主要包含以下操作：
1. 将主启动类设置到集合中，储存起来
2. 设置应用类型ApplicationType
   - NONE：只走正常的启动流程，不额外启动web容器
   - SERVLET：基于Servlet的Web程序，启动内嵌的Servlet Web容器（例如tomcat）
   - REACTIVE：基于Reactive的Web程序，
3. 设置初始化器（Initializers），用于启动过程
   - 用于在IOC容器刷新之前初始化一些组件 
   - 通过loadSpringFactories()方法，从`spring.factories`中**加载需要初始化的类的类名**
      > 因此，如果需要自定义一个ApplicationContextInitializer，只需要实现其接口，并在`spring.factories`中设置即可   
4. 设置一系列监听器，用于监听启动过程中的一些事件

在完成SpringApplication的实例化之后，就是`run()`方法的启动流程了：

1. 创建SpringBoot运行监听器
   -  用于监听ApplicationContext的启动过程，其中包含对应ApplicationContext启动过程的多个接口
   -  具体监听器的类名的获取同样是通过读取`spring.factories`实现的，并通过**反射**进行监听器的实例化
2. 启动上述监听器，广播`ApplicationStartingEvent`事件，从而发现监听该事件的监听器
3. 环境Environment构建
   - 主要用于加载系统配置以及用户的自定义配置(`application.properties`)
   - 同时广播一个`ApplicationEnvironmentPreparedEvent`事件，触发监听器
4. 创建IOC容器
   - 根据webApplicationType决定创建的类型（NONE、SERVLET、REACTIVE）  
5. IOC容器的前置处理
   - 在刷新容器之前做准备，其中有一个非常关键的操作：**将启动类注入容器**，为后续的自动化配置奠定基础
   -  重点步骤：
      1. 调用初始化器  
         - 这里调用的是SpringApplication实例化时，从`spring.factories`中读取得到的初始化器，直接遍历初始化即可
      2. 加载启动类，注入容器
         - 将主启动类加载到IOC容器中，作为后续自动配置的入口
      3. 广播ApplicationContextInitializedEvent 与 ApplicationPreparedEvent 
6. 调用`refresh()`方法，刷新容器，完成IOC容器的创建与Bean的初始化
7. IOC容器的后置处理 
   - 一个干涉接口 
8. 发出结束执行的事件ApplicationStartedEvent
   - 和之前几次广播不同，不是广播给SpringApplication中的监听器，而是在IOC容器中，通过@Component注入的监听器
9. 执行Runners
   - 用于定制一些额外的操作  

![](./../images/spring/springboot启动.jpg)

> 综合来看，大部分都是Spring框架背后的一些概念和实践方式，SpringBoot只是在这些概念和实践上对特定的场景事先进行了固化和升华，而也恰恰是这些固化让我们开发基于Sping框架的应用更加方便高效


## SpringBoot怎么处理异常

在SpringBoot中，可以通过try-catch的方法捕获异常，而SpringBoot本身其实也提供了全局的通用异常处理方案：

SpringBoot通过`@ControllerAdvice`注解开启全局异常的捕获，只需在自定义的方法上使用`@ExceptionHandler`注解，并定义捕获异常的类型，就可以对这种类型的异常进行统一的处理。这种方法也可以用于处理自定义异常（在项目中用了）




## 外挂的tomcat与SpringBoot内置的tomcat有什么区别

为什么要禁掉springboot的tomcat然后将项目部署到独立的tomcat当中？

在开发阶段，个人觉得使用内置的tomcat就足够了，当项目需要上线部署时，才打包部署在独立的tomcat中

打包方法：
1. 在pom.xml移除嵌入式tomcat插件
2. 添加servlet-api依赖
3. 更新SpringBoot启动类，主要是继承SpringBootServletInitializer，并重写configure()方法


## SpringBoot怎么调起tomcat

在SpringBoot启动类的`run()`方法中，会一并调用tomcat：
在SpringBoot的ApplicationContext的`refresh`方法中，进行嵌入式容器tomcat的创建：

1. SpringBoot会根据自定义配置决定`ApplicationContext`的类型，若为`SERVLET`，则ApplicationContext的类型为`ServletWebServerApplicationContext`
2. 完成ApplicationContext的创建后，会调用其`Refresh()`方法，进行刷新
3. 在进行Bean的实例化之前，ApplicationContext会调用`onRefresh()`方法，此处为`ServletWebServerApplicationContext`的`onRefresh()`方法
4. **其中，会通过相应工厂，进行`webServer`的创建，如果使用的是`tomcat`，即`tomcat`的创建，并进行tomcat的配置**。

> 这样一来，就完成了tomcat的创建

Spring Boot 以 **启动线程** 的 `Context ClassLoader` 作 为Tomcat的`WebApp ClassLoader`的**父类加载器**，而Tomcat的`WebApp ClassLoader`使用`TomcatEmbeddedWebAppClassLoader`。
所以整个项目的jar包的加载都是由Spring Boot的主线程Context ClassLoader完成的，于是Context ClassLoader就可以访问我们的Web容器下的所有资源了。

## springboot是怎么启动springmvc的

启动Spring MVC，首先需要启动一个Servlet container，以便与Client端转换Request和Response



# SpringCloud

### SpringCloud的常用组件有哪些

RPC：OpenFeign
注册中心：Eureka、Nacos
配置中心：SpringCloudConfig、Nacos
负载均衡：Nginx、Ribbon
微服务网关：Zuul、Gateway
服务保护组件：Hystrix、Sentinel

# tomcat

## 介绍一下tomcat

## tomcat的原理、结构

## tomcat的类加载器

![](./../images/spring/tomcat.jpeg)

其中：
- `Common ClassLoader`：Tomcat最基本的类加载器，加载路径中的Class可以被Tomcat容器本身及各个WebApp访问。
- `Catalina ClassLoader`：Tomcat容器私有的类加载器，加载路径中的Class对于WebApp不可见。
- `Shared ClassLoader`：各个WebApp共享的类加载器，加载路径中的Class对所有WebApp可见，但是对于Tomcat容器不可见。
- `WebApp ClassLoader`：各个WebApp私有的类加载器，加载路径中的Class只对当前WebApp可见，**各个项目就是通过各自的WebApp ClassLoader加载进入Tomcat容器的**。

## tomcat的设计模式了解吗
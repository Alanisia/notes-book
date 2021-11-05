<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [框架](#%E6%A1%86%E6%9E%B6)
  - [Spring](#spring)
    - [IOC/DI](#iocdi)
    - [Bean](#bean)
      - [生命周期](#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
      - [作用域](#%E4%BD%9C%E7%94%A8%E5%9F%9F)
    - [常用注入方式](#%E5%B8%B8%E7%94%A8%E6%B3%A8%E5%85%A5%E6%96%B9%E5%BC%8F)
    - [装配与自动装配](#%E8%A3%85%E9%85%8D%E4%B8%8E%E8%87%AA%E5%8A%A8%E8%A3%85%E9%85%8D)
    - [事务](#%E4%BA%8B%E5%8A%A1)
      - [隔离级别](#%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB)
      - [事务传播](#%E4%BA%8B%E5%8A%A1%E4%BC%A0%E6%92%AD)
    - [设计模式](#%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F)
    - [AOP](#aop)
      - [AOP基本概念](#aop%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
      - [AOP通知类型](#aop%E9%80%9A%E7%9F%A5%E7%B1%BB%E5%9E%8B)
      - [Spring AOP与AspectJ的区别](#spring-aop%E4%B8%8Easpectj%E7%9A%84%E5%8C%BA%E5%88%AB)
  - [SpringMVC](#springmvc)
    - [Spring MVC工作原理](#spring-mvc%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86)
    - [Spring MVC核心组件](#spring-mvc%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6)
  - [Spring Boot](#spring-boot)
    - [常用注解](#%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3)
  - [MyBatis](#mybatis)
    - [${}和#{}的区别](#%E5%92%8C%E7%9A%84%E5%8C%BA%E5%88%AB)
    - [MyBatis分页](#mybatis%E5%88%86%E9%A1%B5)
    - [MyBatis缓存](#mybatis%E7%BC%93%E5%AD%98)
  - [Netty](#netty)
    - [Netty事件驱动模型](#netty%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8%E6%A8%A1%E5%9E%8B)
    - [Netty线程模型](#netty%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B)
    - [异步处理](#%E5%BC%82%E6%AD%A5%E5%A4%84%E7%90%86)
    - [拆包与粘包处理](#%E6%8B%86%E5%8C%85%E4%B8%8E%E7%B2%98%E5%8C%85%E5%A4%84%E7%90%86)
    - [零拷贝](#%E9%9B%B6%E6%8B%B7%E8%B4%9D)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 框架

## Spring

### IOC/DI

控制反转(Inverse of Control)/依赖注入(Dependency Injection)，将传统的程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。

作用：

1. 管理对象的创建和依赖关系的维护
2. 解耦，由容器去维护具体的对象
3. 托管了类的产生过程

优点：

1. IOC能把代码量降低
2. 使应用容易测试，单元测试不再需要单例和JNDI查找机制
3. 最小的代价和最小的侵入性使松散耦合得以实现
4. IOC容器支持加载服务时的饿汉式初始化和懒加载

### Bean

#### 生命周期

1. Spring启动，查找并加载需要被Spring管理的Bean，进行Bean实例化；
2. Bean实例化后，对Bean的引用和值注入到Bean属性中；
3. 如果Bean实现了`BeanNameAware`接口，Spring将Bean的Id传递给`setBeanName()`方法；
4. 如果Bean实现了`BeanFactoryAware`接口，Spring将调用`setBeanFactory()`方法，将BeanFactory容器实例传入；
5. 如果Bean实现了`ApplicationContextAware`接口，Spring将调用Bean的`setApplicationContext()`方法，将Bean所在应用的上下文引用传入进来； 
6. 如果Bean实现了`BeanPostProcessor`接口，Spring将调用它们的`postProcessBeforeInitialization()`方法；
7. 如果Bean实现了`InitializingBean`接口，Spring将调用它们的`afterPropertiesSet()`方法，类似地，如果Bean使用init-method声明了初始化方法，该方法也会被调用；
8. 如果Bean实现了`BeanPostProcessor`接口，Spring将调用它们的`postProcessAfterInitialization()`方法；
9. 此时，Bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到应用上下文被销毁；
10. 如果Bean实现了`DispoableBean`接口，Spring将调用它的`destroy()`方法，同样，如果Bean使用了destroy-method声明销毁方法，该方法也会被调用。

#### 作用域

可使用`@Scope`注解声明Bean作用域，默认`singleton`。

1. singleton：唯一Bean实例，Spring中的Bean默认都是单例的；
2. prototype：每次请求都会创建一个新的Bean实例；
3. request：每一次HTTP请求都会产生一个新的Bean，该Bean仅在当前HTTP Request内有效；
4. session：每一次HTTP请求都会产生一个新的Bean，该Bean仅在当前HTTP Session内有效；
5. global-session：全局Session作用域，仅仅在基于portlet的web应用才有意义，Spring 5已废除portlet。Portlet是能够生成语义代码片段的小型Java Web插件，它们基于portlet容器，可以像servlet一样处理HTTP请求，但是与servlet不同，每个portlet都有不同的会话。

### 常用注入方式

- 构造器依赖注入
- Setter方法注入
- 基于注解注入

### 装配与自动装配

`@Autowired`与`@Resource`的区别：

1. `@Autowired`默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置其`required`属性为`false`）；
2. `@Resource`默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

### 事务

#### 隔离级别

`TransactionDefinition`接口中定义了五个表示隔离级别的常量：

- `TransactionDefinition.ISOLATION_DEFAULT`：使用后端数据库默认的隔离级别
- `TransactionDefinition.ISOLATION_READ_UNCOMMITTED`：最低隔离级别，读取未提交
- `TransactionDefinition.ISOLATION_READ_COMMITTED`：读取已提交
- `TransactionDefinition.ISOLATION_REPEATABLE_READ`：可重复读
- `TransactionDefinition.ISOLATION_SERIALIZABLE`：最高隔离级别，串行化

#### 事务传播

Spring事务传播行为指的是当多个事务同时存在时，Spring如何管理这些事务。

1. `TransactionDefinition.PROPAGATION_REQUIRED`：如果当前没有事务，则新建一个事务，如果当前存在事务，就加入该事务
2. `TransactionDefinition.PROPAGATION_SUPPORTS`：支持当前事务，如果当前存在事务，就加入该事务，否则以非事务执行
3. `TransactionDefinition.PROPAGATION_MANDATORY`：支持当前事务，如果当前存在事务，就加入该事务，否则抛出异常
4. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`：创建新事务，无论当前存不存在事务
5. `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式执行操作，若当前存在事务，就把当前事务挂起
6. `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式执行操作，若当前存在事务，则抛出异常
7. `TransactionDefinition.PROPAGATION_NESTED`：如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则按REQUIRED属性执行

### 设计模式

- 工厂模式
- 代理模式
- 单例模式
- 模板模式
- 包装器模式
- 观察者模式
- 适配器模式

### AOP

AOP（Aspect-Oriented Programming，面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。

Spring AOP基于动态代理，若要代理的对象实现了某个接口，Spring AOP会使用JDK动态代理去创建代理对象，而对于没有实现接口的对象就无法使用JDK动态代理去代理了，此时Spring AOP会使用Cglib生成一个被代理对象的子类来进行代理。另外，Spring AOP已经集成AspectJ。

使用AOP后可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量，在需要增加新功能时也方便，并且提高了系统扩展性。日志功能、事务管理等场景都用到了AOP。

#### AOP基本概念

- 切面(Aspect)：官方的抽象定义为“一个关注点的模块化，这个关注点可能会横切多个对象”
- 连接点(Joinpoint)：程序执行过程中的某个行为
- 通知(Advice)：“切面”对于某个“连接点”所产生的动作
- 切入点(Pointcut)：匹配连接点的断言，在AOP中通知和一个切入点表达式关联
- 目标对象(Target Object)：被一个或者多个切面所通知的对象
- 织入(Weaving)：将切面应用到目标对象并导致代理对象创建的过程
- AOP代理(AOP Proxy)：在Spring AOP中有两种代理方式，JDK动态代理和CGLIB代理

#### AOP通知类型

1. 前置通知
2. 后置通知
3. 环绕通知
4. 返回后通知
5. 抛出异常后通知

#### Spring AOP与AspectJ的区别

1. Spring AOP属于运行时增强，AspectJ属于编译时增强；
2. Spring AOP基于代理，AspectJ基于字节码操作；
3. AspectJ相比Spring AOP功能更强大，但Spring AOP相对来说更简单；
4. 切面较少时两者性能差异不大，切面太多时AspectJ比Spring AOP快。

## SpringMVC

MVC是Model-View-Controller的简称，是一种架构模式，它分离了表现和交互。被分为三个核心部分：

- 模型（Model）：***TODO***
- 视图（View）：***TODO***
- 控制器（Controller）：***TODO***

### Spring MVC工作原理

1. 客户端发送请求
2. 前端控制器DispatcherServlet接受用户请求
3. 找到处理器映射HandlerMapping解析请求对应的Handler
4. HandlerAdaptor会根据Handler来调用真正的处理器处理请求，并处理相应的业务逻辑
5. 处理器返回一个模型视图ModelAndView
6. 视图解析器进行解析
7. 返回一个视图对象
8. 前端控制器DispatcherServlet渲染数据（Model）
9. 将得到的视图对象返回给用户

### Spring MVC核心组件

- `DispatcherServlet`：前端控制器，相当于Spring MVC的中央处理器，是整个控制流程的中心，由它调用其它组件处理用户的请求，降低组件间的耦合性
- `HandlerMapping`：处理器映射器，根据请求的URL查找Handler（Controller）
- `HandlerAdaptor`：处理器适配器，按照特定规则（HandlerAdaptor要求的规则）去执行Handler，是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行
- `Handler`：处理器，编写具体的业务逻辑
- `ViewResolver`：视图解析器，进行视图解析，根据逻辑视图名解析成真正的视图（View），ViewResolver首先根据逻辑视图名解析成物理视图名即具体之页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展现给用户
- `View`：视图，具体的页面

## Spring Boot

Spring Boot是一个快速开发框架，快速地将一些常用的第三方依赖整合（通过Maven父子工程的形式），简化XML配置，全部采用注解形式，最终以Java应用程序进行执行。

### 常用注解

- `@SpringBootApplication`：可以看作是`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`
- `@ImportAutoConfiguration`

***TODO***

## MyBatis

***TODO***

### ${}和#{}的区别

- `#{}`：预处理编译，处理`#{}`时会将其替换为`?`号，调用`PreparedStatement`的`set()`方法来赋值，此方法可以有效防止SQL注入，提高系统安全性
- `${}`：字符串替换，将`${}`替换成变量的值

### MyBatis分页

***TODO***

### MyBatis缓存

1. 一级缓存

    基于PerpetualCache的HashMap本地缓存，其存储作用域为`session`，当`session`flush或close之后，该`session`中的所有cache就将清空，默认打开一级缓存；

2. 二级缓存

    与一级缓存机制相同，默认也采用PerpetualCache，HashMap存储，不同在于其作用域为mapper(namespaces)，并且可自定义存储源，如EhCache；默认不打开二级缓存，要开启之，使用二级缓存属性类需要实现Serializable序列化接口（用来保存对象状态），可在其映射文件中配置。

对于缓存更新机制，当某一作用域进行了C/U/D操作后，默认该作用域下所有select中的缓存将被clear。

## Netty

一款基于NIO（Nonblocking IO）开发的网络通信框架，对比BIO（Blocking IO），并发性能得到很大的提高，在快速和易用性的同时并未丧失可维护性和性能等优势。

特点：

1. 高并发
2. 传输快
3. 封装好

重要组件：

1. Channel：Netty的网络操作接口，包括基本的IO操作如`bind()`、`connect()`、`read()`、`write()`等；常用的Channel接口实现类有`NioServerSocketChannel`和`NioSocketChannel`；
2. EventLoop：定义了Netty的核心抽象，用于处理连接的生命周期中所发生的事件，即其主要作用为负责监听网络事件并调用事件处理器进行IO相关操作的处理；
3. ChannelFuture：由于Netty是异步非阻塞的，无法立刻得到操作结果，可以通过该接口的`addListener()`注册一个`ChannelFutureListener`进行监听；
4. ChannelHandler：消息的具体处理器，负责处理读写操作、客户端连接等；
5. ChannelPipeline：ChannelHandler的链，提供了一个容器并定义了用于沿着链传播入站和出站时间流的API，当Channel被创建时，它被自动分配到它专属的ChannelPipeline；可以在ChannelPipeline上通过`addLast()`方法添加一个或多个ChannelHandler，因为一个数据或者事件可能会被多个handler处理，当一个ChannelHandler处理完数据之后就交给下一个ChannelHandler；
6. Bootstrap/ServerBootstrap：前者是客户端的启动/引导类，后者是服务端的启动/引导类，Bootstrap只需配置一个线程组，ServerBootstrap需要配置两个，一个用于接受连接，一个用于具体的处理。

### Netty事件驱动模型

### Netty线程模型

基于主从Reactors多线程模型，Netty对此作了修改，其中主从Reactor多线程模型有多个Reactor：

- MainReactor：负责客户端的连接请求，并将请求转发给SubReactor
- SubReactor：负责相应通道的IO读写请求
- 非IO请求（具体逻辑处理）的任务会直接写入队列，等待worker threads进行处理

```java
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workGroup = new NioEventLoopGroup();
```

以上代码中，`bossGroup`和`workGroup`均是线程池，

- `bossGroup`线程池绑定某个端口后获得其中一个线程作为MainReactor，专门处理端口的accept事件，每个端口对应一个boss线程
- `workGroup`线程池会被各个SubReactor和worker线程充分利用

_EventLoopGroup与EventLoop关系：EventLoopGroup包含多个EventLoop（每个EventLoop通常包含一个线程），EventLoop处理的事件都将在它专有的线程上处理，从而保证线程安全。_

### 异步处理

Netty的IO操作是异步的，包括`bind()`，`connect()`，`write()`等操作会简单地返回一个`ChannelFuture`，调用者不能立刻获得结果，通过Future-Listener机制，用户可以方便地主动获取或者通过通知机制获得IO操作结果。

相比传统阻塞IO，异步处理的好处是不会造成线程阻塞，线程在IO操作期间可以执行别的程序，在高并发情形下会更稳定和更高的吞吐量。

### 拆包与粘包处理

Netty对解决拆包粘包问题做了抽象，提供一些解码器来解决拆包粘包问题，如：

- `LineBasedFrameDecoder`：以行为单位（换行符）进行数据包的解码
- `DelimiterBasedFrameDecoder`：以特殊符号作为分隔来进行数据包的解码
- `FixedLengthFrameDecoder`：以固定长度进行数据包的解码
- `LengthFieldBasedFrameDecoder`：适用于消息头包含消息长度的协议

也可以通过自定义序列化编解码器来解决拆包粘包问题。

### 零拷贝

操作系统层面的零拷贝通常指避免在用户态和内核态之间来回拷贝数据，Netty层面主要体现在对数据操作的优化：

- 使用了Netty提供的`CompositeByteBuf`类，可以将多个ByteBuf合并为一个逻辑上的ByteBuf，避免了各个ByteBuf的拷贝；
- ByteBuf支持slice操作，因此可以将ByteBuf分解为多个共享同一个存储区域的ByteBuf，避免内存拷贝；
- 通过`FileRegion`包装的`FileChannel.transferTo`实现文件传输，可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write的方式导致的内存拷贝问题。

### 网络I/O框架与Web容器的比较

- 网络I/O框架直接封装传输层的TCP/UDP协议，而Web容器是基于Servlet的，Servlet 3.0基于传输层，封装了应用层协议，包括HTTP；
- 网络I/O框架无需Web容器的支持，可以直接在程序中应用这些框架实现客户端和服务器通信，而Web容器一般部署在服务器端，对Servlet 3.0提供了不同的实现。
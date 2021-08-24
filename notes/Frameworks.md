# 框架

## Spring

### IOC/DI

控制反转(Inverse of Control)/依赖注入(Dependency Injection)，

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

1. singleton：唯一Bean实例，Spring中的Bean默认都是单例的；
2. prototype：每次请求都会创建一个新的Bean实例；
3. request：每一次HTTP请求都会产生一个新的Bean，该Bean仅在当前HTTP Request内有效；
4. session：每一次HTTP请求都会产生一个新的Bean，该Bean仅在当前HTTP Session内有效；
5. global-session：全局Session作用域，仅仅在基于portlet的web应用才有意义，Spring 5已废除portlet。Portlet是能够生成语义代码片段的小型Java Web插件，它们基于portlet容器，可以像servlet一样处理HTTP请求，但是与servlet不同，每个portlet都有不同的会话。

### 常用注入方式

- 构造器依赖注入
- Setter方法注入
- 基于注解注入

### 事务

#### 隔离级别

`TransactionDefinition`接口中定义了五个表示隔离级别的常量：

- `TransactionDefinition.ISOLATION_DEFAULT`：使用后端数据库默认的隔离级别
- `TransactionDefinition.ISOLATION_READ_UNCOMMITTED`：最低隔离级别，读取未提交
- `TransactionDefinition.ISOLATION_READ_COMMITTED`：读取已提交
- `TransactionDefinition.ISOLATION_REPEATABLE_READ`：可重复读
- `TransactionDefinition.ISOLATION_SERIALIZABLE`：最高隔离级别，串行化

#### 事务传播

***TODO***

### 设计模式

- 工厂模式
- 代理模式
- 单例模式
- 模板模式
- 包装器模式
- 观察者模式
- 适配器模式

### AOP

AOP（Aspect-Oriented Programming，面向切面编程）

#### AOP基本概念

- 切面(Aspect)：官方的抽象定义为“一个关注点的模块化，这个关注点可能会横切多个对象”
- 连接点(Joinpoint)：程序执行过程中的某个行为
- 通知(Advice)：“切面”对于某个“连接点”所产生的动作
- 切入点(Pointcut)：匹配连接点的断言，在AOP中通知和一个切入点表达式关联
- 目标对象(Target Object)：被一个或者多个切面所通知的对象
- 织入(Weaving)
- AOP代理(AOP Proxy)：在Spring AOP中有两种代理方式，JDK动态代理和CGLIB代理

#### AOP代理方式

- JDK动态代理
- CGLIB代理

#### AOP通知类型

1. 前置通知
2. 后置通知
3. 环绕通知
4. 返回后通知
5. 抛出异常后通知

## SpringMVC

MVC是Model-View-Controller的简称，是一种架构模式，它分离了表现和交互。被分为三个核心部分：

- 模型（Model）：***TODO***
- 视图（View）：***TODO***
- 控制器（Controller）：***TODO***

### Spring MVC工作原理

***TODO***

### Spring MVC核心组件

***TODO***

## Spring Boot

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

***TODO***

2. 二级缓存

***TODO***

## Netty

***TODO***
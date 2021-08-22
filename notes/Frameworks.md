# 框架

## Spring

### IOC/DI

控制反转(Inverse of Control)/依赖注入(Dependency Injection)，

### Bean

#### 生命周期

#### 作用域

### 常用注入方式

### 事务

#### 隔离级别


#### 事务传播

### 设计模式

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

## SpringBoot

## MyBatis

## Netty
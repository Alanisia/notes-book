# 面向对象

## 特征

- 封装
- 继承
- 多态

## 与面向过程的区别

# String

## String, StringBuilder, StringBuffer区别

- String不可变
- StringBuilder可变，线程不安全，性能好
- StringBuffer可变，线程安全，性能较StringBuilder差

# 包装类

## 基本类型

int, short, char, byte, boolean, long, double, float

对应的包装类为

Integer, Short, Character, Byte, Boolean, Long, Double, Float

其中Integer, Short, Long, Double, Float继承自Number

- 自动拆箱
- 自动装箱

# 序列化

## trancient关键字

# 异常

# 反射

# 动态代理

## 静态代理

## 动态代理

1. JDK

核心接口：

```java
public interface InvocationHandler {
  /**
   * @param proxy 被代理的类实例
   * @param method 调用被代理类的方法
   * @param args 该方法需要的参数
   */
  public Object invoke(
    Object proxy,
    Method method,
    Object[] args
  ) throws Throwable;
}
```

使用方法：

实现该接口，重写invoke方法，在invoke方法调用被代理类的方法并获取返回值，并可以在调用方法的前后做一些其他的事情，实现动态代理。

```java
/**
 * @param loader 被代理的类或加载器
 * @param interfaces 被代理类的接口数组
 * @param h 调用处理器类的对象实例
 */
public static Object newProxyInstance(
  ClassLoader loader,
  Class<?>[] interfaces,
  InvocationHandler h
) throws IllegalArgumentException
```

该方法会返回一个被修改过的类的实例，从而可以自由地调用该实例的方法。

2. cglib

3. javassist
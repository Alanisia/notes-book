# Java基础

## 面向对象

### 特征

- 封装：把数据和操作数据的方法封装起来，对数据的访问只能通过已定义的接口
- 继承：是从已有类得到继承信息创建新类的过程，提供继承信息的类称为父类（超类），得到继承信息的类成为子类（派生类）
- 多态：分为编译时多态和运行时多态

  - 编译时多态（方法重载）：同一个类中同名方法具有不同的参数列表，不能根据返回类型区分（函数调用时不能指定类型信息，编译器不知你要调用哪个函数）
  - 运行时多态（方法重写）：子类重写父类的方法具有相同的返回类型、更好的访问权限；继承父类并重写父类的方法，并用父类型引用子类型对象，这样同样的引用调用同样的方法就会根据子类对象的不同而表现出不同的行为

### 与面向过程的区别

面向过程性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源，但没有面向对象易维护、易扩展、易复用。

### 接口和抽象类

区别：

1. 接口方法默认为`public`，所有方法在接口中不能有实现（JDK8开始可以有方法的默认实现），而抽象类可以有非抽象方法；抽象类中的抽象方法可以有`public`，`protected`，`default`修饰符（不能是`private`）
2. 接口中除了`static`，`final`变量，不能有其他变量，抽象类则不一定
3. 一个类能实现多个接口，但只能继承一个抽象类；接口本身可以通过`extends`关键字继承多个接口
4. 从设计层面来说，抽象是对类的抽象，是一种模板设计；而接口是对行为的抽象，是一种行为的规范

## String

**String, StringBuilder, StringBuffer区别**

- `String`不可变
- `StringBuilder`可变，线程不安全，性能好
- `StringBuffer`可变，线程安全，性能较`StringBuilder`差

## 包装类

基本类型`int`, `short`, `char`, `byte`, `boolean`, `long`, `double`, `float`

对应的包装类为`Integer`, `Short`, `Character`, `Byte`, `Boolean`, `Long`, `Double`, `Float`

其中`Integer`, `Short`, `Long`, `Double`, `Float`继承自`Number`

- 自动拆箱：将基本类型用它们的引用类型包装起来
- 自动装箱：将包装类型转换为基本数据类型

## static关键字

## 克隆

- 浅克隆（Shallow Clone）：
- 深克隆（Deep Clone）：

`Cloneable`接口和`clone()`方法

***TODO***

## 序列化

***TODO***

序列化的实现：将需要被序列化的类实现`java.io.Serializable`接口

什么时候需要序列化：

1. 把内存中的对象状态保存到文件或数据库的时候
2. 用套接字在网络上传输对象的时候
3. 通过RMI传输对象的时候

### trancient关键字

***TODO***

## 异常

***TODO***

## 泛型

***TODO***

## 反射

***TODO***

## 代理

***TODO***

### 静态代理

***TODO***

### 动态代理

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
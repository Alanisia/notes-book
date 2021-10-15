# 设计模式

## 单例模式

特点：

- 保证一个类只有一个实例；
- 要提供一个访问该类对象实例的全局访问点。

**单例模式实现**

- 饿汉模式（预先加载方式）

    ```java
    public class Singleton {
        private static Singleton instance = new Singleton();
        private Singleton() {}
        public static Singleton getInstance() {
            return instance;
        }
    }
    ```

    对象预先加载，线程是安全的，在类创建好的同时对象生成，调用获得对象实例的方法反应速度快，代码简练。缺点是资源效率不高，可能`getInstance()`永远不会被执行，但执行该类的其他静态方法或者加载了该类，该实例仍然初始化。

- 懒汉模式（延迟加载方式）

    ```java
    public class Singleton {
        private static Singleton instance = null;
        private Singleton() {}
        public static Singleton getInstance() {
            if (instance == null) {
                instance = new Instance();
                return instance;
            }
        }
    }
    ```

    对象延迟加载，效率高，只在使用时才实例化对象，但若设计不当线程会不安全（以下双重校验锁法可以解决该问题），代码相比饿汉式复杂，第一次加载类对象时反应不快。

- 双重校验锁

    ```java
    public class Singleton {
        // volatile作用：防止指令重排
        private volatile static Singleton instance;
        private Singleton() {}
        public static Singleton getInstance() {
            // 先判断是否为空再进入同步块
            if (instance == null) {
                // 类对象加锁
                synchonized (Singleton.class) {
                    if (instance == null) {
                        /* 
                         * 分三步执行：
                         * 1. 为instance分配空间
                         * 2. 初始化instance
                         * 3. 将instance指向分配的内存地址
                         * JVM可能会对其进行指令重排变成1->3->2，多线程环境下可能会导致线程获得还未初始化的实例
                         */
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
    ```

    资源利用率高，不执行`getInstance()`就不会被实例化，可以执行该类的其他静态方法，但第一次加载时不够快，多线程使用不必要的同步开销大。

    _两次判空的原因：_

    - _内层判空：加入多个线程已经通过外层判断，如果内层不加判断，会进行多次实例化；_
    - _外层判空：提高效率，如果已经实例化则直接返回实例，无需同步。_

## 工厂模式

***TODO***

## 观察者模式

***TODO***
# 设计模式

## 单例模式

单例模式实现

- 懒汉模式

    ```java
    ```

- 饿汉模式

    ```java
    ```

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

## 工厂模式

## 观察者模式
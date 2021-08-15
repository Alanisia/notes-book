# Java并发

## 进程&线程

- 进程：程序运行的基本单位
- 线程：

## Thread类

- Runnable接口

## synchronized关键字

用法：

1. 通过对一个对象进行加锁来实现同步：

```java
synchronized (object) {}
```

2. 对一个方法进行加锁实现同步：

```java
public synchronized void method() {}
```

_无论是对对象加锁还是对方法加锁，本质上都是对对象加锁。对于方法2，虚拟机会根据synchronized修饰的是实例方法还是静态方法，去取对应的实例对象或类对象进行加锁。_

### 实现原理

### 锁升级过程

对象头结构

|长度|内容|说明|
|----|----|----|
|32/64bit|Mark Word|hashCode GC分代年龄，锁信息|
|32/64bit|Class Pointer|指向对象类型的指针|
|32/64bit|Array Length|数组长度（当对象为数组时）|

由此可以看出，锁信息存在Mark Word里。当创建一个对象时，该对象的Mark Word关键数据如下：

|bit fields|是否偏向锁|锁标志位|
|---|---|---|
|hash|0|01|

偏向锁状态为01表示该对象尚未被加上偏向锁（1表示被加上偏向锁）。

**偏向锁**

偏向锁会偏向于第一个获得它的线程，在接下来的执行过程中，假如该锁未被其他线程获取，没有其他线程来竞争该锁，那么持有偏向锁的线程将永远不需要进行同步操作。

- 锁膨胀：当有两个以上的线程竞争锁，则偏向锁失效，此时锁膨胀为轻量级锁
- 锁撤销：撤销失效的偏向锁

**轻量级锁**

偏向锁升级为轻量级锁后的Mark Word部分数据如下：

|bit fields|锁标志位|
|---|---|
|指向LockRecord的指针|00|

标志位00表示轻量级锁，轻量级锁主要有两种：

1. 自旋锁

2. 自适应自旋锁


**重量级锁**

轻量级锁膨胀后升级为重量级锁。重量级锁依赖对象内部的monitor锁实现，monitor锁又依赖于操作系统的MutexLock实现，所以重量级锁又被称为互斥锁。（JDK1.6以前的synchronized为重量级锁）

当轻量级所经过锁撤销等步骤升级为重量级锁之后，它的Mark Word部分数据大体如下：

|bit fields|锁标志位|
|---|---|
|指向Mutex的指针|10|

重量级锁开销大的原因：




### `wait()`&`notify()`&`notifyAll()`

这三个方法是Object类当中用于进行线程调度的方法，都必须在synchronized标示的代码块中使用。

## 悲观锁

## volatile

volatile关键字用于保证程序指令的有序性和可见性。被volatile修饰的变量，具有以下两条特性：

 1. 禁止指令重排序；

 2. 保证不同线程对该变量操作的内存可见性。

## 线程池

线程池提供了一种限制和管理资源（包括执行一个任务）的方式。每个线程池还维护了一些基本统计信息，例如：已完成任务的数量。

为什么要使用线程池：

1. 降低资源消耗
2. 提高响应速度
3. 提高线程的可管理性

### 创建

两种创建线程池的方式：

1. 通过创建ThreadPoolExecutor对象

	ThreadPoolExecutor类的七个参数：

	- corePoolSize：线程池核心线程数量
	- maximumPoolSize：线程池最大线程数量
	- keepAliveTime：多余的空闲线程存活时间
	- unit：keepAliveTime的单位
	- workQueue：任务队列，用于保存等待任务的阻塞队列
	- threadFactory：线程工厂
	- handler(rejectPolicy)：拒绝策略

2. 通过Executors创建

	1. Executors.newFixedThreadPool()

	2. Executors.newSingleThreadPool()

	3. Executors.newCachedThreadPool()

_注：尽量不要使用该方法创建线程池_

### 阻塞队列

1. ArrayBlockingQueue

2. LinkedBlockingQueue

3. SynchronousQueue

4. PriorityBlockingQueue

### 拒绝策略

四种拒绝策略：

- DiscardOldestPolicy：丢弃队列里最近任务并执行
- DiscardPolicy：丢弃任务
- CallRunsPolicy：调用任务所在线程执行任务
- AbortPolicy：抛出异常

## 乐观锁

总是假设最好的情况，每次拿数据的时候都认为别人不会修改，所以不会上锁，但在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号和CAS算法实现。乐观锁多用于读多写少的情景，可以提高吞吐量。

 - 版本号

  一般是在数据库表中加上一个version字段，表示表被修改的次数，每次表被修改的时候version + 1。当一个线程读取这个数据库表的时候也会一并读取version值，在提交更新时，会检查当前的version值是否与刚才读取过的version值相等，若相等才更新表，否则尝试更新操作，直至更新成功。

 - CAS(Compare And Swap，比较并替换)

**CAS缺点**

1. ABA问题

2. 循环时间开销过大

3. 只能保证一个共享变量的原子操作


## Atomic类

JUC下的原子类可以分为4类：

1. 基本类型

2. 数组类型

3. 引用类型

4. 对象的属性修改类型


## AQS(AbstractQueuedSynchronizer，抽象队列同步器)

**CLH队列**

CLH队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

### Lock



#### Lock与synchronized区别

1. synchronized是关键字，Lock是类
2. synchronized在线程执行完毕或线程产生异常时释放锁，Lock需要手动释放锁否则容易产生死锁
3.
4.
5.
6. synchronized通过Object类的`wait()`/`notify()`/`notifyAll()`调度，Lock类通过Condition类调度
7. synchronized是非公平锁，Lock是公平锁
8. synchronized少量同步，Lock大量同步
9. 
10. synchronized底层通过操作系统指令码控制，Lock通过CAS乐观锁实现

### Condition（条件变量）



### ReentrantLock（可重入锁）



### Semaphore（信号量）



### CountDownLatch（线程发令枪）

应用场景：

1. 死锁检测
2. 实现多个线程开始执行任务的最大并行性
3. 某一个线程在开始运行前等待n个线程执行完毕

### CyclicBarrier

应用场景：用于多线程计算数据，最后合并计算结果的应用场景。

## 公平锁&非公平锁

- 公平锁：
- 非公平锁：

## ThreadLocal


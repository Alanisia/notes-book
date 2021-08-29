# Java并发

## Thread类

### 线程的状态

- 创建（NEW）
- 就绪（RUNNABLE）
- 阻塞（BLOCKED）
- 计时等待（TIME_WAITING）
- 等待（WAITING）
- 死亡（DEAD）

### 上下文切换

***TODO***

### 方法

```java
public void sleep() throws InterruptedException
```

***TODO***

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

***TODO***

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

***TODO***

### `wait()`&`notify()`&`notifyAll()`

这三个方法是Object类当中用于进行线程调度的方法，都必须在synchronized标示的代码块中使用。

## 悲观锁

***TODO***

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

总是假设最好的情况，每次拿数据的时候都认为别人不会修改，所以不会上锁，但在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号和CAS算法实现。乐观锁多用于**读多写少**的情景，可以提高吞吐量。

- 版本号

  一般是在数据库表中加上一个version字段，表示表被修改的次数，每次表被修改的时候version + 1。当一个线程读取这个数据库表的时候也会一并读取version值，在提交更新时，会检查当前的version值是否与刚才读取过的version值相等，若相等才更新表，否则尝试更新操作，直至更新成功。

- CAS(Compare And Swap，比较并替换)

	***TODO***

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

***TODO***

## AQS(AbstractQueuedSynchronizer，抽象队列同步器)

AQS核心思想是，如果被请求的共享资源空闲，则将请求资源的线程设置为有效的工作线程，并将共享资源设定为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配机制，这个机制AQS使用CLH队列锁实现，即将暂时获取不到锁的线程加入到队列中。

> **CLH队列**
>
> CLH队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state; // 共享变量，使用volatile保证线程可见性
```

状态信息通过`protected`类型的`getState`，`setState`，`compareAndSetState`进行操作。

#### AQS对资源的共享方式

- Exclusive（独占）

	只有一个线程能执行，如`ReentrantLock`。又分公平锁与非公平锁：

	- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
	- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- Share（共享）：多个线程同时执行，如`CountDownLatch`、`Semaphore`、`ReadWriteLock`、`CyclicBarrier`。

`ReentrantReadWriteLock`可以看作是组合式，因为`ReentrantReadWriteLock`也就是读写锁允许多个线程同时对某一资源进行读。

### Lock

***TODO***

#### Lock与synchronized区别

1. synchronized是关键字，Lock是类
2. synchronized在线程执行完毕或线程产生异常时释放锁，Lock需要手动释放锁否则容易产生死锁
3. synchronized在需要同步的对象加入此控制，Lock一般使用ReentrantLock，手动加解锁
4. synchronized不可判断锁的状态，Lock可以
5. 对于synchronized，当一个线程获得锁，另一个线程需要等待，当获得锁的线程阻塞，则另一线程须一直等待；对于Lock，当一个线程获得锁，另一个线程会尝试判断，不会一直等待
6. synchronized通过Object类的`wait()`/`notify()`/`notifyAll()`调度，Lock类通过Condition类调度
7. synchronized是非公平锁，Lock是公平锁
8. synchronized不可中断， Lock可中断
9. synchronized少量同步，Lock大量同步
10. synchronized底层通过操作系统指令码控制，Lock通过CAS乐观锁实现

### Condition（条件变量）

同步调度方法：

```java
await();
```

***TODO***

```java
signal();
```

***TODO***

```java
signalAll();
```

***TODO***

### ReentrantLock（可重入锁）

***TODO***

### ReadWriteLock（读写锁）

***TODO***

### Semaphore（信号量）

允许多个线程同时访问。`synchronized`和`ReentrantLock`都是一次只允许一个线程访问某个资源，`Semaphore`可以指定多个线程同时访问某个资源。

### CountDownLatch（倒计时器）

用于协调多个线程之间的同步。这个工具常用于控制线程的等待，它可以让某个线程等待直到倒计时结束，再开始执行。

应用场景：

1. 死锁检测
2. 实现多个线程开始执行任务的最大并行性
3. 某一个线程在开始运行前等待n个线程执行完毕

### CyclicBarrier（循环栅栏）

应用场景：用于多线程计算数据，最后合并计算结果的应用场景。

## ThreadLocal

ThreadLocal类能够实现每一个线程都有自己的专属本地变量，让每个线程绑定自己的值。创建一个ThreadLocal变量，那么访问该变量的每个线程都有这个变量的本地副本，并可以使用`get()`和`set()`来获取默认值或者将其值修改为当前线程所存副本之值，从而避免线程安全问题。

### ThreadLocal原理

***TODO***

### ThreadLocal内存泄露问题

ThreadLocalMap中使用的key是弱引用，而value是强引用，所以，如果ThreadLocal没有被外部强引用的情况下，在垃圾回收的时候，key会被清理掉而value没有被清理。这样一来ThreadLocalMap就会出现key为null的Entry，如果不做任何措施则value永远不会被GC，这个时候就容易产生内存泄露。

ThreadLocalMap实现中已经考虑这种情况，在调用`set()`，`get()`，`remove()`时会清理掉key为null的记录。使用完ThreadLocal方法后最好手动调用`remove()`方法。

## Future

JUC包下的一个接口。对于某些耗时的操作，如果一直原地等待其执行完毕，会使得程序的执行效率大大降低，这时可以把该耗时任务放到子线程去执行，再通过Future去控制子线程执行的过程，最后获取结果，使程序执行效率提高，是一种异步思想。对于具体的Runnable或Callable任务的执行结果进行获取、取消、查询是否完成等操作，必要时可以通过`get()`方法获取执行结果，该方法会阻塞直到任务返回结果。

```java
boolean cancel(boolean mayInterruptIfRunning);
```

用来取消任务，取消成功返回`true`。参数`mayInterruptedRunning`表示是否允许取消正在执行却没有执行完毕的任务，如设置为`true`则表示可以取消。若任务已完成，则无论该参数为`true`或`false`，此方法肯定返回`false`；若任务尚未执行，则无论该参数为`true`或`false`，此方法肯定返回`true`；若参数`mayInterruptedRunning`设置为`true`，则该方法返回`true`，否则返回`false`。

```java
boolean isCancelled();
```

表示任务是否被取消成功，如果在任务正常完成前被取消成功，返回`true`。

```java
boolean isDone();
```

表示任务是否已经完成，若任务完成则返回`true`。

```java
V get() throws InterruptedException, ExecutionException;
```

用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回。

```java
V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
```

用来获取执行结果，如果在指定时间内还未获取到结果，直接返回`null`。

### FutureTask

实现了`RunnableFuture`接口，而`RunnableFuture`接口继承了`Runnable`和`Future`接口。该类是`Future`接口的唯一实现类。


# Java并发

## Thread类

### 线程的状态

- 创建（NEW）：初始状态，线程被构建，但尚未调用`start()`方法
- 就绪（RUNNABLE）：运行状态，包含就绪和运行
- 阻塞（BLOCKED）：阻塞状态，表示线程阻塞于锁
- 等待（WAITING）：等待状态，表示当前线程需要等待其他线程做出一些特定动作（通知或中断）
- 超时等待（TIME_WAITING）：超时等待状态，不同于WAITING，可以在指定时间自行返回
- 死亡（TERMINATED）：终止状态，表示当前线程已执行完毕

### 方法

```java
public static native void sleep(long millis) throws InterruptedException;
```

在指定的毫秒数使当前线程休眠，进入阻塞状态（暂停执行），若线程在睡眠状态被中断会抛出`InterruptedException`异常。另还有方法

```java
public static void sleep(long millis, int nanos) throws InterruptedException;
```

此方法在指定的毫秒数加指定的纳秒数让当前正在执行的线程休眠（暂停执行）。

```java
public final void join() throws InterruptedException;
```

主线程等待子线程的终止，在子线程调用了`join()`，之后的代码只能等到子线程结束了才能执行。

```java
public static native void yield();
```

使当前线程从执行状态（运行状态）变为可执行态（就绪状态）。当一个线程使用了这个方法之后，它会把自己CPU执行的时间让掉，让自己或者其它线程运行。

```java
public void interrupt();
```

给线程设置中断标志，中断调用该方法的Thread实例所代表的线程。

```java
public boolean isInterrupted();
```

检测调用该方法的Thread实例所代表的线程是否中断。

```java
public static boolean interrupted();
```

检测中断并清除中断状态，作用于当前线程。

```java
public final void setPriority(int newPriority);
```

用于设置更改线程的优先级，每个线程都有一个优先级，由1到10之间的整数表示，Thread类提供3个常量属性：

```java
public static final int MIN_PRIORITY = 1;  // 最大优先级
public static final int NORM_PRIORITY = 5; // 普通优先级
public static final int MAX_PRIORITY = 10; // 最小优先级
```

以下为Object类中的方法：

```java
public final native void wait(long timeoutMillis) throws InterruptedException;
public final void wait() throws InterruptedException; // 调用以上方法
```

该方法须在synchronized块中调用。

_`wait()`与`sleep()`的区别：_

1. _`sleep()`方法正在执行的线程主动让出CPU（不会释放同步锁），在sleep指定时间后CPU再回到该线程继续往下执行；`wait()`则是指当前线程让自己暂时退让同步资源锁，以便等其他等待该资源的线程得到该资源进而运行，只有调用了`notify()`或`notifyAll()`方法才能解除之前调用`wait()`方法的线程的WAIT状态，可以去参与竞争同步资源锁进而得到运行；_
2. _`sleep()`可在任何地方使用，`wait()`只能在同步块或同步方法中使用；_
3. _`sleep()`是`Thread`类中的方法，`wait()`是`Object`类中的方法。_

```java
public final native void notify();
```

该方法须在synchronized块中调用。

```java
public final native void notifyAll();
```

该方法须在synchronized块中调用。

_`notify()`与`notifyAll()`的区别：_

> 等待池：假设一个线程调用了某个对象的`wait()`方法，该线程会释放对象的锁，进入那个对象的等待池，等待池中的线程不会去竞争该对象的锁。
>
> 锁池：只有获得了对象的锁，线程才能执行对象的synchronized代码，对象的锁每次只能有一个线程可以获得，其他线程只能在锁池等待。

_`notify()`随机唤醒对象的等待池中的一个线程，进入锁池；`notifyAll()`唤醒对象的等待池中的所有线程，进入锁池。_

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

### 锁升级过程

锁的状态会随着竞争激烈逐渐升级（无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁），通常情况下锁的状态只能升级不能降级。这种策略是为了提高获得锁和释放锁的效率。

> 对象头结构
>
> |长度|内容|说明|
> |----|----|----|
> |32/64bit|Mark Word|hashCode GC分代年龄，锁信息|
> |32/64bit|Class Pointer|指向对象类型的指针|
> |32/64bit|Array Length|数组长度（当对象为数组时）|

由此可以看出，锁信息存在Mark Word里。当创建一个对象时，该对象的Mark Word关键数据如下：

|bit fields|是否偏向锁|锁标志位|
|---|---|---|
|hash|0|01|

偏向锁状态为01表示该对象尚未被加上偏向锁（1表示被加上偏向锁）。

**偏向锁**

偏向锁会偏向于第一个获得它的线程，在接下来的执行过程中，假如该锁未被其他线程获取，没有其他线程来竞争该锁，那么持有偏向锁的线程将永远不需要进行同步操作。

偏向锁获取流程：检查对象头中Mark Word是否为可偏向状态，若不是则直接升级为轻量级锁；若是，判断Mark Word中的线程ID是否指向当前线程，若是则执行同步代码块，若不是则进行CAS操作竞争锁，如果竞争到锁则将Mark Word中的线程ID设定为当前线程ID，执行同步代码块；如果竞争失败则升级为轻量级锁。

```
+----------------------------------+  Y   +-------------------------------------+
| 检查对象头中Mark Word是否为可偏向状态 | ---> | 判断Mark Word中的线程ID是否指向当前线程 |
+----------------------------------+      +-------------------------------------+
                 |                              |                      |
                 | N                            | N                    | Y
                 v                              v                      v
         +---------------+      N      +----------------+   Y   +--------------+
         |  升级为轻量级锁  | <--------- | 进行CAS操作竞争锁 | ----> | 执行同步代码块 |
         +---------------+             +----------------+       +--------------+
```

- 锁膨胀：当有两个以上的线程竞争锁，则偏向锁失效，此时锁膨胀为轻量级锁
- 锁撤销：撤销失效的偏向锁，只有等到竞争，持有偏向锁的线程才会撤销偏向锁，偏向锁撤销后会恢复到无锁或轻量级锁的状态

	1. 偏向锁的撤销需要到达全局安全点，全局安全点表示一种状态，该状态下所有线程都处于暂停状态；
	2. 判断锁对象是否处于无锁状态，即获得偏向锁的线程如果已经退出了临界区，表示同步代码已执行完了，重新竞争锁的线程会进行CAS操作替代原来的线程ID；
	3. 如果获得偏向锁的线程还处于临界区之内，表示同步代码未执行完，将获得偏向锁的线程升级为轻量级锁。

**轻量级锁**

在多线程交替执行同步代码块时（未发生竞争），减少传统重量级锁使用操作系统互斥量产生的性能消耗，在使用轻量级锁时不需要申请互斥量，加解锁使用CAS操作。

_如果存在锁竞争，除了互斥量开销外，还有额外的CAS操作，轻量级锁将比传统重量级锁更慢，锁竞争激烈时将膨胀为重量级锁。_

偏向锁升级为轻量级锁后的Mark Word部分数据如下：

|bit fields|锁标志位|
|---|---|
|指向LockRecord的指针|00|

轻量级锁获取流程：

1. 首先判断当前对象是否处于一个无锁的状态，若是则Java虚拟机将在当前线程的栈帧建立一个锁记录（Lock Record）用于存储对象目前的Mark Word拷贝；
2. 将对象的Mark Word复制到栈帧中的Lock Record中并将Lock Record中的owner指向当前对象，并使用CAS操作将对象的Mark Word更新为指向Lock Record的指针；
3. 若第2步执行成功，表示该线程获得了这个对象的锁，将对象Mark Word中的锁标志位设为00，执行同步代码块；
4. 若第2步未执行成功，需要先判断当前对象的Mark Word是否指向当前线程的栈帧，若是表示当前线程已经持有了当前对象的锁，这是一次重入，直接执行同步代码块；若不是则表示多个线程存在竞争，该线程通过自旋尝试获得锁，即重复步骤2，自旋超过一定次数，轻量级锁升级为重量级锁。

轻量级锁的解锁：线程会通过CAS操作将Lock Record中的Mark Word（官方称为Displaced Mark Word）替换回来，若成功表示没有竞争发生，成功释放锁，恢复至无锁状态；如果失败则表示当前存在竞争，升级为重量级锁。

**自旋锁**

1. 自旋锁

	自旋锁指当一个线程在获取锁的时候，如果锁已经被其他线程获取，那么该线程将循环等待，然后不断判断锁是否能被成功获取，直到获取到锁后才退出循环。获取锁的线程一直处于活跃状态，但是并没有执行有效的任务，使用这种锁会造成busy-waiting。

2. 自适应自旋锁

	JDK1.6引入自适应自旋锁，自适应自旋锁的自旋次数不再固定，而是由上一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的，如果对于某个锁对象，刚刚有线程自旋等待成功获取到锁，那么虚拟机将认为这次自旋等待的成功率也很高，会允许线程自旋等待的时间更长一些。如果对于某个锁对象，线程自旋等待很少成功获取到锁，那么虚拟机将会减少线程自旋等待的时间。

**重量级锁**

轻量级锁膨胀后升级为重量级锁。重量级锁依赖对象内部的monitor锁实现，monitor锁又依赖于操作系统的MutexLock实现，所以重量级锁又被称为互斥锁。（JDK1.6以前的synchronized为重量级锁）

当轻量级锁经过锁撤销等步骤升级为重量级锁之后，它的Mark Word部分数据大体如下：

|bit fields|锁标志位|
|---|---|
|指向Mutex的指针|10|

_重量级锁开销大的原因：_

_监视器锁依赖于底层操作系统的`MutexLock`实现，Java的线程是映射到操作系统的原生线程上的，若要挂起或唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态切换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本高。_

## 悲观锁

总是假设最坏的情况，每次拿数据的时候都认为别人会修改，所以悲观锁在持有资源或数据的时候总会把资源或数据锁住，这样其他线程想要请求这个资源的时候就会阻塞，直到等到悲观锁把资源释放为止。`synchronized`和`ReentrantLock`就是悲观锁思想的实现，不管是否持有资源都会尝试加锁。

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

	无锁状态下实现多线程之间的变量同步，亦即在没有线程被阻塞时实现多线程之间的变量同步。

	涉及三个操作数：

	1. 需要读写的内存值V
	2. 进行比较的值A
	3. 拟写入的新值B

	当且仅当V的值为A时，使用原子操作用新值B来更新V的值，否则不会进行任何操作；一般情况下会进行自旋，即不断地重试。

	**CAS缺点**

	1. ABA问题

	2. 循环时间开销过大

	3. 只能保证一个共享变量的原子操作

## Atomic类

JUC下的原子类可以分为4类：

1. 基本类型

	- AtomicInteger：整型原子类
	- AtomicLong：长整型原子类
	- AtomicBoolean：布尔型原子类

2. 数组类型

	- AtomicIntegerArray：整型数组原子类
	- AtomicLongArray：长整型数组原子类
	- AtomicReferenceArray：引用类型数组原子类

3. 引用类型

	- AtomicReference：引用类型原子类
	- AtomicStampedReference：原子更新引用类型里的字段原子类
	- AtomicMarkableReference：原子更新带有标记位的引用类型

4. 对象的属性修改类型

	- AtomicIntegerFieldUpdater：原子更新整型字段的更新器
	- AtomicLongFieldUpdater：原子更新长整型字段的更新器
	- AtomicStampedReference：原子更新带有版本号的引用类型

**Atomic原理**

多线程环境下，当有多个线程同时对单个变量进行操作时，具有排他性，即当多个线程同时对该变量的值进行更新时，仅有一个线程能成功，而未成功的线程可以像自旋锁一样继续尝试，直到执行成功。

## AQS(AbstractQueuedSynchronizer，抽象队列同步器)

AQS核心思想是，如果被请求的共享资源空闲，则将请求资源的线程设置为有效的工作线程，并将共享资源设定为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配机制，这个机制AQS使用CLH队列锁实现，即将暂时获取不到锁的线程加入到队列中。

> **CLH队列**
>
> CLH队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

AQS使用一个int成员变量来表示同步状态（加锁状态），通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state; // 共享变量，使用volatile保证线程可见性
```

状态信息通过`protected`类型的`getState`，`setState`，`compareAndSetState`进行操作。初始状态下，该值为0；加锁时，该值通过CAS操作加1，解锁时减1。

```java
private transient Thread exclusiveOwnerThread;
```

该变量用于记录当前加锁的是哪个线程，初始状态为null。加锁时将该变量赋值为当前加锁的线程。

#### AQS对资源的共享方式

- Exclusive（独占）

	只有一个线程能执行，如`ReentrantLock`。又分公平锁与非公平锁：

	- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
	- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- Share（共享）：多个线程同时执行，如`CountDownLatch`、`Semaphore`、`ReadWriteLock`、`CyclicBarrier`。

`ReentrantReadWriteLock`可以看作是组合式，因为`ReentrantReadWriteLock`也就是读写锁允许多个线程同时对某一资源进行读。

### Lock

一个接口，实例化时通常使用`ReentrantLock`类：

```java
Lock lock = new ReentrantLock();
```

针对需要同步处理的代码设置对象监视器，比整个方法用synchronized修饰要好。通过Lock对象，用`lock.lock()`加锁，用`lock.unlock()`解锁，在二者之中放置需要同步处理的代码。

使用Lock对象加锁时也是一个对象锁，持有对象监视器的线程才能同步执行代码，其他线程只能等待该线程释放对象监视器。

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

由Lock对象所创建：

```java
Lock lock = new ReetrantLock();
Condition condition = lock.newCondition();
```

同步调度方法：

```java
void await() throws InterruptedException;
```

用以实现让线程等待，让线程进入阻塞，作用同`wait()`，需要在同步代码区使用。

```java
void signal();
```

唤醒线程，作用同`notify()`，需要在同步代码区使用。

```java
void signalAll();
```

作用同`notifyAll()`，需要在同步代码区使用。

### ReentrantLock（可重入锁）

可重入锁：如果当前线程已经获得执行序列中的锁，那么执行序列之后的所有方法都可以获得这个锁。

ReentrantLock基于AQS实现。加锁时`state`变量加1，并把`exclusiveOwnerThread`设置为加锁线程；解锁时`state`变量减1。ReentrantLock的可重入性基于`Thread.currentThread()`实现，如果当前线程已经获得锁，那么该线程下的所有方法都可以获得锁。可重入加锁时会先判断当前加锁的线程，若当前加锁的线程是自己则会对`state`变量进行累加1。

若`state`不为0，当前线程欲争得锁，此时会先判断`exclusiveOwnerThread`的值，若非当前线程则该线程会进入等待队列等待加锁的线程释放锁。进入等待队列的线程会通过调用`LockSupport.park()`被挂起，而唤醒时则是通过调用`LockSupport.unpark()`。

ReentrantLock有两个内部类：`FairSync`（公平锁）和`NonFairSync`（非公平锁）。默认采用非公平锁，可在构造方法中传入`true`采用公平锁。

非公平锁在调用`lock()`后，首先就会调用CAS进行一次抢锁，如果此时恰巧锁未被占用，直接获取锁并返回。CAS失败后会进入`tryAcquire()`方法，在`tryAcquire()`方法中如果发现锁这个时候被释放了，非公平锁会直接CAS抢锁，若CAS失败就进入等待队列；公平锁会判断等待队列是否有线程处于等待状态，如果有则不会抢锁而是进入等待队列。

相对来说非公平锁会有更好的性能，因为它的吞吐量较大，但非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态。

***TODO***

### ReadWriteLock（读写锁）

读写锁分为两个锁，读锁和写锁。读锁与读锁之间是共享的，写锁与读锁之间是互斥的，写锁与写锁之间亦是互斥的。

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

线程局部变量，同一个ThreadLocal所包含的对象，在不同的线程中有不同的副本。

ThreadLocal类能够实现每一个线程都有自己的专属本地变量，让每个线程绑定自己的值。创建一个ThreadLocal变量，那么访问该变量的每个线程都有这个变量的本地副本，并可以使用`get()`和`set()`来获取默认值或者将其值修改为当前线程所存副本之值，从而避免线程安全问题。

### ThreadLocal原理

`ThreadLocal`是一个泛型类，保证可以接受任何类型的对象。由于一个线程内可以存在多个`ThreadLocal`对象，`ThreadLocal`内部维护了一个`ThreadLocalMap`的静态类。ThreadLocal的`set()`/`get()`方法调用了`ThreadLocalMap`对应的`set()`/`get()`方法。即`ThreadLocal`其实为`ThreadLocalMap`的封装，传递了变量值。

**为何将key设置为弱引用？**

***TODO***

### ThreadLocal内存泄露问题

`ThreadLocalMap`中使用的key是弱引用，而value是强引用，所以，如果ThreadLocal没有被外部强引用的情况下，在垃圾回收的时候，key会被清理掉而value没有被清理。这样一来ThreadLocalMap就会出现key为`null`的Entry，如果不做任何措施则value永远不会被GC，这个时候就容易产生内存泄露。

`ThreadLocalMap`实现中已经考虑这种情况，在调用`set()`，`get()`，`remove()`时会清理掉key为`null`的记录。使用完ThreadLocal方法后最好手动调用`remove()`方法。

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

***TODO***
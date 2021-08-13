# 进程&线程

## synchronized

### 实现原理

#### 偏向锁

#### 轻量级锁

#### 重量级锁

### wait() & notify() & notifyAll()


## volitile

## 线程池

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

## CAS(Compare And Swap，比较并替换)



### Atomic类



## AQS(AbstractQueueSynchronizer，抽象队列同步器)

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

### Condition

### ReentrantLock

### Semephore

## 公平锁&非公平锁
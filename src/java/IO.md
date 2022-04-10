# I/O

Java I/O分类：

- 按功能来分：输入流、输出流
- 按类型来分：
    - 字节流：`InputStream`/`OutputStream`是字节流的抽象类，二者又派生出若干子类，不同子类分别处理不同的操作类型，按8位传输，以字节为单位输入输出数据
    - 字符流：`Reader`/`Writer`是字符流的抽象类，二者亦派生出若干子类，不同子类分别处理不同的操作类型，按16位传输，以字符为单位输入输出数据

    _注：无论是文件读写还是网络发送接收，信息的最小存储单元都是字节。_

**IO操作为何需要手动关闭？**

使用完IO流需要手动回收，这是为了节约系统资源。一般来说，需要手动关闭的都是用了虚拟机之外的资源，如端口、文件等，虚拟机无法通过垃圾回收释放这些资源，只能显式调用关闭方法来释放。调用`finalize()`方法虽然可以释放非Java资源，但是该方法的执行时机是在GC之前，而GC具有时间不确定性，所以`finalize()`执行时间亦不具确定性，对于需要及时回收的资源此方法无法保证及时，另外`finalize()`不是析构函数，JVM不能保证`finalize()`一定会执行，不能依赖`finalize()`来释放资源。

许多情况下，如果在一些比较频繁的操作中，不对流进行关闭，很容易出现输入输出流已经超越了JVM边界，所以有时候可能无法回收资源，所以流操作的时候凡是跨出虚拟机边界的资源都要求程序员自己关闭，不要指望垃圾回收。

## BIO

阻塞IO（Blocking IO）：数据的读取和写入须阻塞在一个线程内等待其完成。由于同步阻塞，因此新请求来时只能通过新建线程的方式来接受请求，导致系统占用资源大，或是线程堆栈溢出。可以通过线程池来优化，不过在并发量增加时会导致线程数量急剧膨胀。

## NIO

非阻塞IO（Non-blocking IO）：BIO面向流，而NIO面向缓冲区，任何时候访问NIO的数据，都是面向缓冲区的，最常用的缓冲区是`ByteBuffer`。

- Channel（通道）：NIO使用通道进行读写，通道是双向的，可读可写
- Selector（选择器）：用于使用单线程处理多个通道

## IO多路复用

一种同步IO模型，实现一个线程可以监听多个文件句柄，一旦某个文件句柄就绪，就能够通知应用程序进行相应的读写操作，没有文件句柄就绪就会阻塞应用程序，交出CPU。

_注：多路是指网络连接，复用是指同一个线程。_

### Select/Poll

```c
// select
int select(int max_fd, fd_set *readset, fd_set *writeset, fd_set *exceptset, struct timeval *timeout);
typedef struct {
  unsigned long fds_bits[__FDSET_LONGS];
} fd_set;
FD_ZERO(int fd, fd_set *fds);   // 清空集合
FD_SET(int fd, fd_set *fds);    // 将给定的描述符加入集合
FD_ISSET(int fd, fd_set *fds);  // 判断指定描述符是否在集合中
FD_CLR(int fd, fd_set *fds);    // 将给定的描述符从文件中删除
// poll
int poll(struct pollfd fds[], nfds_t nfds, int timeout);
struct pollfd {
  int fd;
  short events;
  short revents;
};
```

`select()`底层是一个`fd_set`的数据结构，本质上是一个long类型数组，数组中的每一个元素都对应于一个文件描述符，通过轮询所有的文件描述符来检查是否有事件发生。`poll()`与`select()`差不多，但`poll()`的文件描述符无最大数量限制，但是依旧采用轮询遍历的方式检查是否有事件发生。

优点：

1. 可移植性好
2. 连接数少且连接都十分活跃的情况下效率不错

缺点：

1. 单个进程所打开的fd是有限的，通过`FD_SETSIZE`处理，默认1024（`select()`有此限制，`poll()`无最大连接数限制）
2. 每次调用`select()`，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
3. 对socket扫描时是线性扫描，采用轮询的方法，效率较低（高并发）

### Epoll

```c
typedef union epoll_data {
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;
struct epoll_event {
  uint32_t events;
  epoll_data_t data;
};
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *event, int maxevents, int timeout);
```

Epoll是一种更高效的IO多路复用的方式，它可以监视的文件描述符数量突破了1024的限制，同时不需要通过轮询遍历的方式去检查文件描述符上是否有事件发生，因为`epoll_wait()`返回的就是有事件发生的文件描述符。Epoll本质上是事件驱动。

Epoll具体是通过红黑树和就绪链表实现的，红黑树存储所有的文件描述符，就绪链表存储有事件发生的文件描述符；`epoll_ctl()`可以对文件描述符结点进行增删改查，并告知内核注册回调函数（事件），一旦文件描述符上有事件发生时，内核将该文件描述符结点插入到就绪链表里面，这时`epoll_wait()`将会接收到消息，并且将数据拷贝到用户空间。

表面上看epoll的性能最好，但是在连接数少并且连接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。

#### 水平触发&边缘触发

- 水平触发（LT）：一个事件只要有，就会一直触发

    - Socket接收缓冲区不为空，有数据可读，读事件一直触发
    - Socket发送缓冲区不满，可以继续写入数据，写事件一直触发

    处理过程：

    - Accept一个连接，添加到epoll中监听`EPOLLIN`事件
    - 当`EPOLLIN`事件到达时，读`fd`中的数据并处理
    - 当需要写出数据时，把数据写到`fd`中，若数据较大，无法一次性写出，则在epoll中监听`EPOLLOUT`事件
    - 当`EPOLLOUT`事件到达时，继续把数据写到`fd`中，若数据写出完毕，则在epoll中关闭`EPOLLOUT`事件

    LT的处理过程中，直到返回`EAGAIN`不是硬性要求，但通常的处理过程都会读写直到返回`EAGAIN`，但LT比ET多了个开关`EPOLLOUT`事件的步骤。LT的编程与`poll()`/`select()`接近，符合一直以来的习惯，不易出错。
- 边缘触发（ET）：只有一个事件从无到有才会触发

    - Socket的接收缓冲区状态变化时触发读事件，即空的接收缓冲区刚接收到数据时触发读事件
    - Socket的发送缓冲区状态变化时触发写事件，即满的缓冲区刚空出空间时触发写事件

    处理过程：

    - Accept一个连接，添加到epoll中监听`EPOLLIN|EPOLLOUT`事件
    - 当`EPOLLIN`事件到达时，读`fd`中的数据并处理，`read()`需一直读，直到返回`EAGAIN`为止
    - 当需要写出数据时，把数据写到`fd`中，直到数据全部写完，或者`write()`返回`EAGAIN`
    - 当`EPOLLOUT`事件到达时，继续把数据写到`fd`中，直到数据全部写完，或者`write()`返回`EAGAIN`

    _仅在状态变化时触发。_

    ET的要求是一直读写，直到返回`EAGAIN`，否则就会遗漏事件。ET的编程可以做到更加简洁，某些场景下更加高效，但一方面容易遗漏事件，产生错误（如`EPOLLIN`时要循环读至`EAGAIN`，如果在读的`fd`一直有数据到来，会造成其他描述符饥饿）。

### Reactor模型

Reactor模型要求主线程只负责监听文件描述上是否有事件发生，有的话立即将该事件通知给工作线程，除此之外，主线程不做任何其他实质性的工作，读写数据、接收新的连接以及处理客户请求均在工作线程上完成。

Reactor模型主要包含两个组件：

- Reactor：负责查询、响应IO事件，当检测到IO事件时，分发给Handlers处理；
- Handler：与IO事件绑定，负责IO事件的处理。

包含几种实现方式：

1. 单线程单Reactor：该模式Reactor和Handler在同一线程中，若某个Handler阻塞会导致其他Handler无法执行，且无法充分利用多核的性能
2. 多线程单Reactor：由于decode、compute、encode操作并非IO操作，多线程单Reactor思路就是充分发挥多核的特性，同时把非IO的操作剥离开，但由于单个Reactor承担了所有的事件监听、响应工作，若连接过多还是有可能存在性能问题
3. 多线程多Reactor：为了解决单Reactor的性能问题产生的多Reactor模式，其中mainReactor建立连接，多个subReactor负责数据读写

## AIO

异步：当一个异步过程被调用后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。与同步相对。

异步IO（Asynchronized IO）：基于事件和回调机制实现

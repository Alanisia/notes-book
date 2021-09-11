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
## BIO

阻塞IO（Blocking IO）：数据的读取和写入须阻塞在一个线程内等待其完成。由于同步阻塞，因此新请求来时只能通过新建线程的方式来接受请求，导致系统占用资源大，或是线程堆栈溢出。可以通过线程池来优化，不过在并发量增加时会导致线程数量急剧膨胀。

## NIO

非阻塞IO（Non-blocking IO）：BIO面向流，而NIO面向缓冲区，任何时候访问NIO的数据，都是面向缓冲区的，最常用的缓冲区是`ByteBuffer`。

- Channel（通道）：NIO使用通道进行读写，通道是双向的，可读可写
- Selector（选择器）：用于使用单线程处理多个通道

## IO多路复用

***TODO***

### Select/Poll

***TODO***

### Epoll

***TODO***

### Reactor模型

Reactor模型要求主线程只负责监听文件描述上是否有事件发生，有的话立即将该事件通知给工作线程，除此之外，主线程不做任何其他实质性的工作，读写数据、接收新的连接以及处理客户请求均在工作线程上完成。

Reactor模型主要包含两个组件：

- Reactor：负责查询、响应IO事件，当检测到IO事件时，分发给Handlers处理；
- Handler：与IO事件绑定，负责IO事件的处理。

包含几种实现方式：

1. 单线程单Reactor：该模式Reactor和Handler在同一线程中，若某个Handler阻塞会其他导致其他Handler无法执行，且无法充分利用多核的性能
2. 多线程单Reactor：由于decode、compute、encode操作并非IO操作，多线程单Reactor思路就是充分发挥多核的特性，同时把非IO的操作剥离开，但由于单个Reactor承担了所有的事件监听、响应工作，若连接过多还是有可能存在性能问题
3. 多线程多Reactor：为了解决单Reactor的性能问题产生的多Reactor模式，去中mainReactor建立连接，多个subReactor负责数据读写

## AIO

异步：当一个异步过程被调用后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。与同步相对。

异步IO（Asynchronized IO）：基于事件和回调机制实现
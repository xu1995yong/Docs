## Linux中的I/O模型

### 同步IO

两个阻塞过程

- 用户进程执行recv()/recvfrom()系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程中用户进程会被阻塞。

- 第二个阶段：当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，这个过程中用户进程同样要阻塞。

#### 阻塞IO（blokingIO）

在阻塞IO中，用户空间的进程执行系统调用recvfrom，然后进程在该处被阻塞，无法继续向下执行。直到数据准备好并且从内核复制到用户进程的缓冲区，此时recvfrom调用返回，进程才能继续向下执行。

![è¾å¥å¾çè¯´æ](https://static.oschina.net/uploads/img/201604/20150405_VKYH.png)

#### 非阻塞IO（non-blockingIO）

在这个模型中，用户进程执行recvfrom调用，如果内核中的数据没有准备好，此时recvfrom调用立刻返回一个错误。所以说用户进程在此处不需要等待。recvfrom返回之后，用户进程可以干点别的事情，然后再发起recvfrom系统调用，重复上面的过程，直到recvform返回数据已经准备好，再拷贝数据到用户进程。这个过程通常被称之为轮询。

需要注意，拷贝数据整个过程，进程仍然是属于阻塞的状态。

![è¾å¥å¾çè¯´æ](https://static.oschina.net/uploads/img/201604/20152818_DXcj.png)

#### 多路复用IO（multiplexingIO）

IO多路复用模型有两个特别的系统调用select/poll/epoll。

在这个模型中，由select/poll/epoll系统调用提供轮询来代替用户进程的主动轮询，select/poll/epoll系统调用的优势在于它可以同时处理多个连接。这样用户进程在调用select/poll/epoll时阻塞，当其中任意一个连接进入可读的就绪状态，select/poll/epoll就可以立即返回。然后进程再在执行recvform调用时阻塞，将数据由内核拷贝到用户进程。

将多个IO的阻塞复用到同一个select的阻塞上，使得系统在单线程的情况下同时处理更多的客户端请求，而不需要再创建多个线程，大大减少了系统开销。

![è¾å¥å¾çè¯´æ](https://static.oschina.net/uploads/img/201604/20164149_LD8E.png)

##### SELECT的特点

1. 单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024。
2. 对socket进行扫描时是线性扫描，即采用轮询的方法，随着FD的增加效率下降。
3. 需要维护一个用来存放大量文件描述符的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。
4. 消息需要由内核空间复制到用户空间。

##### POLL的特点

1. 基于链表来存储数据，对能够监视的文件描述符的数量没有限制。
2. 其他同SELECT。
3. 需要维护一个用来存放大量文件描述符的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。

##### EPOLL的特点

1. 对能够监视的文件描述符的数量没有限制。
2. 不采用轮询的方式，只有活跃可用的FD才会调用callback函数，所以不会随着FD数目的增加效率下降。
3. 通过内核空间和用户空间共享内存，消息不需要再进行复制。

### 异步IO（asynchronousIO）

在异步IO中，用户进程通过aio_read()函数进行读取操作后，可以立刻返回到本进程中继续执行其他的操作。kernel收到asynchronousread操作后，会等待数据准备完成，然后将数据拷贝到用户进程的缓冲区中。当这一切都完成之后，kernel会给用户发送一个signal通知用户进程read操作完成。

![è¾å¥å¾çè¯´æ](https://static.oschina.net/uploads/img/201604/20175459_gtgw.png)

## JAVA中的IO模型

### BIO

在JDK1.4之前，同步阻塞式的IO。没有数据缓冲区的概念。只有流的概念。

### NIO

JDK1.4新增了java.nio包，提供了进行异步IO的api和类库。

1. 缓冲区：实质是一个数组，提供了对存储数据的结构化访问和读写位置信息维护的功能。

![âjava nio ç¼å²åºâçå¾çæç´¢ç"æ](https://rmt-files.saowen.com/rmt_tk/img1/NvEVNzu.png)

2. 通道：网络数据通过channel读取和写入。通道与流的不同在于通道是双向的，支持阻塞和非阻塞两种模式。Channel主要分为网络读写的SelectableChannel和用于文件操作的FileChannel，SocketChannel和ServerSocketChannel都是SelectableChannel的子类。

3. 多路复用器：多路复用器提供了选择已经就绪的任务的能力。Selector会轮询注册在其上的channel，如果某个channel发生读写事件，这个channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey获取已经就绪的channel集合，进行后续的I/O操作。

   一个多路复用器Selector可以同时轮询多个Channel，一个线程负责Selector的轮询，就可以接入成千上万个客户端。

### AIO

JDK1.7将原来的NIO类库进行了升级，实现了真正意义上的异步非阻塞I/O。它不需要通过多路复用器Selector对注册的通道进行轮训操作即可实现异步读写，从而简化了NIO的编程模型。

主要在java.nio.channels包下增加了下面四个异步通道：

- AsynchronousSocketChannel
- AsynchronousServerSocketChannel
- AsynchronousFileChannel
- AsynchronousDatagramChannel



## BIO、NIO、AIO适用场景分析

- BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中。

- NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中。

- AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作。


## IO设计模式：Reactor和Proactor

- Reactor模式是基于NIO中实现多路复用的一种模式。Reactor实现了一个被动的事件分离和分发模型，服务等待请求事件的到来，再通过不受间断的同步处理事件，从而做出反应。

![ç¸å³å¾ç](http://i2.51cto.com/images/blog/201810/22/558b2e08a35ab4b7f45711389f227550.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- Proactor实现了一个主动的事件分离和分发模型；这种设计允许多个任务并发的执行，从而提高吞吐量；并可执行耗时长的任务


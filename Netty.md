# Netty知识点总结

## Netty介绍

Netty是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。

不选择JAVA原生NIO和IO的原因
基于IO的经典同步堵塞模型：
经典的IO模型也就是传统的服务器端同步阻塞I/O处理（也就是BIO，Blocking I/O）的经典编程模型，当我们每得到一个新的连接时，就会开启一个线程来处理这个连接的任务。之所以使用多线程，主要原因在于socket.accept()、socket.read()、socket.write()三个主要函数都是同步阻塞的，当一个连接在处理I/O的时候，系统是阻塞的，如果是单线程的话必然就挂死在那里；但CPU是被释放出来的，开启多线程，就可以让CPU去处理更多的事情。

因此这个模型最本质的问题在于，严重依赖于线程。但线程是很”贵”的资源，主要表现在：

线程的创建和销毁成本很高，在Linux这样的操作系统中，线程本质上就是一个进程。创建和销毁都是重量级的系统函数。 
线程本身占用较大内存，像Java的线程栈，一般至少分配512K～1M的空间，如果系统中的线程数过千，恐怕整个JVM的内存都会被吃掉一半。

线程的切换成本是很高的。操作系统发生线程切换的时候，需要保留线程的上下文，然后执行系统调用。如果线程数过高，可能执行线程切换的时间甚至会大于线程执行的时间，这时候带来的表现往往是系统load偏高、CPU sy使用率特别高（超过20%以上)，导致系统几乎陷入不可用的状态。

容易造成锯齿状的系统负载。因为系统负载是用活动线程数或CPU核心数，一旦线程数量高但外部网络环境不是很稳定，就很容易造成大量请求的结果同时返回，激活大量阻塞线程从而使系统负载压力过大。
参考：[Java NIO浅析]http://mp.weixin.qq.com/s/HhwaXd8x7zONr8N1ojSnsQ

基于NIO的异步模型：
NIO是一种同步非阻塞的I/O模型，也是I/O多路复用的基础，而且已经被越来越多地应用到大型应用服务器，成为解决高并发与大量连接、I/O处理问题的有效方式。[Java NIO介绍（二）————无堵塞io和Selector简单介绍]

不使用NIO的原因：

NIO的类库和API繁杂。需要很多额外的技能做铺垫。例如需要很熟悉Java多线程编程、Selector线程模型。导致工作量和开发难度都非常大。
扩展 ByteBuffer：NIO和Netty都有ByteBuffer来操作数据，但是NIO的ByteBuffer长度固定而且操作复杂，许多操作甚至都需要自己实现。而且它的构造函数是私有，不能扩展。Netty 提供了自己的 
ByteBuffer 实现， Netty 通过一些简单的 APIs 对 ByteBuffer 进行构造、使用和操作，以此来解决 NIO 中的一些限制。
NIO 对缓冲区的聚合和分散操作可能会操作内存泄露，到jdk7才解决了内存泄露的问题
存在臭名昭著的epoll bug，导致Selector空轮询：这个bug会导致linux上导致cpu 100%：http://www.blogjava.net/killme2008/archive/2009/09/28/296826.html
为什么选择Netty
API使用简单，开发门槛低。
功能强大，预置了多种编解码功能，支持多种协议开发。
定制能力强，可以通过ChannelHadler进行扩展。
性能高，对比其它NIO框架，Netty综合性能最优。
经历了大规模的应用验证。在互联网、大数据、网络游戏、企业应用、电信软件得到成功，很多著名的框架通信底层就用了Netty，比如Dubbo
稳定，修复了NIO出现的所有Bug。
切换IO和NIO，因为IO和NIO的API完全不同，相互切换非常困难。

## Netty中的组件介绍

- **Bootstrap**：netty的辅助启动器，netty客户端和服务器的入口，Bootstrap是创建客户端连接的启动器，ServerBootstrap是监听服务端端口的启动器。
- **Channel**：关联Jdk原生socket的组件，常用的是NioServerSocketChannel和NioSocketChannel，NioServerSocketChannel负责监听一个tcp端口，有连接进来通过boss reactor创建一个NioSocketChannel将其绑定到worker reactor，然后worker reactor负责这个NioSocketChannel的读写等io事件。
- **EventLoop**：netty最核心的几大组件之一，就是我们常说的reactor，人为划分为boss reactor和worker reactor。通过EventLoopGroup（Bootstrap启动时会设置EventLoopGroup）生成，最常用的是nio的NioEventLoop，就如同EventLoop的名字，EventLoop内部有一个无限循环，维护了一个selector，处理所有注册到selector上的io操作，在这里实现了一个线程维护多条连接的工作。
- **ChannelHandler**：netty最核心的几大组件之一，netty处理io事件真正的处理单元，开发者可以创建自己的ChannelHandler来处理自己的逻辑，完全控制事件的处理方式。ChannelHandler和ChannelPipeline组成责任链，使得一组ChannelHandler像一条链一样执行下去。ChannelHandler分为inBound和outBound，分别对应io的read和write的执行链。ChannelHandler用ChannelHandlerContext包裹着，有prev和next节点，可以获取前后ChannelHandler，read时从ChannelPipeline的head执行到tail，write时从tail执行到head，所以head既是read事件的起点也是write事件的终点，ChannelHandler与io交互最紧密。
- **ChannelPipeline**：netty最核心的几大组件之一，ChannelHandler的容器，netty处理io操作的通道，与ChannelHandler组成责任链。write、read、connect等所有的io操作都会通过这个ChannelPipeline，依次通过ChannelPipeline上面的ChannelHandler处理，这就是netty事件模型的核心。ChannelPipeline内部有两个节点，head和tail，分别对应着ChannelHandler链的头和尾。
- **Unsafe**：顾名思义这个类就是不安全的意思，但并不是说这个类本身不安全，而是不要在应用程序里面直接使用Unsafe以及他的衍生类对象，实际上Unsafe操作都是在reactor线程中被执行。Unsafe是Channel的内部类，并且是protected修饰的，所以在类的设计上已经保证了不被用户代码调用。Unsafe的操作都是和jdk底层相关。EventLoop轮询到read或accept事件时，会调用unsafe.read()，unsafe再调用ChannelPipeline去处理事件；当发生write事件时，所有写事件都会放在EventLoop的task中，然后从ChannelPipeline的tail传播到head，通过Unsafe写到网络中。



Nettty 有如下几个核心组件：

- Channel
- ChannelFuture
- EventLoop
- ChannelHandler
- ChannelPipeline





## Netty中的NioEventLoopGroup

无锁化的串行设计

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                        EventExecutorChooserFactory chooserFactory, Object... args) {
    if (nThreads <= 0) {
        throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
    }

    if (executor == null) {
        executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());//任务执行器，每
    }

    children = new EventExecutor[nThreads];

    for (int i = 0; i < nThreads; i ++) {
        boolean success = false;
        try {
            children[i] = newChild(executor, args);
            success = true;
        } catch (Exception e) {
            // TODO: Think about if this is a good exception type
            throw new IllegalStateException("failed to create a child event loop", e);
        } finally {
            if (!success) {
                for (int j = 0; j < i; j ++) {
                    children[j].shutdownGracefully();
                }

                for (int j = 0; j < i; j ++) {
                    EventExecutor e = children[j];
                    try {
                        while (!e.isTerminated()) {
                            e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                        }
                    } catch (InterruptedException interrupted) {
                        // Let the caller handle the interruption.
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
    }

    chooser = chooserFactory.newChooser(children);

    final FutureListener<Object> terminationListener = new FutureListener<Object>() {
        @Override
        public void operationComplete(Future<Object> future) throws Exception {
            if (terminatedChildren.incrementAndGet() == children.length) {
                terminationFuture.setSuccess(null);
            }
        }
    };

    for (EventExecutor e: children) {
        e.terminationFuture().addListener(terminationListener);
    }

    Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
    Collections.addAll(childrenSet, children);
    readonlyChildren = Collections.unmodifiableSet(childrenSet);
}
```



## NioEventLoop



```java
private static void doBind0(final ChannelFuture regFuture, final Channel channel,final SocketAddress 	localAddress, final ChannelPromise promise) {

    // This method is invoked before channelRegistered() is triggered.  Give user handlers a chance to set up
    // the pipeline in its channelRegistered() implementation.
    channel.eventLoop().execute(new Runnable() {
        @Override
        public void run() {
            if (regFuture.isSuccess()) {
                channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            } else {
                promise.setFailure(regFuture.cause());
            }
        }
    });
}

public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }

    boolean inEventLoop = inEventLoop();  //返回false
    addTask(task);
    if (!inEventLoop) {
        startThread();
        if (isShutdown() && removeTask(task)) {
            reject();
        }
    }
    if (!addTaskWakesUp && wakesUpForTask(task)) {
        wakeup(inEventLoop);
    }
}

private void startThread() {
    if (state == ST_NOT_STARTED) {
        if (STATE_UPDATER.compareAndSet(this, ST_NOT_STARTED, ST_STARTED)) {//CAS的方式判断
            try {
                doStartThread();
            } catch (Throwable cause) {
                STATE_UPDATER.set(this, ST_NOT_STARTED);
                PlatformDependent.throwException(cause);
            }
        }
    }
}
private void doStartThread() {
    assert thread == null;
    executor.execute(new Runnable() {
        @Override
        public void run() {
            thread = Thread.currentThread();
            if (interrupted) {
                thread.interrupt();
            }

            boolean success = false;
            updateLastExecutionTime();
            try {
                SingleThreadEventExecutor.this.run();
                success = true;
            } catch (Throwable t) {
                logger.warn("Unexpected exception from an event executor: ", t);
            } finally {
                for (;;) {
                    int oldState = state;
                    if (oldState >= ST_SHUTTING_DOWN || STATE_UPDATER.compareAndSet(
                        SingleThreadEventExecutor.this, oldState, ST_SHUTTING_DOWN)) {
                        break;
                    }
                }

                // Check if confirmShutdown() was called at the end of the loop.
                if (success && gracefulShutdownStartTime == 0) {
                    if (logger.isErrorEnabled()) {
                        logger.error("Buggy " + EventExecutor.class.getSimpleName() + " implementation; " +
                                     SingleThreadEventExecutor.class.getSimpleName() + ".confirmShutdown() must " +
                                     "be called before run() implementation terminates.");
                    }
                }

                try {
                    // Run all remaining tasks and shutdown hooks.
                    for (;;) {
                        if (confirmShutdown()) {
                            break;
                        }
                    }
                } finally {
                    try {
                        cleanup();
                    } finally {
                        STATE_UPDATER.set(SingleThreadEventExecutor.this, ST_TERMINATED);
                        threadLock.release();
                        if (!taskQueue.isEmpty()) {
                            if (logger.isWarnEnabled()) {
                                logger.warn("An event executor terminated with " +
                                            "non-empty task queue (" + taskQueue.size() + ')');
                            }
                        }

                        terminationFuture.setSuccess(null);
                    }
                }
            }
        }
    });
}

```











## Netty中的Bootstrap

### 1. ServerBootstrap

### 2. Bootstrap 



## Netty中的SocketChannel

### 1.  NioServerSocketChannel

### 2.  NioSocketChannel



## Netty中的ChannelConfig和ChannelOption



## Channel

Channel是通讯的载体

## ChannelPipeline



## ChannelHandler（接口）

处理I/O事件或拦截I/O操作，并将其转发到所在的ChannelPipeline中的下一个handler。

```java
//当ChannelHandler已被添加到上下文，并且已准备好处理事件后调用。
//该方法常用来初始化
void handlerAdded(ChannelHandlerContext ctx)throws Exception
//当ChannelHandler已被从上下文中删除，并且不再处理事件后调用。
void handlerRemoved(ChannelHandlerContext ctx)throws Exception
```

### 1. ChannelInboundHandler

处理入站数据以及各种状态变化。

```java
//当Channel已经注册到它的EventLoop并且能够处理I/O时被调用。
void channelRegistered(ChannelHandlerContext ctx) throws Exception
//当Channel从它的EventLoop注销并且无法处理任何io时被调用。
void channelUnregistered(ChannelHandlerContext ctx)throws Exception
//当Channel处于活动状态（已经连接到它的远程节点），并且可以接收和发送数据时被调用。
void channelActive(ChannelHandlerContext ctx) throws Exception
//当Channel离开活动状态时被调用。
void channelInactive(ChannelHandlerContext ctx) throws Exception
//当从Channel中读取数据时被调用。
void channelRead(ChannelHandlerContext ctx,Object msg)throws Exception
//当Channel上的一个读操作完成时被调用。
void channelReadComplete(ChannelHandlerContext ctx)throws Exception
//
void userEventTriggered(ChannelHandlerContext ctx,Object evt)throws Exception
//当Channel的可写状态发生改变时被调用。
void channelWritabilityChanged(ChannelHandlerContext ctx)throws Exception
//
void exceptionCaught(ChannelHandlerContext ctx,Throwable cause)throws Exception
```

### 2. ChannelOutboundHandler

处理出站数据并且允许拦截所有的操作。

```java

```



### 3. SimpleChannelInboundHandler



## 





## ChannelHandlerAdapter

### 1. ChannelHandlerAdapter

ChannelHandler的实现类。

```java
//如果实现是可共享的，则返回true，因此可以添加到不同的ChannelPipelines
public boolean isSharable()
```

### 2. ChannelInboundHandlerAdapter

ChannelInboundHandlerAdapter是ChannelInboundHandler的一个实现。 ChannelInboundHandler提供了可以覆盖的各种事件处理程序方法。

ChannelInboundHandlerAdapter只是将操作转发到ChannelPipeline中的下一个ChannelHandler。子类可以覆盖方法实现以更改此方法。

```java
//该方法在收到数据时调用
//注意该方法不会自动释放消息，所以需要使用手动释放： ReferenceCountUtil.release(msg);
public void channelRead(ChannelHandlerContext ctx, Object msg)
//该方法在建立连接并准备生成流量时调用
public void channelActive(final ChannelHandlerContext ctx) 
```

### 3. ChannelOutboundHandlerAdapter





## ChannelHandlerContext







```java
ChannelFuture write(Object msg);
ChannelFuture writeAndFlush(Object msg);
```





## Netty中的ChannelFuture

ChannelFuture表示异步通道I/O操作的结果。因为在Netty中所有操作都是异步的，这意味着任何I/O调用都将立即返回，并且不保证在调用结束时所请求的I/O操作已完成。所以需要ChannelFuture实例提供有关I/O操作的结果或状态的信息。还可以向ChannelFuture添加ChannelFutureListeners，以便在I/O操作完成时收到通知。



如下：

```java
final ChannelFuture f = ctx.writeAndFlush(time); // (3)
f.addListener(new ChannelFutureListener() {

    public void operationComplete(ChannelFuture future) {
        assert f == future;
        ctx.close();
    }
}); 
```



## Netty中的FutureListener

### 1. GenericFutureListener

GenericFutureListener用于监听Future的结果。

```java
//与Future关联的操作完成时调用。
void operationComplete(F future)
```



### 2. ChannelFutureListener

ChannelFutureListener用于监听ChannelFuture的结果。并将ChannelFuture快速的返回给调用者。

```java
//一个用于关闭与指定的ChannelFuture关联的Channel的ChannelFutureListener。
static final ChannelFutureListener CLOSE 
static final ChannelFutureListener CLOSE_ON_FAILURE
static final ChannelFutureListener FIRE_EXCEPTION_ON_FAILURE
```





## Netty中的编解码器

### 1. ByteToMessageDecoder





# ByteBuf

ByteBuf中类的继承结构

```java
ByteBuf
	AbstractByteBuf
		AbstractReferenceCountedByteBuf
			UnpooledDirectByteBuf：底层使用jdk的ByteBuffer的DirectByteBuffer实现
			UnpooledHeapByteBuf
			UnpooledUnsafeDirectByteBuf
			PooledByteBuf<T>
				PooledDirectByteBuf
				PooledHeapByteBuf
				PooledUnsafeDirectByteBuf
```

分类方式：

 	1. Pooled、Unpooled
 	2. Heap、Direct
 	3. Unsafe、非Unsafe




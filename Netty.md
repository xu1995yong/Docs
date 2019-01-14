# Netty知识点总结

## Netty的特点

Netty是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。



## Netty中的NioEventLoopGroup



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




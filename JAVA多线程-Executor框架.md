# Executor框架

## 1. 为什么使用线程池

1. 单线程串行执行的缺点：造成线程阻塞、cpu利用率低、

2. 无限制的创建线程的缺点：线程的生命周期开销高、资源消耗大、

## 2. Executor框架介绍

JDK1.5后引入的Executor框架将任务的提交和执行进行解耦，调用者只需要描述Task，然后提交给Executor即可。

简单示例如下：

```java
ExecutorService executor = Executors.newFixedThreadPool(7);
Callable task = new Callable() {
	@Override
    public Object call() throws Exception {
        System.out.println("Hello");
        return null;
    }
};
Future future = executor.submit(task);
System.out.println(future.get());
```

## 3. Executor框架的结构

Executor框架的类图如下所示：

![Executor框架类图](https://upload-images.jianshu.io/upload_images/2192701-8f7e90ad3ae4f116.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/718/format/webp)

Executor框架主要有3大部分组成：

- 任务：Executor框架中用Runnable接口或Callable接口表示。Runnable接口不会返回结果，而Callable接口可以返回结果。
- 任务的执行者：
  - 核心接口：Executor接口是Executor框架的基础。声明了方法`void execute(Runnable command)`实现了任务的提交和执行的解耦。只能执行Runnable任务。
  - ExecutorService接口：ExecutorService继承自Executor接口，并提供了线程的生命周期/管理方法。
  - ExecutorService接口的实现类：
    - ThreadPoolExecutor：是线程池的核心实现类，用来执行被提交的任务。通常使用工厂类Executors创建。
    - ScheduledThreadPoolExecutor：可以在给定的延迟后运行任务，或定期执行任务。通常使用工厂类Executors创建。

- 任务的结果：Executor框架中用Future接口和实现Future接口的FutureTask类。

## 4.  ExecutorService详解

```java
//平缓的关闭线程池。线程池停止接受新的任务，同时等待已经提交的任务执行完毕，包括那些进入队列还没有开始的任务。shutdown()方法执行过程中，线程池处于SHUTDOWN状态。
void shutdown()
//立即关闭线程池并返回从未开始执行的任务的列表。线程池停止接受新的任务，同时线程池取消所有执行的任务和已经进入队列但是还没有执行的任务。shutdownNow()方法执行过程中，线程池处于STOP状态。shutdownNow方法本质是调用Thread.interrupt()方法。所以线程中必须要有处理interrupt事件的机制。
List<Runnable> shutdownNow()
//如果此执行程序已关闭，则返回 true。
boolean isShutdown()
//如果关闭后所有任务都已完成，则返回 true。注意，除非首先调用 shutdown 或 shutdownNow，否则 isTerminated 永不为 true。
boolean isTerminated()
//请求关闭、发生超时或者当前线程中断，无论哪一个首先发生之后，都将导致阻塞，直到所有任务完成执行。
boolean awaitTermination(long timeout,TimeUnit unit)throws InterruptedException
//提交一个返回值的任务用于执行，返回一个表示任务的未决结果的 Future。
<T> Future<T> submit(Callable<T> task)
Future<?> submit(Runnable task)
//执行给定的任务，当所有任务完成时，返回保持任务状态和结果的 Future 列表。返回列表的所有元素的 Future.isDone()为true。
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,long timeout,
TimeUnit unit)                
```

## 5. ThreadPoolExecutor详解

### 1. 重点参数详解

- corePoolSize（核心线程数）：有新任务提交时，如果此时线程池中的线程数少于 corePoolSize，则即使存在空闲线程，线程池也会创建新线程来执行该任务。当线程池中的线程数大于corePoolSize时就不再创建新线程，而是将任务保存在BlockingQueue中。
- maximumPoolSize（允许的最大线程数）：如果线程池中的线程数多于 corePoolSize 而少于 maximumPoolSize，且任务队列已经饱和，则线程池会继续创建新线程，直到线程数达到maximumPoolSize。
- keepAliveTime ：当线程池中的线程数大于corePoolSize时，则大余的线程在空闲时间超过keepAliveTime后将会终止。keepAliveTime提供了当池处于非活动状态时减少资源消耗的方法。
- BlockingQueue任务队列：用于保存等待执行的任务的阻塞队列。
  - 如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。
  - 如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。
  - 如果无法将请求加入队列，则创建新的线程，除非线程的数量超出 maximumPoolSize，在这种情况下，任务将被拒绝。
    BlockingQueue有三种常用策略：
    1. 直接提交。即使用SynchronousQueue。SynchronousQueue是一个没有容量的阻塞队列。当有新任务到达时，首先执行SynchronousQueue.offer()，当前线程池中有空闲线程正在执行SynchronousQueue.poll()，则主线程将任务分配给空闲线程执行。如果不存在这样的空闲线程，则试图把任务加入队列将失败，因此会构造一个新的线程。使用SynchronousQueue通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。
    2. 无界队列。即使用LinkedBlockingQueue。这样当所有 corePoolSize 线程都忙时新任务会加入LinkedBlockingQueue队列中等待。这样创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize、keepAliveTime的值也就无效了。）无界队列适合于任务执行互不影响时。
    3. 有界队列。即使用ArrayBlockingQueue。有助于防止资源耗尽。
    4. 优先级队列：PriorityBlockingQueue
- ThreadFactory：使用ThreadFactory创建新线程。默认使用Executors.defaultThreadFactory()创建线程。通过提供不同的ThreadFactory，可以改变线程的名称、线程组、优先级、守护进程状态。
- RejectedExecutionHandler：当 Executor 已经关闭，或者当BlockingQueue为有界队列且线程池中的线程数达到maximumPoolSize时（即队列和线程池都处于饱和状态），这时再有任务进入时线程池会执行拒绝执行策略。JDK1.5中Java线程池提供了4种策略：

  - ThreadPoolExecutor.AbortPolicy：处理程序遭到拒绝将抛出运行时 RejectedExecutionException。
  - ThreadPoolExecutor.CallerRunsPolicy：用调用者所在线程来运行任务。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
  - ThreadPoolExecutor.DiscardPolicy：不做任何处理直接删除该任务。
  - ThreadPoolExecutor.DiscardOldestPolicy：删除位于工作队列头部的任务，然后重试执行该任务（如果再次失败，则重复此过程）。

### 2. 常见的ThreadPoolExecutor类型

#####  1. FixedThreadPool
固定线程数的线程池，采用LinkedBlockingQueue任务队列。线程池中最多有 corePoolSize 个线程会处于活动状态。当线程数超过corePoolSize时，新任务会放入LinkedBlockingQueue中等待。如果某个线程由于未预期的Exception而结束，线程池会创建新线程来执行后续的任务。所有线程都会一直存于线程池中，直到显式的执行 ExecutorService.shutdown() 关闭。

```java
// 参数定义
corePoolSize:nThreads
maximumPoolSize:nThreads
keepAliveTime:0L //超过corePoolSize的线程在执行完后会立即终止。
BlockingQueue：LinkedBlockingQueue<Runnable>
```


#####  2. SingleThreadExecutor
如果这个线程异常结束，会创建一个新的线程。

```java
//参数定义
	corePoolSize:1
    maximumPoolSize:1,
    keepAliveTime:0L,
    BlockingQueue：LinkedBlockingQueue<Runnable>
```


##### 3. CachedThreadPool
可根据需要创建新线程的线程池，如果现有线程可用则重用现有线程，否则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任何资源。适用于执行短期异步任务。
```java
//参数定义
	corePoolSize:0, 
	maximumPoolSize:Integer.MAX_VALUE,
    keepAliveTime:60L, //意味着CachedThreadPool中的空闲线程等待新任务的最长时间为60秒
    BlockingQueue:SynchronousQueue<Runnable>
```

## 6. ScheduledThreadPoolExecutor详解

##### 1. ScheduledThreadPool

#####  2. SingleThreadScheduledExecutor



## 7. Executors工厂类

Executors可以创建3种类型的ThreadPoolExecutor：FixedThreadPool、SingleThreadExecutor、CachedThreadPool。

Executors可以创建2种类型的ScheduledThreadPoolExecutor：ScheduledThreadPool、SingleThreadScheduledExecutor。

## 8. Future

`Future` 表示异步计算的结果。

Future的三种状态：

- 未启动：FutureTask对象的任务还没有被执行。
- 已启动：FutureTask对象的任务正在被执行。
- 已完成：FutureTask对象的任务正常结束，或者被取消，或者抛出异常。

```java
boolean	cancel(boolean mayInterruptIfRunning)  //试图取消对此任务的执行。
 V	get() // 等待计算完成（阻塞当前线程来），然后获取其结果。
 V	get(long timeout, TimeUnit unit) //最多等待给定的时间之后，获取其结果（如果结果可用）。
 boolean isCancelled()  //如果在任务正常完成前将其取消，则返回 true。
 boolean isDone() //如果任务已完成，则返回 true。
```

####  FutureTask

FutureTask主要用于要异步获取执行结果或取消执行任务的场景。FutureTask实现了RunnableFuture接口，RunnableFuture接口继承了Runnable和Future接口。FutureTask是Future接口的一个唯一实现类。FutureTask除了实现Future接口外，还实现了Runnable接口，因此FutureTask可以交给Executor执行。

## 9. CompletionService

将生产新的异步任务与使用已完成任务的结果分离开来的服务。生产者提交执行的任务。使用者获取已完成的任务，并按照完成这些任务的顺序处理它们的结果。

由于使用FutureTask的get()方法获取任务的结果时，会阻塞当前线程，此时可使用CompletionService。

CompletionService维护一个保存已完成任务的Future对象的BlockQueue，可以通过take()方法阻塞式或者poll()方法非阻塞式取出其中的Future对象。

### 1. 使用示例

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
CompletionService cs = new ExecutorCompletionService(executor);
for (int i = 0; i < 5; i++) {
    Callable task = () -> {
        int time = new Random().nextInt(20);
        Thread.sleep(time);
        return Thread.currentThread().getName();
    };
    cs.submit(task);
}
for (int i = 0; i < 5; i++) {
    Future take = cs.take();
    System.out.println(take.get());
}
```




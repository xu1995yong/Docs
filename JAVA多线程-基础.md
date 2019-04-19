## 线程与进程

### 线程与进程的区别

进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含多个线程。**进程是资源分配的最小单位**

线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。**线程是cpu调度的最小单位**



## 进程间通信

**进程间通信（IPC，InterProcess Communication）**是指在不同进程之间传播或交换信息。

1. 共享内存：指两个或多个进程共享一个给定的存储区。共享内存是最快的一种 IPC，因为进程是直接对内存进行存取。因为多个进程可以同时操作，所以需要进行同步。信号量+共享内存通常结合在一起使用，信号量用来同步对共享内存的访问。

2. 管道：通常指无名管道，是 UNIX 系统IPC最古老的形式。 管道是半双工的（即数据只能在一个方向上流动），具有固定的读端和写端。管道只能用于具有亲缘关系的进程之间的通信（也是父子进程或者兄弟进程之间）。管道可以看成是一种特殊的文件，对于它的读写也可以使用普通的read、write等函数。但是它不是普通的文件，并不属于其他任何文件系统，并且只存在于内存中。

3. 套接字：

4. 消息队列：

5. 信号量： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。



## 线程的生命周期

![çº¿ç¨ç¶æå¾](https://img-blog.csdnimg.cn/20181120173640764.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BhbmdlMTk5MQ==,size_16,color_FFFFFF,t_70)

1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
2. 就绪：线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权。
3. 运行：就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
4. 阻塞(BLOCKED)：表示线程阻塞于锁。
5. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
6. 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
7. 终止(TERMINATED)：表示该线程已经执行完毕。



## 中断线程

### 常用API

```java
//作用于当前运行的线程。测试当前线程是否已经是中断状态，并清除线程的中断状态标志。
public static boolean interrupted();
//测试调用该方法的线程对象是否已经是中断状态，不清除线程的中断状态标志。
public boolean isInterrupted();
//用于中断线程。即将中断状态标志置为"中断"状态，而不是真正停止线程。需要用户自己去监视线程的状态为并做处理。
public void interrupt(); 
```
### 停止线程的正确方式--利用异常

```java
//Main.java
public class Main {    
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new MyThread());
        t.start();

        Thread.sleep(1000);
        t.interrupt();
        Thread.sleep(5000);
        System.out.println("t还活动吗" + t.isAlive()); // 返回false，说明线程t已结束
    }    
}
//MyThread.java
public class MyThread implements Runnable {
    @Override
    public void run() {
        try {
            //....
            //判断线程的终止状态是否为true，如果为true则抛出InterruptedException异常，跳出正在执行的代码
            if (Thread.interrupted()) {
                throw new InterruptedException();
            }
        } catch (InterruptedException e) {
            //执行特定于任务的清理工作，然后线程便结束了
        }
    }
}    
```

### InterruptedException异常

若线程在处于wait、join、sleep状态时被中断，该线程会抛出**InterruptedException异常**，并清除当前线程的中断状态。        



## Thread类解析

```java
//使当前线程释放CPU，休眠指定的毫秒数。该方法不会释放当前线程上的锁。
public static void sleep(long millis)throws InterruptedException;
//使当前线程释放CPU时间片
public static native void yield();
//当前线程调用t.join()意味着当前线程在此处等待线程T结束。该方法使当前线程释放持有的锁
//该方法内部调用了wait()方法。当线程T终止时将调用this.notifyAll()方法
public final void join(long millis) throws InterruptedException
public final void join(long millis) throws InterruptedException
//判断线程对象是否处于活动状态
public final boolean isAlive()
```







## 线程安全

### 线程安全的定义

多个线程访问同一个对象时，如果不用考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他操作，调用这个对象的行为都可以获得正确的结果，那么这个对象就是线程安全的。

线程安全问题只存在于实例变量或者静态变量中，方法内部的局部变量不存在线程安全问题（局部变量逃逸问题除外），因为方法是在栈中的，而每个线程都有属于自己的栈。







CAS的缺点



ABA问题




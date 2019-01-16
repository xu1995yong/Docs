# JAVA多线程

## 一.	基础知识

### Java中线程的生命周期：

1. 新建(NEW)：新创建了一个线程对象。
2. 可运行(RUNNABLE)：线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于一个可运行线程池中，等待被线程调度选中，获取cpu 的使用权。    
    (线程调用的随机性：某线程的start()方法被调用后，该线程不是立即开始运行，而是进入一个可运行线程池中，等待Cpu以一个不确定的时间来调用线程的run()方法，所以说执行start()方法的顺序不代表线程启动的顺序)	
3. 运行(RUNNING)：可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。
4. 阻塞(BLOCKED)：阻塞状态是指线程因为某种原因放弃了cpu 使用权，也即让出了cpu timeslice，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分三种：   
    1.  等待阻塞：运行(running)的线程执行o.wait()方法，JVM会把该线程放入等待队列(waitting queue)中。    
    2. 同步阻塞：运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。    
    3. 其他阻塞：运行(running)的线程执行Thread.sleep(long ms)或t.join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入可运行(runnable)状态。
5. 死亡(DEAD)：线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。    

### Thread类常用API

1.	currentThread():返回正在调用该段代码的进程的信息。
2.	isAlive()
3.	sleep()
4.	getId()
5.	public static void yield()

### 停止线程

1. 使用interrupt()方法终止线程   
   1).	常用API
   - public static boolean interrupted():作用于当前运行该代码的线程，测试当前线程是否已经是中断状态，并清除线程的中断状态标志。
   - public boolean isInterrupted()：测试调用该方法的线程对象是否已经是中断状态，不清除线程的中断状态标
   - public void interrupt()：用于中断线程。调用该方法的线程的状态为将被置为"中断"状态。    
     注意：线程中断仅仅是置线程的中断状态位，不会停止线程。需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出interruptedException的方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。

2. 停止线程的正确方式--利用异常

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
            //正常代码
            ----------
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

3. 停止阻塞的线程：若线程A在处于wait、join、sleep状态时被中断，线程A会抛出**InterruptedException异常**，并清除当前线程的中断状态。    
4. 守护线程
         

### 线程中断

###  InterruptedException异常

### 关于wait() 和 notify() 方法的说明

1. 为什么wait() 和 notify() 方法属于Object类？

   因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致因调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

2. 为什么wait() 和 notify() 方法却必须在 synchronized 方法或块中调用？

   只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现 IllegalMonitorStateException 异常。

3. 关于 wait() 和 notify() 方法最后再说明两点：
     第一：调用 notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中随
     机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问
     题。

     第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用
     notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有
     获得锁的那一个线程才能进入可执行状态。




## 二.	线程安全

1. 非线程安全：指多个线程对同一个对象中的实例变量进行并发访问时产生的“脏读”的情况。    
   （非线程安全问题只存在于实例变量中，方法内部的私有变量不存在线程安全问题，因为方法是在栈中的，而每个线程都有属于自己的栈。）

   

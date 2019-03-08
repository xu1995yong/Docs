## 线程与进程

进程是资源分配的最小单位，线程是CPU调度的最小单位

| **对比维度**   | **多进程**                                                   | **多线程**                                                   | **总结** |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------- |
| 数据共享、同步 | 数据共享复杂，需要用IPC；数据是分开的，同步简单              | 因为共享进程数据，数据共享简单，但也是因为这个原因导致同步复杂 | 各有优势 |
| 内存、CPU      | 占用内存多，切换复杂，CPU利用率低                            | 占用内存少，切换简单，CPU利用率高                            | 线程占优 |
| 创建销毁、切换 | 创建销毁、切换复杂，速度慢                                   | 创建销毁、切换简单，速度很快                                 | 线程占优 |
| 编程、调试     | 编程简单，调试简单                                           | 编程复杂，调试复杂                                           | 进程占优 |
| 可靠性         | 进程间不会互相影响                                           | 一个线程挂掉将导致整个进程挂掉                               | 进程占优 |
| 分布式         | 适应于多核、多机分布式；如果一台机器不够，扩展到多台机器比较简单 | 适应于多核分布式                                             | 进程占优 |



## 线程的生命周期

- 新建：新创建了一个线程对象。
- 可运行：线程对象创建后，其他线程调用了该线程对象的start()方法，该线程等待获取cpu 的使用权。线程调用具有随机性，执行start()方法的顺序不代表线程启动的顺序。
- 运行：线程获得cpu 时间片，执行程序代码。
- 等待：
- 阻塞：阻塞状态是指线程因为某种原因放弃了cpu 使用权，暂时停止运行。
- 死亡：线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。    


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








## 二.	线程安全

1. 非线程安全：指多个线程对同一个对象中的实例变量进行并发访问时产生的“脏读”的情况。    
   （非线程安全问题只存在于实例变量中，方法内部的私有变量不存在线程安全问题，因为方法是在栈中的，而每个线程都有属于自己的栈。）

   

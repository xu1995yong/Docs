# 线程同步工具类

## CountDownLatch

倒数计数器，初始时设定计数器值，线程可以在计数器上等待，当计数器值归0后，所有等待的线程继续执行。

### CountDownLatch主要接口分析

CountDownLatch工作原理相对简单，可以简单看成一个倒计数器，在构造方法中指定初始值，每次调用*countDown()*方法时将计数器减1，而*await()*会等待计数器变为0。CountDownLatch关键接口如下

- **countDown()** ：如果当前计数器的值大于1，则将其减1；若当前值为1，则将其置为0并唤醒所有通过await等待的线程；若当前值为0，则什么也不做直接返回。
- **await()** ：等待计数器的值为0，若计数器的值为0则该方法返回，否则将持续阻塞；若等待期间该线程被中断，则抛出**InterruptedException**并清除该线程的中断状态。
- **await(long timeout, TimeUnit unit)** ：在指定的时间内等待计数器的值为0，若在指定时间内计数器的值变为0，则该方法返回true；若指定时间内计数器的值仍未变为0，则返回false；若指定时间内计数器的值变为0之前当前线程被中断，则抛出**InterruptedException**并清除该线程的中断状态。
- **getCount()**： 读取当前计数器的值，一般用于调试或者测试。

### CountDownLatch用途分析

1. 将计数器初始化为1的 CountDownLatch用作入口，在调用countDown() 的线程打开入口前，所有调用 await 的线程都一直在入口处等待。
```java
public class Driver {
    private static final int N = 10;
    public static void main() throws InterruptedException {
        CountDownLatch switcher = new CountDownLatch(1);
        for (int i = 0; i < N; ++i) {
            new Thread(new Worker(switcher)).start();//启动Worker线程
        }
        doSomething();
        switcher.countDown();       // 主线程开启开关
    }
}
class Worker implements Runnable {
    private final CountDownLatch startSignal;
    Worker(CountDownLatch startSignal) {
        this.startSignal = startSignal;
    }
    public void run() {
        try {
            startSignal.await();    //所有执行线程在此处等待开关开启
            doWork();
        } catch (InterruptedException ex) {
        }
    }
}
```
2. 将初始计数值为N的 CountDownLatch作为一个完成信号点：使某个线程在其它N个线程完成某项操作之前一直等待。

```java
public class Driver {
    private static final int N = 10;
    public static void main() throws InterruptedException {
        CountDownLatch compsignal = new CountDownLatch(N);
        for (int i = 0; i < N; ++i) {
            new Thread(new Worker(compsignal)).start();//启动Worker线程
        }
        compsignal.await();       // 主线程等待其它N个线程完成
        doSomething();
    }
}
class Worker implements Runnable {
    private final CountDownLatch compSignal;
    Worker(CountDownLatch compSignal) {
        this.compSignal = compSignal;
    }
    public void run() {
        try {
            doWork();
            compSignal.countDown(); //每个线程做完自己的事情后，就将计数器减去1
        } catch (InterruptedException ex) {
        }
    }
}
```

## CyclicBarrier

CyclicBarrier是一个可以循环使用的障栅，初始时设定参与线程数，当线程到达栅栏后，会等待其它线程的到达，当到达栅栏的总数满足指定数后，指定数量的等待线程会被放行。

### CyclicBarrier主要接口分析

CyclicBarrier提供的关键方法如下：

- **await()** ：等待其它参与方的到来（调用await()）。如果当前调用是最后一个调用，则唤醒所有其它的线程的等待并且如果在构造CyclicBarrier时指定了action，当前线程会去执行该action，然后该方法返回该线程调用await的次序（getParties()-1说明该线程是第一个调用await的，0说明该线程是最后一个执行await的），接着该线程继续执行await后的代码；如果该调用不是最后一个调用，则阻塞等待；如果等待过程中，当前线程被中断，则抛出**InterruptedException**；如果等待过程中，其它等待的线程被中断，或者其它线程等待超时，或者该barrier被reset，或者当前线程在执行barrier构造时注册的action时因为抛出异常而失败，则抛出**BrokenBarrierException**。
- **await(long timeout, TimeUnit unit)** ：与*await()*唯一的不同点在于设置了等待超时时间，等待超时时会抛出**TimeoutException**。
- **reset()** ：该方法会将该barrier重置为它的初始状态，并使得所有对该barrier的await调用抛出**BrokenBarrierException**。

### CyclicBarrier用途分析

1. 使用多个线程分段求和，最后汇总结果计算总和。

```java
public static void main(String[] args) {
    final List<Worker> list = new ArrayList<>();
    CyclicBarrier barrier = new CyclicBarrier(10, new Runnable() {
        @Override //当最后一个线程进入障栅后，由最后一个进入 barrier 的线程执行执行此处。
        public void run() {
            int result = 0;
            for (Worker worker : list) {
                result += worker.getResult();
            }
            System.out.println("计算完成，结果为" + result);
        }
    });
    for (int i = 0; i < 10; i++) {
        int start = i * 100 + 1;
        int end = start + 99;
        Worker worker = new Worker(start, end, barrier);
        new Thread(worker).start();
        list.add(worker);
    }
}
class Worker implements Runnable {
    private int result = 0;
    private final int start;
    private final int end;
    private CyclicBarrier barrier;
    public Worker(int start, int end, CyclicBarrier barrier) {
        this.start = start;
        this.end = end;
        this.barrier = barrier;
    }
    @Override
    public void run() {
        for (int i = start; i <= end; i++) {
            this.result += i;
        }
        try {
            barrier.await();//线程在屏障处处阻塞，直到阻塞的线程数达到预设的线程数。
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public int getResult() {
        return this.result;
    }
}
```

2. CyclicBarrier的障栅是可以循环使用的。假设CyclicBarrier预设的线程数为N，而此时有N+1个线程到达障栅，CyclicBarrier会按线程到达的顺序放行前N个线程，并将计数器减N，最后一个线程继续阻塞在障栅处。

```java
public static void main(String[] args) throws InterruptedException {
    int N = 2;
    CyclicBarrier barrier = new CyclicBarrier(N);
    for (int i = 0; i < N + 1; i++) {
        new Thread(new Worker(barrier)).start();
    }
    Thread.sleep(10000);
    new Thread(new Worker(barrier)).start();
}
static class Worker implements Runnable {
    private CyclicBarrier cyclicBarrier;
    public Worker(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }
    @Override
    public void run() {
        System.out.println("线程" + Thread.currentThread().getName() + "开启");
        try {
            Thread.sleep(500);
            cyclicBarrier.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "线程运行结束...");
    }
}
```

### CyclicBarrier中的BrokenBarrierException异常

CyclicBarrier的**await()**方法除了抛出**InterruptedException**异常外，还会抛出**BrokenBarrierException**。**BrokenBarrierException**表示当前的障栅已经损坏，已经等不到所有线程都到达障栅了，而已经在等待的线程也没必要再等了。

出现以下几种情况之一时，当前等待线程会抛出**BrokenBarrierException**异常：
1. 其它某个正在await等待的线程被中断了
2. 其它某个正在await等待的线程超时了
3. 某个线程重置了**CyclicBarrier**(调用了**reset**方法)

另外，只要正在Barrier上等待的任一线程抛出了异常，那么Barrier就会认为肯定是凑不齐所有线程了，就会将栅栏置为损坏（Broken）状态，并传播**BrokenBarrierException**给其它所有正在等待（await）的线程。

## Phaser

Phaser是一个多阶段栅栏，可以在初始时设定参与线程数，也可以中途注册/注销参与者，当到达的参与者数量满足栅栏设定的数量后，会进行阶段升级（advance）。Phaser适用于一种任务可以分为多个阶段，但是又不确定几个阶段的情形。

### Phaser中的一些概念

1. phase(阶段)：在Phaser中，phase(阶段)就是栅栏，在任意时间点，Phaser只处于某一个phase(阶段)，初始阶段为0，最大达到Integerr.MAX_VALUE，然后再次归零。当所有参与者都到达后，阶段值会递增。
2. parties(参与者)：parties(参与者)是指参与的线程。在Phaser中，既可以在初始构造时指定参与者的数量，也可以中途通过register()、bulkRegister()、arriveAndDeregister()等方法注册/注销参与者。
3. arrive(到达) / advance(进阶)：Phaser注册完**parties（参与者）**之后，参与者的初始状态是**unarrived**的，当参与者**到达（arrive）**当前阶段（phase）后，状态就会变成**arrived**。当阶段的到达参与者数满足条件后（注册的数量等于到达的数量），阶段就会发生**进阶（advance）**——也就是phase值+1。
4. ermination（终止）：代表当前**Phaser**对象达到终止状态，有点类似于**CyclicBarrier**中的栅栏被破坏的概念。
5. Tiering（分层）：Phaser支持**分层（Tiering）** —— 一种树形结构，通过构造函数可以指定当前待构造的Phaser对象的父结点。之所以引入**Tiering**，是因为当一个Phaser有大量**参与者（parties）**的时候，内部的同步操作会使性能急剧下降，而分层可以降低竞争，从而减小因同步导致的额外开销。 在一个分层Phasers的树结构中，注册和撤销子Phaser或父Phaser是自动被管理的。当一个Phaser的**参与者（parties）**数量变成0时，如果有该Phaser有父结点，就会将它从父结点中移除。

### Phaser主要接口分析

- **arriveAndAwaitAdvance()** 当前线程当前阶段执行完毕，等待其它线程完成当前阶段。如果当前线程是该阶段最后一个未到达的，则该方法直接返回下一个阶段的序号（阶段序号从0开始），同时其它线程的该方法也返回下一个阶段的序号。该方法不响应中断。
- **arriveAndDeregister()** 该方法立即返回下一阶段的序号，并且其它线程需要等待的个数减一，并且把当前线程从之后需要等待的成员中移除。如果该Phaser是另外一个Phaser的子Phaser（层次化Phaser会在后文中讲到），并且该操作导致当前Phaser的成员数为0，则该操作也会将当前Phaser从其父Phaser中移除。
- **arrive()** 该方法不作任何等待，直接返回下一阶段的序号。
- **awaitAdvance(int phase)** 该方法等待某一阶段执行完毕。如果当前阶段不等于指定的阶段或者该Phaser已经被终止，则立即返回。该阶段数一般由*arrive()*方法或者*arriveAndDeregister()*方法返回。返回下一阶段的序号，或者返回参数指定的值（如果该参数为负数），或者直接返回当前阶段序号（如果当前Phaser已经被终止）。
- **awaitAdvanceInterruptibly(int phase)** 效果与*awaitAdvance(int phase)*相当，唯一的不同在于若该线程在该方法等待时被中断，则该方法抛出**InterruptedException**。
- **awaitAdvanceInterruptibly(int phase, long timeout, TimeUnit unit)** 效果与*awaitAdvanceInterruptibly(int phase)*相当，区别在于如果超时则抛出**TimeoutException**。
- **bulkRegister(int parties)** 注册多个party。如果当前phaser已经被终止，则该方法无效，并返回负数。如果调用该方法时，*onAdvance*方法正在执行，则该方法等待其执行完毕。如果该Phaser有父Phaser则指定的party数大于0，且之前该Phaser的party数为0，那么该Phaser会被注册到其父Phaser中。
- **forceTermination()** 强制让该Phaser进入终止状态。已经注册的party数不受影响。如果该Phaser有子Phaser，则其所有的子Phaser均进入终止状态。如果该Phaser已经处于终止状态，该方法调用不造成任何影响。

### Phaser用途分析

1. 通过Phaser控制多个线程的执行时机

```java
public static void main(String[] args) {
    Phaser phaser = new Phaser();
    for (int i = 0; i < 10; i++) {
        phaser.register();                  // 注册各个参与者线程
        new Thread(new Task(phaser), "Thread-" + i).start();
    }
}
static class Task implements Runnable {
    private final Phaser phaser;
    Task(Phaser phaser) {
        this.phaser = phaser;
    }
    @Override
    public void run() {
        int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达
        // do something
        System.out.println(Thread.currentThread().getName() + ": 执行完任务，当前phase =" + i + "");
    }
}
```

2. 通过**Phaser**控制任务终止

```java
public static void main(String[] args) throws IOException {
    int repeats = 3;    // 指定任务最多执行的次数
    Phaser phaser = new Phaser() {
 //在创建Phaser对象时，覆写了onAdvance方法，这个方法类似于CyclicBarrier中的barrierAction任务。也就是说，当最后一个参与者到达时，会触发onAdvance方法，入参phase表示到达时的phase值，registeredParties表示到达时的参与者数量。当返回为true时表示需要终止Phaser。
        @Override
        protected boolean onAdvance(int phase, int registeredParties) {
            System.out.println("------PHASE[" + phase + "],Parties[" + registeredParties + "] --");
            return phase + 1 >= repeats || registeredParties == 0;
        }
    };
    for (int i = 0; i < 10; i++) {
        phaser.register();                      // 注册各个参与者线程
        new Thread(new Task(phaser), "Thread-" + i).start();
    }
}
static class Task implements Runnable {
    private final Phaser phaser;
    Task(Phaser phaser) {
        this.phaser = phaser;
    }
    @Override
    public void run() {
        while (!phaser.isTerminated()) {   //只要Phaser没有终止, 各个线程的任务就会一直执行
            int i = phaser.arriveAndAwaitAdvance();     // 等待其它参与者线程到达
            // do something
            System.out.println(Thread.currentThread().getName() + ": 执行完任务");
        }
    }
}
```

  

## 三者的比较

三者都能够实现线程之间的等待，只不过它们侧重点不同：

1. CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
2. CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
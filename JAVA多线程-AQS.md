
## AbstractQueuedSynchronizer队列同步器

AbstractQueuedSynchronizer是用来构建锁或者其他同步组件的基础框架。AQS框架提供了一套通用的机制来管理同步状态、阻塞/唤醒线程、管理等待队列。

### AbstractQueuedSynchronizer的使用

AbstractQueuedSynchronizer主要用来构建同步组件。如果需要自定义一个同步组件，只需在自定义同步组件中实现一个静态内部类并继承AbstractQueuedSynchronizer。之后将组件的操作代理到该AbstractQueuedSynchronizer的子类中即可。具体的使用可参考ReentrantLock。

由于AbstractQueuedSynchronizer的设计是基于模板方法模式的，所以使用者需要重写AbstractQueuedSynchronizer类中指定的方法。AbstractQueuedSynchronizer提供的模板方法会调用使用者重写的方法。

需要重写的方法包括（只需要重写其中需要的部分即可）：

```java
//独占式锁的获取和释放
protected boolean tryAcquire(int arg)
protected boolean tryRelease(int arg)
//共享式锁的获取和释放
protected int tryAcquireShared(int arg)//共享式获取同步状态，若返回大于等于0的值则表示获取成功，
protected boolean tryReleaseShared(int arg)
//表示同步器是否被当前线程在独占模式下使用。此方法仅在ConditionObject方法内部调用。
protected boolean isHeldExclusively()
```

### 核心成员变量
```java
private transient volatile Node head; //指向等待队列的头节点。头节点一定是获取了资源的节点
private transient volatile Node tail;//指向等待队列的尾节点

//表示同步状态。在ReentrantLock中，该值表示获取资源的线程重入锁的次数。为0时表示资源没有加锁。
private volatile int state;
 
 
// 等待队列节点的数据类型
static final class Node {
        static final Node SHARED = new Node(); //共享模式
        static final Node EXCLUSIVE = null;  //独占模式

        static final int CANCELLED =  1;//表示节点被取消
        static final int SIGNAL    = -1;//当前节点的后继节点包含的线程需要被唤醒
        static final int CONDITION = -2;//表示节点在condition队列中
        static final int PROPAGATE = -3;//doReleaseShared中设置的(仅用于head节点)，以确保传播继续，即使此后发生了其他操作。

        volatile int waitStatus; //表示后继节点的等待状态。初始值为0
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;// 表示附加在该节点上的线程
        Node nextWaiter;//表示在独占锁模式下，条件队列中的下一个节点。（条件队列复用了等待队列）

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```
### 独占式同步状态的获取

#### acquire(int arg)方法

```java
//该方法是AQS独占式同步状态获取的模板方法
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();//获取资源及等待资源过程中不会响应中断，获取资源后补响应中断
}
```
#### tryAcquire(int arg)方法

```java
//此方法需要在子类中实现。
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

#### addWaiter(Node mode)方法

```java
//将该线程加入等待队列的尾部，并标记为独占模式；
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);//独占模式下mode为null
    //先快速尝试将线程加到队列尾部，如果失败，则调用enq(Node node)方法将线程加到队列尾部
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) { //CAS的方式
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node; //返回该节点
}
```
#### enq(final Node node)方法

```java
private Node enq(final Node node) {
    //通过“死循环”的方式保证节点可以正确添加，只有成功添加后当前线程才会从该方法返回。
    for (;;) {
        Node t = tail;
        if (t == null) { //初始化等待队列
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {//CAS的方式将该节点添加到等待队列的尾部
                t.next = node;
                return t; //设置成功后，返回等待队列中该节点的前一个节点
            }
        }
    }
}
```
#### acquireQueued(final Node node, int arg)方法

```java
//该方法使线程在等待队列中“死循环式”获取资源，获取到资源后才返回。返回值是线程的中断状态
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;//该线程是否被中断
        for (;;) {
            final Node p = node.predecessor();//获取该线程的前驱节点
            //只有当线程的前驱节点是头节点时，才尝试获取资源。
            if (p == head && tryAcquire(arg)) {
                setHead(node);//如果获取成功，将自己置为头节点，
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //如果无法获取资源，则判断是否应该阻塞自己
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
#### shouldParkAfterFailedAcquire(Node pred, Node node)方法

```java
//该方法判断当前线程是否应该阻塞自己
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)//Node.SIGNAL，代表前驱释放资源后会通知后继结点，此时该节点可以安心阻塞
        return true;
    if (ws > 0) { //表示前驱节点已经被取消，需要找到一个没有被取消的前驱节点
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;//删除被取消的前驱节点
    } else {
        //CSA的方式将前驱节点的状态置为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;//不能阻塞自己
}
```
#### parkAndCheckInterrupt()方法

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);//阻塞自己
    return Thread.interrupted();
}
```

### 独占式同步状态的释放

#### release(int arg)方法

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0) //状态不为0则需要唤醒后继结点
            unparkSuccessor(h);//唤醒后继节点
        return true;
    }
    return false;
}
```
#### tryRelease(int arg)方法

```java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```
#### unparkSuccessor(Node node)方法

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);//将当前结点状态设置为零
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {//后继节点已经被取消
        s = null;
        // 从队尾开始向前寻找，找到队列中第一个没有被取消的结点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)//找到后唤醒该线程
        LockSupport.unpark(s.thread);
}
```

### 共享式同步状态的获取

#### acquireShared(int arg)方法

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
#### tryAcquireShared(int arg)方法

```java
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
```
#### doAcquireShared(int arg)方法

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);//以共享模式加入队尾
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();//获取该节点的前驱节点
            if (p == head) {//如果前驱节点是头节点，再次尝试获取资源
                int r = tryAcquireShared(arg);
                if (r >= 0) {//获取资源成功
                    setHeadAndPropagate(node, r);//将自己置为头节点并继续向后唤醒节点
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

```
#### setHeadAndPropagate(Node node, int propagate)方法

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);

    if (propagate > 0 || h == null || h.waitStatus < 0 || (h = head) == null || h.waitStatus < 0){
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```
#### doReleaseShared()方法

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            
                unparkSuccessor(h);
            }
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;               
        }
        //后继节点被唤醒之后，首先会将自己设置成等待队列的头节点。
        //该处判断等待队列的头节点被改变后，会继续循环，来唤醒后面的节点。
        if (h == head)                
            break;
    }
}
```
### 共享式同步状态的释放

#### releaseShared(int arg)方法

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
#### tryReleaseShared(int arg) 方法

```java
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
```




## Condition接口

任意一个JAVA对象，都拥有一组定义在Object类上的监视器方法，比如：wait()方法、wait(long outtime)方法、notify()方法、notifyAll()方法等。这些方法与synchronized配合使用，可以实现等待/通知模式。

Condition接口也提供了类似Object的监视器方法，且与Lock类配合使用同样可以实现等待/通知模式。Condition只在独占锁中使用。

**Condition对象是绑定到锁上的。要获取特定Lock上的Condition对象，应使用锁对象的newCondition()方法**

### Condition接口与Object类的监视器方法的对比

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170521191239130?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnV5dXdlaTIwMTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 常用方法

```java
// 使当前线程处于等待状态，释放与Condition绑定的lock锁。直到 singal()方法被调用后被唤醒。之后该线程会再次获取与Condition绑定的lock锁
void await() throws InterruptedException;
// 相比较await()而言，不响应中断
void awaitUninterruptibly();
// 在wait()的返回条件基础上增加了超时响应，返回值表示当前剩余的时间 < 0 ，则表示超时
long awaitNanos(long nanosTimeout) throws InterruptedException;
// 同上，只是时间参数不同而已
boolean await(long time, TimeUnit unit) throws InterruptedException;
// 同上，只是时间参数不同而已
boolean awaitUntil(Date deadline) throws InterruptedException;
void signal();
// 唤醒所有被条件阻塞的线程。
void signalAll();
```



## ConditionObject

ConditionObject是Condition接口在AQS中的实现类。

### 实现原理分析

#### 核心成员变量

```java
private transient Node firstWaiter;//条件队列的头节点
private transient Node lastWaiter;//条件队列的尾节点
```

#### await()方法

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();//将线程加入到条件队列中
    int savedState = fullyRelease(node);//释放该线程持有的锁，并保存该线程重入该锁的次数
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {//判断该节点是否在等待队列中。如果不在等待队列中则阻塞该线程。
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //运行到此处时该节点已被唤醒。此时节点已被加入到等待队列。
    //节点在等待队列中等待获取之前被释放的锁，并设置锁被释放前的重入次数
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

#### addConditionWaiter()方法

```java
private Node addConditionWaiter() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node t = lastWaiter;
    // 如果尾节点被取消，就清理掉
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Node.CONDITION);//以当前线程新建一个节点，并将其添加到条件队列的末尾
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

#### fullyRelease(Node node)方法

```java
final int fullyRelease(Node node) {
    try {
        int savedState = getState();
        if (release(savedState))
            return savedState;
        throw new IllegalMonitorStateException();
    } catch (Throwable t) {
        node.waitStatus = Node.CANCELLED;
        throw t;
    }
}
```
#### isOnSyncQueue(Node node)方法

```java
final boolean isOnSyncQueue(Node node) {
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    if (node.next != null) // If has successor, it must be on queue
        return true;
    /*
         * node.prev can be non-null, but not yet on queue because
         * the CAS to place it on queue can fail. So we have to
         * traverse from tail to make sure it actually made it.  It
         * will always be near the tail in calls to this method, and
         * unless the CAS failed (which is unlikely), it will be
         * there, so we hardly ever traverse much.
         */
    return findNodeFromTail(node);
}
```

#### unlinkCancelledWaiters()方法

```java
//从条件队列中删除节点的状态不为Node.CONDITION的节点
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

#### signal()方法

```java
//该方法将条件队列中的第一个节点移动到等待队列中
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

#### doSignal(Node first)方法

```java
private void doSignal(Node first) {
    do {
        //将条件队列的firstWaiter指针修改为其nextWaiter
        //如果头节点的下一节点为null，则将Condition的lastWaiter置为null
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) && (first = firstWaiter) != null);//如果转移失败且条件队列中头节点不为空，则继续循环尝试转移
}
```

####  transferForSignal(Node node)方法

```java
//将节点从条件队列中转移到等待队列中
final boolean transferForSignal(Node node) {
    //如果当前节点的waitStatus重新设置为初始值0
    if (!node.compareAndSetWaitStatus(Node.CONDITION, 0))
        return false;
    Node p = enq(node);//将该节点“死循环”的方式添加到等待队列，并返回等待队列中该节点的前一个节点。
    int ws = p.waitStatus;
    //将前驱节点的waitStatus设置为SIGNAL
    //如果前驱节点已被取消或者将前驱节点的waitStatus设置SIGNAL失败，则直接唤醒当前节点。注意此时该节点还在等待队列中，所以该节点被唤醒后会再次在await()中阻塞。
    if (ws > 0 || !p.compareAndSetWaitStatus(ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```


## Lock接口

### Lock接口方法

```java
public void lock()//获取锁。如果锁被其他线程获取则阻塞当前线程，直到获取到锁。
public void lockInterruptibly()//可中断的获取锁。如果锁被其他线程获取则阻塞当前线程，直到获取到锁或被中断。
public boolean tryLock()//尝试获取锁并立即返回。获取成功返回true，获取失败返回false。
public boolean tryLock(long timeout,TimeUnit unit)//在指定的等待时间内尝试获取锁。如果获取到锁则立即返回true，如果指定时间内还未获取到锁则返回false。
public void unlock()// 释放锁
public Condition newCondition()//返回用来与此 Lock 实例一起使用的 Condition 实例。
```

## ReentrantLock类

### 总体介绍

ReentrantLock类，实现了Lock接口，是一种**可重入**的**独占锁**，ReentrantLock不仅具有与 `synchronized` 相同的一些基本行为和语义，还提供了许多扩展功能。ReentrantLock内部通过内部类实现了AQS框架(AbstractQueuedSynchronizer)的API来实现**独占锁**的功能。

### ReentrantLock与synchronized的对比

相同点：

ReentrantLock与synchronized都实现了可重入的功能。

不同点：

1. ReentrantLock可以在创建时指定锁的争用策略。默认为**非公平策略**。
    **非公平策略：**在多个线程争用锁的情况下，能够最终获得锁的线程是随机的（由底层OS调度）。一般情况下，非公平策略效率更高。
    **公平策略：**在多个线程争用锁的情况下，按先后顺序获得争夺锁。

2. `ReentrantLock`可以响应中断。`synchronized`无法响应中断。

3. ReentrantLock 可限时获取锁，超时不能获得锁，就返回false，不会永久等待构成死锁。

### ReentrantLock实现原理分析

#### 公平锁的获取

##### tryAcquire(int acquires)方法

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //公平锁在获取锁之前首先判断等待队列中有没有前驱节点，如果没有，再将state被设置为1，表示已取得锁
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) { 
            setExclusiveOwnerThread(current);//设置取得独占锁的线程
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {//可重入锁
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
##### hasQueuedPredecessors()方法

```java
public final boolean hasQueuedPredecessors() {
    Node h, s;
    if ((h = head) != null) {
        if ((s = h.next) == null || s.waitStatus > 0) {
            s = null; // traverse in case of concurrent cancellation
            for (Node p = tail; p != h && p != null; p = p.prev) {//从后往前找到一个没有被取消的节点
                if (p.waitStatus <= 0)
                    s = p;
            }
        }
        if (s != null && s.thread != Thread.currentThread())//存在前驱节点
            return true;
    }
    return false;
}
```

#### 非公平锁的获取

##### nonfairTryAcquire(int acquires)

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //非公平锁不管有没有前驱节点，都要尝试去获取锁。
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

#### 锁的释放

##### tryRelease(int releases)

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```
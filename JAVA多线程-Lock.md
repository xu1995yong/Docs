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

1. ReentrantLock可以在创建时指定锁的争用策略。默认为**非公平策略**。synchronized只能是非公平策略。
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

## ReadWriteLock接口

### 总体介绍

**ReadWriteLock接口提供了获取读锁和写锁的方法。 **

读写锁是一对相关的锁，读锁用于只读操作，写锁用于写入操作。读锁可以由多个线程同时保持，而写锁是独占的，只能由一个线程获取。

由于读写锁本身的实现就远比独占锁复杂，因此，读写锁比较适用于以下情形：

1. 高频次的读操作，相对较低频次的写操作；
2. 读操作所用时间不会太短。（否则读写锁本身的复杂实现所带来的开销会成为主要消耗成本）

### 接口方法

```java
Lock readLock();//获取读锁
Lock writeLock();//获取写锁
```



## ReentrantReadWriteLock类

### 总体介绍

ReentrantReadWriteLock类是ReadWriteLock接口的直接实现。ReentrantReadWriteLock类也是通过定义内部类实现AQS框架的API来实现独占/共享的功能。

ReentrantReadWriteLock中有两个静态内部类，分别为ReadLock类和WriteLock类。其中ReadLock类采用共享式同步状态的获取和释放。写锁使用独占式同步状态的获取和释放。



#### ReentrantReadWriteLock类的特点

1. **锁重入**：
2.  **锁降级**：同一个线程可以同时拥有写锁与读锁。但必须先获取写锁再获取读锁，先获取写锁再获取读锁的过程称为锁降级。如果先获取读锁再获取写锁，则获取写锁的过程会使当前线程阻塞，即造成所有线程阻塞，从而导致死锁。



### 实现原理

#### 读写锁计数


写锁是独占锁，使用AQS中的status字段的低16位记录独占锁的重入次数。同时AQS可以记录独占锁的所有信息。

读锁是共享锁，使用AQS中的status字段的高16位记录共享锁的重入次数。由于共享锁可以由多个线程可以同时持有，所以必须记录每个持有共享锁的线程及其所持有的共享锁（读锁）的数量。

```java
//这两个值用来记录第一个获取读锁的线程和该线程持有的该读锁的数量。这两个值的意义是：很多时候，读锁只被一个线程获取，这时候可以单独使用这两个计数值来记录，而不放入readHolds中，从而避免了当只有一个线程操作读锁的时候频繁地在readHolds上读取的问题，提高了效率。
private transient Thread firstReader;
private transient int firstReaderHoldCount;
//HoldCounter类用来记录每个线程持有的读锁数量。
static final class HoldCounter {
    int count;         
    final long tid = LockSupport.getThreadId(Thread.currentThread());
}

//cachedHoldCounter是一个缓存。很多情况下，一个线程获取读锁之后要更新该线程对应的HoldCounter对象，然后有很大可能在很短的时间内就释放掉读锁，这时候需要再次更新HoldCounter，甚至需要从readHolds中删除（如果重入的读锁都被释放掉的话），需要调用readHolds的get方法，这是有一定开销的。因此，设置cachedHoldCounter作为一个缓存，在某个线程需要这个记录值的时候，先检查cachedHoldCounter对应的线程是否是这个线程自己，如果不是的话，再熊readHolds中取出来，这提高了效率。
private transient HoldCounter cachedHoldCounter;

//使用ThreadLocal记录每个线程所持有的共享锁（读锁）的数量。
private transient ThreadLocalHoldCounter readHolds;
```

#### 构造函数

```java
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);//创建读锁
    writerLock = new WriteLock(this);//创建写锁
}
```

#### 读锁的获取

##### tryAcquireShared(int unused)方法

```java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    //如果写锁被其他线程获取，直接返回-1
    if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    //如果获取读锁不需要等待，且CAS式获取读锁成功
    if (!readerShouldBlock() && r < MAX_COUNT && compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) { //如果读锁还没有被获取过
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) { //如果读锁已被获取过，且第一个获取读锁的线程是当前线程
            firstReaderHoldCount++;
        } else {//如果读锁已被获取过，且第一个获取读锁的线程不是当前线程
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != LockSupport.getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();//获取当前线程对应的HoldCounter
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);//如果获取读锁需要等待，或CAS式获取读锁失败
}
```

##### readerShouldBlock()方法

```java
//在公平锁中，如果AQS等待队列中存在未取消的节点，则当前获取读锁的线程应该阻塞
final boolean readerShouldBlock() {
    return hasQueuedPredecessors();
}
public final boolean hasQueuedPredecessors() {
    Node h, s;
    if ((h = head) != null) {
        if ((s = h.next) == null || s.waitStatus > 0) {
            s = null; // traverse in case of concurrent cancellation
            for (Node p = tail; p != h && p != null; p = p.prev) {
                if (p.waitStatus <= 0)
                    s = p;
            }
        }
        if (s != null && s.thread != Thread.currentThread())
            return true;
    }
    return false;
}

//在非公平锁中，如果AQS等待队列中head 的后继节点在等待写锁，则当前获取读锁的线程应该阻塞，避免写锁饥饿。
final boolean readerShouldBlock() {
    return apparentlyFirstQueuedIsExclusive();
}
final boolean apparentlyFirstQueuedIsExclusive() {
    Node h, s;
    return (h = head) != null && (s = h.next)  != null && !s.isShared() && s.thread != null;
}
```

#####  fullTryAcquireShared(Thread current)方法

```java
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        } 
        /**
        如果readerShouldBlock()为真，则通过当前线程是否获取过该读锁，来进一步判断当前线程是否要等待。
        1. 如果当前线程从未获取过该读锁，则将当前线程添加到AQS等待队列中排序。
        2. 如果当前线程获取过该读锁，即此次获取是一次重入，此时如果将当前线程添加到AQS等待队列，就会使当前线程阻塞，从而造成死锁。所以这种情况下只能通过死循环的方式争夺锁。
        **/
        else if (readerShouldBlock()) {
            //如果第一个获取读锁的线程是当前线程，即此次获取读锁是当前线程对读锁的一次重入
            if (firstReader == current) {  
                // assert firstReaderHoldCount > 0;
            } else {  //如果第一个获取读锁的线程不是当前线程
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != LockSupport.getThreadId(current)) {
                        rh = readHolds.get();//获取当前线程对应的HoldCounter
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                //如果rh.count==0，说明当前线程从未获取过读锁。否则说明当前线程获取过读锁
                if (rh.count == 0)
                    return -1; 
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {//CAS式获取锁，如果不成功则“死循环式”自旋
            if (sharedCount(c) == 0) {//如果还没有线程获取过读锁
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != LockSupport.getThreadId(current))
                    rh = readHolds.get();//从readHolds中取出该线程对应的HoldCounter
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

#### 读锁的释放

##### tryReleaseShared(int unused)方法

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        //如果cachedHoldCounter没有缓存HoldCounter，或者缓存的不是当前线程的HoldCounter
        if (rh == null || rh.tid != LockSupport.getThreadId(current))
            rh = readHolds.get();//获取当前线程的HoldCounter
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();//删除是为了防止内存泄漏。因为当前线程已经不再持有读锁
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            //如果 nextc == 0，即 state 全部32位都为0，则说明当前资源读、写锁均不存在，此时需要唤醒AQS等待队列中获取写锁的线程。
            return nextc == 0;
    }
}
```

#### 写锁的获取

##### tryAcquire(int acquires)方法

```java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        //如果 c != 0 且 w == 0 说明存在读锁。此时直接返回false
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        //c != 0 && w ！= 0 && current == getExclusiveOwnerThread() 即该次是写锁重入
        setState(c + acquires);
        return true;
    }
    //如果当前线程获取写锁应该阻塞，或者竞争锁失败，则返回false
    if (writerShouldBlock() ||  !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

##### writerShouldBlock()方法

```java
//公平锁中，如果AQS等待队列中存在未取消的节点，则当前获取写锁的线程应该阻塞
final boolean writerShouldBlock() {
    return hasQueuedPredecessors();
}
public final boolean hasQueuedPredecessors() {
    Node h, s;
    if ((h = head) != null) {
        if ((s = h.next) == null || s.waitStatus > 0) {
            s = null; // traverse in case of concurrent cancellation
            for (Node p = tail; p != h && p != null; p = p.prev) {
                if (p.waitStatus <= 0)
                    s = p;
            }
        }
        if (s != null && s.thread != Thread.currentThread())
            return true;
    }
    return false;
}
//非公平锁中，
final boolean writerShouldBlock() {
    return false; // writers can always barge
}
```

#### 写锁的释放

##### tryRelease(int releases)方法

```java
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively()) //如果获取独占锁的线程不是当前线程
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;//如果独占计数等于0，说明该写锁释放完全，则需要唤醒AQS队列中的节点。
}
```






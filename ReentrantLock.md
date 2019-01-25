## ReentrantLock

### 总体介绍

ReentrantLock类，实现了[Lock](https://segmentfault.com/a/1190000015562196#articleHeader0)接口，是一种**可重入**的**独占锁**，ReentrantLock不仅具有与 `synchronized` 相同的一些基本行为和语义，还提供了许多扩展功能。ReentrantLock内部通过内部类实现了AQS框架(AbstractQueuedSynchronizer)的API来实现**独占锁**的功能。

### ReentrantLock的使用

```java
public static ReentrantLock lock = new ReentrantLock();
public static void main(String[] args) throws InterruptedException {
    Runnable task = () -> {
            lock.lock();
            try {
               // ... method body
            } finally {
                lock.unlock(); //为了确保锁的最终释放，unlock()方法一般写在finally中
            }
        }
    };
}
```



### ReentrantLock常用方法

```java
/***********************继承自LOCK接口的方法************************/
public void lock()//获取锁。如果锁被其他线程获取则阻塞当前线程，直到获取到锁。
public void lockInterruptibly()//可中断的获取锁。如果锁被其他线程获取则阻塞当前线程，直到获取到锁或被中断。
public boolean tryLock()//尝试获取锁并立即返回。获取成功返回true，获取失败返回false。
public boolean tryLock(long timeout,TimeUnit unit)//在指定的等待时间内尝试获取锁。如果获取到锁则立即返回true，如果未获取到锁则此线程阻塞。如果指定时间内还未获取到锁则返回false。
public void unlock()// 释放锁
public Condition newCondition()//返回用来与此 Lock 实例一起使用的 Condition 实例。
/********************************/
public boolean isHeldByCurrentThread()//
```

### ReentrantLock与synchronized的对比

相同点：

ReentrantLock与synchronized都实现了可重入的功能。

不同点：

1. ReentrantLock可以在创建时指定锁的争用策略。默认为**非公平策略**。
    **非公平策略：**在多个线程争用锁的情况下，能够最终获得锁的线程是随机的（由底层OS调度）。一般情况下，非公平策略效率更高。
    **公平策略：**在多个线程争用锁的情况下，公平策略倾向于将访问权授予等待时间最长的线程。

2. `ReentrantLock`可以响应中断。`synchronized`无法响应中断。

3. ReentrantLock 可限时获取锁，超时不能获得锁，就返回false，不会永久等待构成死锁。
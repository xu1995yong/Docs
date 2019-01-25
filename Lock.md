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


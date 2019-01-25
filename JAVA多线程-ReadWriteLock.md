## ReadWriteLock接口

### 总体介绍

ReadWriteLock接口是一个单独的接口（未继承Lock接口），该接口提供了获取读锁和写锁的方法。

读写锁是一对相关的锁——读锁和写锁，读锁用于只读操作，写锁用于写入操作。读锁可以由多个线程同时保持，而写锁是独占的，只能由一个线程获取。

由于读写锁本身的实现就远比独占锁复杂，因此，读写锁比较适用于以下情形：

1. 高频次的读操作，相对较低频次的写操作；
2. 读操作所用时间不会太短。（否则读写锁本身的复杂实现所带来的开销会成为主要消耗成本）

### 接口方法

```java
Lock readLock();//获取读锁
Lock writeLock();//获取写锁
```



### ReentrantReadWriteLock实现类

#### 总体介绍

ReentrantReadWriteLock类是[ReadWriteLock接口](https://segmentfault.com/a/1190000015562196#articleHeader6)的直接实现。ReentrantReadWriteLock类也是通过定义内部类实现AQS框架的API来实现独占/共享的功能。

#### ReentrantReadWriteLock的使用

```java
private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
private final Lock rLock = rwl.readLock();
private final Lock wLock = rwl.writeLock();
//读取
public void get(String key) {
    rLock.lock();
    try {
        //读操作
    } finally {
        rLock.unlock();
    }
}
//写入
public void put(String key, Object value) {
    wLock.lock();
    try {
        //写操作
    } finally {
        wLock.unlock();
    }
}
```


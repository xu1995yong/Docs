
## AbstractQueuedSynchronizer队列同步器

AbstractQueuedSynchronizer是用来构建锁或者其他同步组件的基础框架。AQS框架提供了一套通用的机制来管理同步状态（synchronization state）、阻塞/唤醒线程、管理等待队列。



### AbstractQueuedSynchronizer的使用

AbstractQueuedSynchronizer主要用来构建同步组件。如果需要自定义一个同步组件，只需在自定义同步组件中实现一个静态内部类并继承AbstractQueuedSynchronizer。之后将组件的操作代理到该AbstractQueuedSynchronizer的子类中即可。

由于AbstractQueuedSynchronizer的设计是基于模板方法模式的，所以使用者需要重写AbstractQueuedSynchronizer类中指定的方法。AbstractQueuedSynchronizer提供的模板方法会调用使用者重写的方法。

```java
public class Mutex implements Lock {
    //在自定义的同步组件类内部实现一个AbstractQueuedSynchronizer的静态子类
    private static class Sync extends AbstractQueuedSynchronizer { 
        @Override
        protected boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
        @Override
        protected boolean tryRelease(int arg) {
            if (getState() == 0) {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
    }
    //将自定义同步组件的操作代理到AbstractQueuedSynchronizer的子类中
    private final Sync sync = new Sync();
    @Override
    public void lock() {
        sync.acquire(1);
    }
    @Override
    public void unlock() {
        sync.release(1);
    }
}

```

需要重写的方法包括（）：

```java
///////////////////////独占式获取和释放///////////////////
protected boolean tryAcquire(int arg)//独占式获取同步状态
protected boolean tryRelease(int arg)//独占式释放同步状态
///////////////////////共享式获取和释放///////////////////
protected int tryAcquireShared(int arg)//共享式获取同步状态，若返回大于等于0的值则表示获取成功，否则表示获取失败。
protected boolean tryReleaseShared(int arg)//共享式释放同步状态
//////////////////////////////////////////////////////////
//表示同步器是否被当前线程在独占模式下使用。此方法仅在ConditionObject方法内部调用。
protected boolean isHeldExclusively()
```

### AbstractQueuedSynchronizer模板方法解析

```java

```


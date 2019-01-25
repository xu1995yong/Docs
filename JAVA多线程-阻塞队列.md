# BlockingQueue接口

## 总体概述

BlockingQueue具有以下特点：
- BlockingQueue接口继承自Queue接口。BlockingQueue提供了普通队列的方法。
- BlockingQueue提供了阻塞的插入和移除元素的方法，即**当队列满时，阻塞队列会阻塞插入元素的线程，直到队列不满。当队列空时，阻塞队列会阻塞移除元素的线程，直到队列不空**。
- BlockingQueue队列中不包含null元素；
- BlockingQueue接口的实现类都是线程安全的，实现类一般通过“锁”保证线程安全；
- BlockingQueue可以是限定容量的。remainingCapacity()方法用于返回剩余可用容量，对于没有容量限制的BlockingQueue实现，该方法总是返回Integer.MAX_VALUE 。

阻塞队列一般用在**“生产者-消费者”**模式中，用于线程之间的数据交换或系统解耦。

BlockingQueue接口提供的方法如下表所示：

| 操作类型 | 抛出异常  | 返回特殊值 | 阻塞线程 | 超时退出             |
| -------- | --------- | ---------- | -------- | -------------------- |
| 插入     | add(e)    | offer(e)   | put(e)   | offer(e, time, unit) |
| 删除     | remove()  | poll()     | take()   | poll(time, unit)     |
| 读取     | element() | peek()     | /        | /                    |

## ArrayBlockingQueue

### 概述

ArrayBlockingQueue是一个底层数据类型为数组的有界阻塞队列。ArrayBlockingQueue在创建时必须指定底层数组的大小，且创建后数组的大小不能再改变。

默认情况下ArrayBlockingQueue不保证线程公平的访问队列，但是可以通过构造函数来指定公平性，该做法通常会降低吞吐量。所谓公平访问队列是指当队列可用时，按照线程被阻塞的先后顺序访问队列。非公平性是指当队列可用时，所有阻塞的线程都争夺访问队列的资格。

### 常用方法解析



## LinkedBlockingQueue

LinkedBlockingQueue是基于单链表实现的**近似有界的阻塞队列**。LinkedBlockingQueue的默认容量大小为`Integer.MAX_VALUE`，可以在创建时指定队列的容量。

LinkedBlockingQueue与ArrayBlockingQueue不同是在LinkedBlockingQueue中维护了两把锁——`takeLock`和`putLock`。
其中takeLock用于控制出队的并发，putLock用于控制入队的并发。同一时刻，只能只有一个线程能执行入队/出队操作，其余入队/出队线程会被阻塞；但是，入队和出队之间可以并发执行，即同一时刻，可以同时有一个线程进行入队，另一个线程进行出队，这样就可以提升吞吐量。

## DelayQueue
## 概览

![âjava éåæ¡æ¶ æ¥å£âçå¾çæç´¢ç"æ](https://upload-images.jianshu.io/upload_images/3985563-e7febf364d8d8235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/939/format/webp)

###  ArrayList

#### 总体介绍

1.	ArrayList允许放入`null`元素。
2.	ArrayList未实现多线程同步。
3.	ArrayList的底层数据类型为数组。其数据结构如下所示：

```java
Object[] elementData; 
private int size;//表示实际存储的元素的数量。
```

####  ArrayList的扩容机制

ArrayList创建时如果在构造函数中指定了其初始大小，则创建时就创建指定大小的数组。

若未指定初始大小，则使用一个长度为零的空数组，待添加数据时再扩容。

当容量不足时，ArrayList会调用grow()方法自动扩容：

```java
private Object[] grow() {
    return grow(size + 1);
}
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,newCapacity(minCapacity));
}
//计算新容量的方式：
private int newCapacity(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//newCapacity是1.5倍的oldCapacity
    if (newCapacity - minCapacity <= 0) {
  //如果要求的minCapacity比计算出的newCapacity大，且当前elementData是空数组，则返回minCapacity与10相比更大的那个
        //即如果创建ArrayList对象时未指定初始容量，则返回默认容量10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
   //如果要求的minCapacity比计算出的newCapacity大，且当前elementData不是空数组，则返回要求的minCapacity
        return minCapacity;
    }
    //如果计算出的newCapacity比要求的minCapacity大，则比较newCapacity和MAX_ARRAY_SIZE：
    //如果newCapacity比MAX_ARRAY_SIZE小或相等，则返回newCapacity。
    //如果newCapacity比MAX_ARRAY_SIZE大，则比较minCapacity和MAX_ARRAY_SIZE
    //如果minCapacity 比 MAX_ARRAY_SIZE大，则返回Integer.MAX_VALUE，否则返回MAX_ARRAY_SIZE
    return (newCapacity - MAX_ARRAY_SIZE <= 0)? newCapacity: hugeCapacity(minCapacity);
}
```


### LinkedList

#### 总体介绍

*LinkedList*同时实现了*List*接口和*Deque*接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（*Queue*），同时又可以看作一个栈（*Stack*）。

*LinkedList*底层通过**双向链表**实现。



## Deque接口

Deque接口提供了在队列两端操纵元素的方法。Stack和Queue都可以直接使用Deque接口来实现其操作。

Deque有两个通用实现：ArrayDeque和LinkedList。官方更推荐使用*AarryDeque*用作栈和队列。

### ArrayDeque

#### 总体介绍

ArrayDeque的底层通过循环数组实现。数组的任何一点都可能被看作起点或者终点。其数据结构如下所示：
```java
Object[] elements;
int head;//head指向队列首端第一个有效元素
int tail;//tail指向队列尾端第一个可以插入元素的空位
```
ArrayDeque不允许放入`null`元素。
ArrayDeque是非线程安全的。

#### 方法剖析

addFirst(E e)方法

```java
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    final Object[] es = elements;
//每次添加元素时，head的值先减1，再将元素添加到head值减1后指向的位置，这样head就始终指向队首的第一个元素。
    es[head = dec(head, es.length)] = e;
    if (head == tail)
        grow(1);
}
```

addLast(E e)方法

```java
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    final Object[] es = elements;
//每次添加元素时，先将元素添加到tail指向的位置，再将tail的值加1。这样tail就始终指向队尾第一个可以插入的位置。
    es[tail] = e;
    if (head == (tail = inc(tail, es.length)))
        grow(1);
}
```

pollFirst()方法

```java
public E pollFirst() {
    final Object[] es;
    final int h;
    E e = elementAt(es = elements, h = head);
    if (e != null) {
    //从对首取元素时，先获取head指向位置中的元素，head指向的位置赋空值，然后head的值加1.
        es[h] = null;
        head = inc(h, es.length);
    }
    return e;
}
```

pollLast()方法

```java
public E pollLast() {
    final Object[] es;
    final int t;
    //从队尾取元素时，tail的值先减1，再获取tail指向的位置中的元素
    E e = elementAt(es = elements, t = dec(tail, es.length));
    if (e != null)
        es[tail = t] = null;
    return e;
}
```

removeFirst()方法

```java
public E removeFirst() {
    E e = pollFirst();
    if (e == null)
        throw new NoSuchElementException();
    return e;
}
```

removeLast()方法

```java
public E removeLast() {
    E e = pollLast();
    if (e == null)
        throw new NoSuchElementException();
    return e;
}
```

## PriorityQueue

### 总体介绍

PriorityQueue是基于小顶堆(任意一个非叶子节点的权值，都不大于其左右子节点的权值)的无界的优先队列。优先队列的元素根据其自然顺序排序，或者由队列构造时传入的比较器排序。

由于PriorityQueue是基于小顶堆实现，所以其内部以数组作为底层数据类型。PriorityQueue不允许插入null值。

### 方法剖析

#### add()方法和offer()方法

add(E e)方法和offer(E e)方法在PriorityQueue中完全相同。将元素添加到队列的尾端，然后通过siftUp()方法调整堆。

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);//自动扩容
    siftUp(i, e);//调整小顶堆
    size = i + 1;
    return true;
}
```

#### siftUp()方法

```java
//从k指定的位置开始，将要插入的节点逐层与当前的parent进行比较并交换，直到满足要插入的节点大于其parent节点为止
private static <T> void siftUpComparable(int k, T x, Object[] es) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = es[parent];
        if (key.compareTo((T) e) >= 0) //要插入的节点每次和其父节点相比
            break; //如果符合小顶堆的定义，直接跳出
        es[k] = e; //如果不符合调整堆
        k = parent;
    }
    es[k] = key;
}
```

#### remove()和poll()方法

remove()`和`poll()`方法都是获取并删除队首元素，区别是当方法失败时前者抛出异常，后者返回`null。由于删除操作会改变队列的结构，为维护小顶堆的性质，需要进行必要的调整。

```java
public E poll() {
    final Object[] es;
    final E result;

    if ((result = (E) ((es = queue)[0])) != null) {
        modCount++;
        final int n;
        final E x = (E) es[(n = --size)];
        es[n] = null;
        if (n > 0) {
            final Comparator<? super E> cmp;
            if ((cmp = comparator) == null)
                siftDownComparable(0, x, es, n);
            else
                siftDownUsingComparator(0, x, es, n, cmp);
        }
    }
    return result;
}
```
#### siftDown()方法

```java
//从k指定的位置开始，将要插入的值逐层向下与当前的左右孩子中较小的交换，直到要插入的值小于等于其左右孩子为止
private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
    Comparable<? super T> key = (Comparable<? super T>)x;
    int half = n >>> 1;           // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; //child指向左孩子
        Object c = es[child];
        int right = child + 1;
        if (right < n && ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
            c = es[child = right];  //如果右孩子比左孩子小，则child指向右孩子
        if (key.compareTo((T) c) <= 0)
            break;
        es[k] = c;
        k = child;
    }
    es[k] = key;
}
```

- 

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK; //该节点的颜色
}
private static final boolean RED   = false;
private static final boolean BLACK = true;
```
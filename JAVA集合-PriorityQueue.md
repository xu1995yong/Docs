## PriorityQueue

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
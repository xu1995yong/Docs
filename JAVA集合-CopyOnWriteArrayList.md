## CopyOnWriteArrayList

CopyOnWriteArrayList是ArrayList的一个线程安全的变体，其中所有能使元素产生变化的操作（add、set 等等）都是通过对底层数组进行一次新的复制来实现的。

CopyOnWriteArrayList，运用了一种**“写时复制”**的思想。即如果需要修改（增/删/改）列表中的元素，不直接进行修改，而是先将列表Copy，然后在新的副本上进行修改，修改完成之后，再将引用从原列表指向新列表。这样做的好处是**读/写是不会冲突**的，可以并发进行，读操作还是在原列表，写操作在新列表。仅仅当有多个线程同时进行写操作时，才会进行同步。所以CopyOnWriteArrayList适用于读多写少”的情形

### 核心方法解析

#### add()方法

**add()**方法首先会进行加锁，保证只有一个线程能进行修改；然后会创建一个大小为n+1新数组，并将原数组的值复制到新数组，新元素插入到新数组的最后；最后将新数组设置为其内部数组。

```java
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}
```

#### remove()方法

```java
public E remove(int index) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        E oldValue = elementAt(es, index);// 获取旧数组中的元素, 用于返回
        int numMoved = len - index - 1;// 需要移动多少个元素
        Object[] newElements;
        if (numMoved == 0) // index位置刚好是最后一个元素
            newElements = Arrays.copyOf(es, len - 1);
        else {
            newElements = new Object[len - 1];
            System.arraycopy(es, 0, newElements, 0, index);
            System.arraycopy(es, index + 1, newElements, index, numMoved);
        }
        setArray(newElements);
        return oldValue;
    }
}
```

#### get()方法

**get()**方法先找到当前指向的内部数组，然后直接返回内部数组对应索引位置的值。可以看出，读操作读到的数据只是一份快照。所以如果希望写入的数据可以立刻被读到，那CopyOnWriteArrayList并不适合。

```java
public E get(int index) {
    return get(getArray(), index);
}
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

#### 迭代

从迭代器的构造方法中可以看出，迭代器的迭代是在旧数组上进行的，因为snapshot指向的对象从创建迭代器的那一刻就确定了，之后也没有改变，所以迭代过程中不会抛出ConcurrentModificationException并发修改异常。另外，迭代器对象也不支持修改方法，全部会抛出UnsupportedOperationException异常

```java
static final class COWIterator<E> implements ListIterator<E> {
    /** Snapshot of the array */
    private final Object[] snapshot;
    /** Index of element to be returned by subsequent call to next.  */
    private int cursor;
    COWIterator(Object[] es, int initialCursor) {
        cursor = initialCursor;
        snapshot = es;
    }
}
```




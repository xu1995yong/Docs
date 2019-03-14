## ConcurrentHashMap

### JDK7中的实现分析

ConcurrentHashMap在JDK1.7中采用锁分段技术 Segment+ HashEntry实现。即在ConcurrentHashMap中，把容器分为多个segment段，每个segment段中有一个HashEntry类型的数组用来保存数据，同时每个segment段有一把锁。这样当多线程修改不同segment段的数据时，线程间就不会存在竞争关系。但是如果多线程同时修改同一个segment段中的数据，还是会存在对锁的争用。

如下所示：

![img](http://upload-images.jianshu.io/upload_images/2184951-af57d9d50ae9f547.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### put(K key, V value)

```java
public V put(K key, V value) {
    Segment<K,V> s;
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask; //根据key的hash值定位到所属的segment段
    if ((s = (Segment<K,V>)UNSAFE.getObject(segments, (j << SSHIFT) + SBASE)) == null)
        s = ensureSegment(j);
    return s.put(key, hash, value, false);//调用所属的segment段的put()方法添加数据
}
```

#### segment类介绍

##### 重要字段

```java
//segment段中用来保存数据的HashEntry类型的数组。其中HashEntry代表一个链表节点
volatile HashEntry<K,V>[] table;
```

##### put()

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;//当前segment段中保存数据的数组
        int index = (tab.length - 1) & hash;//
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                if ((k = e.key) == key || (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}

```
##### scanAndLockForPut(K key, int hash, V value)

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 && (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```



### JDK8中的实现分析

ConcurrentHashMap在JDK1.8中，摒弃了Segment的方式，而是采用Node数组+链表+红黑树的数据结构和Synchronized、死循环 + CAS + volatile的并发控制方式来实现。

![img](http://upload-images.jianshu.io/upload_images/2184951-d9933a0302f72d47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 重要字段

```java
volatile Node<K,V>[] table;

volatile Node<K,V>[] nextTable;

volatile long baseCount;
/*
表初始化和大小调整的控制字段。该值为负数时，表明表正在被初始化或调整大小。其中
 *当为负数时：-1代表正在初始化，-N代表有N-1个线程正在 进行扩容
 *当为0时：代表当时的table还没有被初始化
 *当为正数时：表示初始化或者下一次进行扩容的大小

如果处于扩容状态的话,sizeCtl前 16 位是数据校验标识，后 16 位是当前正在扩容的线程总数
*/
volatile int sizeCtl;

/*
ForwardingNode节点是在table扩容期间插入到桶头的节点。在扩容操作中，我们需要对每个桶中的结点进行分离和转移，如果某个桶结点中所有节点都已经迁移到新 table 中，则会在原 table 的该位置挂上一个 ForwardingNode 结点，说明此桶已经完成迁移，且指明新table的位置。
*/
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        //key,value,next 都为 null,hash 值却为 MOVED。nextTable指向新创建的table
        super(MOVED, null, null);
        this.nextTable = tab;
    }
}
```

####  put()

首先当前线程会获取key对应的hashCode，之后当前线程进入for循环。在for循环中，当前线程首先判断Node数组是否已被初始化。如果已被初始化，当前线程会根据key的hashCode获取在Node数组中对应位置的元素，如果该位置元素为空则以CAS的方式将值插入该位置后跳出循环。

否则如果该位置的元素是forwarding节点，说明Node数组正在扩容且整体还未完成，但该处已迁移完毕，则当前线程进入helpTransfer()方法协助扩容。

如果该位置的元素不为空，且不是forwarding节点类型，则当前线程使用synchronized 锁住该元素，并判断该节点的类型是链表或者红黑树，之后向其中插入值。之后如果链表深度超过 8，则将链表转换为红黑树。此时用synchronized 是因为多线程在链表或者红黑树中遍历然后插入节点的操作，会有线程安全问题。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
        if (tab == null || (n = tab.length) == 0) //如果表为空则初始化表
            tab = initTable();
        //获取key的hash值在table中的对应位置的桶头节点
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //如果桶头节点为空，则CAS的方式赋值
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                  
        }
        /*
        如果桶头节点的hash值为MOVED（即桶头节点是forwarding节点），说明table正在扩容且整体还未完成，但该处已迁移完毕，则当前线程进入helpTransfer()方法协助扩容。
        */
        else if ((fh = f.hash) == MOVED) 
            tab = helpTransfer(tab, f);
        //否则锁住该桶头结点，然后向桶中添加一个节点
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    //向链表中添加节点
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果key在链表中已存在
                            if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            //否则向链表尾部添加节点
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);
                                break;
                            }
                        }
                    }
                    //向红黑树中添加元素
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                //如果链表深度超过 8，则将链表转换为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

####  get(Object key)

在ConcurrentHashMap中，get()方法不需要上锁，因为不会存在线程安全问题。

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());//获取传入的key的hashcode
    //如果传入的key的hash值在数组中对应位置有元素
    if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {
        //该元素的hash值与key的hash值相等，
        if ((eh = e.hash) == h) {
            //再比较该元素的key的值与传入的key的值是否相等，如果相等则返回该元素的value。
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //eh小于0有两种情况，该节点是ForwardingNode节点，或者是红黑树节点。
        else if (eh < 0)  //所以要到ForwardingNode中或者红黑树中查找节点
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {//在链表中查找
            if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

#### initTable()

initTable()方法只允许一个线程对表初始化。首先线程会判断当前sizeCtl字段的值，如果sizeCtl < 0 ，表示已经有线程正在进行初始化操作。则当前线程放弃cpu时间。否则当前线程先以CAS的方式将sizeCtl字段的值改为-1，表示表正在被初始化。然后对Node数组进行初始化。

```java
//该方法只允许一个线程对表初始化
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0) //sizeCtl < 0 说明已经有线程正在进行初始化操作。则当前线程放弃cpu时间
            Thread.yield(); 
        else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) { //当前线程获取了表的初始化资格
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

#### helpTransfer()

```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab = ((ForwardingNode<K,V>) f).nextTable; 
    int sc;
    if (tab != null && (f instanceof ForwardingNode) && nextTab != null) {
        //根据原 table 的长度，返回一个 16 位的扩容校验标识
        int rs = resizeStamp(tab.length);
        while (nextTab == nextTable && table == tab && (sc = sizeCtl) < 0) {
             //判断校验标识是否相等，如果校验符不等或者扩容操作已经完成了，直接退出循环
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
             //否则将sc + 1 标识增加了一个线程进行扩容，然后该线程进入transfer()方法协助扩容。
            if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

#### transfer()

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //计算单个线程允许处理的最少table桶首节点个数，最少为16
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; 
    //如果nextTab为空，则创建一个新table，大小为原table的2倍，并赋给nextTab
    if (nextTab == null) {            
        try {
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1]; 
            nextTab = nt;
        } catch (Throwable ex) {      
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    //创建一个ForwardingNode节点，指向创建的nextTab
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
         	nextBound = (nextIndex > stride ?nextIndex - stride : 0)
            else if (U.compareAndSetInt(this, TRANSFERINDEX, nextIndex,nextBound )) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSetInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```







## CopyOnWriteArrayList

### 总体介绍
CopyOnWriteArrayList是ArrayList的一个线程安全的变体，其中所有能使元素产生变化的操作（add、set 等等）都是通过对底层数组进行一次新的复制来实现的。

CopyOnWriteArrayList，运用了一种**“写时复制”**的思想。即如果需要修改（增/删/改）列表中的元素，不直接进行修改，而是先将列表Copy，然后在新的副本上进行修改，修改完成之后，再将引用从原列表指向新列表。这样做的好处是**读/写是不会冲突**的，可以并发进行，读操作还是在原列表，写操作在新列表，但是**写入的数据不可能立刻被读到**。仅仅当有多个线程同时进行写操作时，才会进行同步。所以CopyOnWriteArrayList适用于读多写少”的情形

### 核心方法解析

#### add()方法

**add()**方法首先会使用synchronized加锁，保证只有一个线程能进行修改；然后会创建一个大小为n+1新数组，并将原数组的值复制到新数组，新元素插入到新数组的最后；最后将新数组设置为其内部数组。

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




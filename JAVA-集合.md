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

## HashSet and HashMap

### 总体介绍

HashSet和HashMap有相同的实现，前者仅仅是对后者做了一层包装，即HashSet里有一个HashMap（适配器模式）。

- **HashMap中的数据类型为数组 + 链表 +红黑树。**其中数组中存放链表头节点的指针。当链表长度大于等于8时，链表会转化为红黑树。
- HashMap实现了Map接口，允许放入 key 为 null 的元素，也允许插入 value 为 null 的元素
- HashMap不保证线程安全。
- HashMap不保证元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散。

将对象放入到 HashMap 或 HashSet 或者 ConcurrentHashMap 中时：hashCode()`和`equals()`。**`hashCode()`方法决定了对象会被放到哪个`bucket`里，当多个对象的哈希值冲突时，`equals()`方法决定了这些对象是否是“同一个对象”**。所以，如果要将自定义的对象放入到`HashMap`或`HashSet`中，需要重写key对象的`hashCode()`和`equals()`方法。**在比较时，先比较key的hash和key的地址是否相同，再使用equals()方法比较两个key对象是否相同**。

### 重要字段

```java
// 最大容量 =  2的30次方（若传入的容量过大，将被最大值替换）  
static final int MAXIMUM_CAPACITY = 1 << 30;
// 2. 加载因子(Load factor)：HashMap在其容量自动增加前可达到多满的一种尺度 加载因子越大，填满的元素越多，空间利用率越高，但是冲突的机会增大；加载因子越小，填满的元素越少，冲突的机会越小，但空间浪费增多。hash值冲突时，会将对象在同一位置存放链表，增加了数据结构的复杂性。冲突的机会越大，则查找的成本越高；冲突的机会越小，查找的成本越小，换言之，查找时间就越小，更快一些。 
final float loadFactor; // 实际加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f; // 默认加载因子 = 0.75

// 3. 扩容阈值（threshold）：当哈希表的大小 ≥ 扩容阈值时，就会扩容哈希表
// a. 扩容 = 对哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数
// b. 扩容阈值 = 容量 x 加载因子
int threshold;

// 4. 其他
transient Node<K,V>[] table;  // 存储数据的Node类型 数组，长度 = 2的幂；数组的每个元素 = 1个单链表
transient int size;// HashMap的大小，即 HashMap中存储的键值对的数量

/** 
* 与红黑树相关的参数
*/
// 1. 桶的树化阈值：即 链表转成红黑树的阈值，在存储数据时，当链表长度 > 该值时，则将链表转换成红黑树
static final int TREEIFY_THRESHOLD = 8; 
// 2. 桶的链表还原阈值：即 红黑树转为链表的阈值，当在扩容（resize（））时（此时HashMap的数据存储位置会重新计算），在重新计算存储位置后，当原有的红黑树内数量 < 6时，则将 红黑树转换成链表
static final int UNTREEIFY_THRESHOLD = 6;
// 3. 最小树形化容量阈值：即 当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树）
// 否则，若桶内元素太多时，则直接扩容，而不是树形化
// 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
static final int MIN_TREEIFY_CAPACITY = 64;
```

![ç¤ºæå¾](https://user-gold-cdn.xitu.io/2018/3/12/16217d70954c947f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 方法剖析

#### hash()

```java
/**
     * 该函数在JDK 1.7 和 1.8 中的实现不同，但原理一样 = 扰动函数 = 使得根据key生成的hash值分布更加均匀、更具备随机性，避免出现hash值冲突（即指不同key但生成同1个hash值）
     * JDK 1.7 做了9次扰动处理 = 4次位运算 + 5次异或运算
     * JDK 1.8 简化了扰动函数 = 只做了2次扰动 = 1次位运算 + 1次异或运算
     */

// JDK 1.7实现：将 键key 转换成 哈希码（hash值）操作  = 使用hashCode() + 4次位运算 + 5次异或运算（9次扰动）
static final int hash(int h) {
    h ^= k.hashCode(); 
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

// JDK 1.8实现：将 键key 转换成 哈希码（hash值）操作 = 使用hashCode() + 1次位运算 + 1次异或运算（2次扰动）
// 1. 取hashCode值： h = key.hashCode() 
// 2. 高位参与低位的运算：h ^ (h >>> 16)  
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    // a. 当key = null时，hash值 = 0，所以HashMap的key 可为null      
    // 注：对比HashTable，HashTable对key直接hashCode（），若key为null时，会抛出异常，所以HashTable的key不可为null
    // b. 当key ≠ null时，则通过先计算出 key的 hashCode()（记为h），然后 对哈希码进行 扰动处理： 按位 异或（^） 哈希码自身右移16位后的二进制
}
```

#### indexFor()

```java
/**
     * 计算存储位置的函数分析：indexFor(hash, table.length)
     * 注：该函数仅存在于JDK 1.7 ，JDK 1.8中实际上无该函数（直接用1条语句判断写出），但原理相同
     * 为了方便讲解，故提前到此讲解
     */
static int indexFor(int h, int length) {  
    return h & (length-1); 
    // 将对哈希码扰动处理后的结果 与运算(&) （数组长度-1），最终得到存储在数组table的位置（即数组下标、索引）
}
```

#### put(K key, V value)方法

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //根据key的hash值查找要存储的位置。如果p为null，说明该位置还没有存储元素，则直接在该位置保存值
    //(n - 1) & hash 等价于 取模运算，一个数对2^n取模 == 一个数和(2^n – 1)做按位与运算 
    if ((p = tab[i = (n - 1) & hash]) == null) 
        tab[i] = newNode(hash, key, value, null);
    else {
        //否则检查在该位置的链表中是否有了该key,是先检查头结点是否为该key，如果不等于则在剩余的节点中寻找
        Node<K,V> e; K k;
        //如果链表的头节点的key与要插入的key相同，则e指针指向头节点
        if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //否则遍历链表
            for (int binCount = 0; ; ++binCount) {
                //如果e指向null，说明链表中找不到相等的key，则新创建节点，插入到链表的最后。
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //如果新插入节点后链表数量 >=8，则将链表转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                //如果链表中找到了与要插入的key相等的节点，则跳出循环。
                if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //如果e不为空，则说明key已经存在，则只需要更新e指向的节点的value即可。
        if (e != null) { 
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

#### get(Object key)方法

该方法的实现思想为：首先根据key得到hashcode，然后根据hashcode得到该key在数组table的存储位置，接着在该位置寻找key值和hashcode值一致的节点即可，如果没有找到，返回null。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&(first = tab[(n - 1) & hash]) != null) {
        //头节点的key的hash与要查找节点的key的hash相同并且地址相同，或者equals()方法判断这两个对象相同
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //否则在剩余的节点中找
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### resize()方法

此方法实现的思想为：

1. 如果table为null，即table还未被初始化时，如果指定了初始化容量（容量必须为2的整数次幂），则按指定的大小初始化table并且计算新阈值，否则按照默认的大小（16）初始化table。
2. 如果table不为空时，则先判断table的原来大小，如果原来table的容量大于要求的最大容量，则使阈值也等于最大容量，且不再对其扩容，而是直接返回原table。否则如果原来table的容量大于等于默认的初始化值，且翻倍后也不会超过最大值，则将table的容量扩大为原来的2倍。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
 
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

#### V remove(Object key)

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?null : e.value;
}
Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable){
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length)>0 &&(p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&((k = e.key) == key ||(key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||(value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

### HashMap的线程安全问题

1. 在JDK1.7中，在多线程下HashMap的transfer()方法在转移链表的过程中，由于采用头插法创建链表，会形成环形链表，这样在调用get()方法获取元素的时候，会造成死循环的问题。但是在JDK1.8中采用尾插法修复了这个问题。
2. 在JDK1.8中，在多线程下向hashmap插入元素时，还是会造成数据丢失的问题。

### HashMap 和 HashTable 的区别

1. HashMap是非线程安全的，HashTable是线程安全的，内部的方法基本都经过synchronized修饰。
2. HashMap允许有null值的存在，而在HashTable中put进的键值只要有一个null，直接抛出NullPointerException。
3. HashMap默认初始化数组的大小为16，HashTable为11。前者扩容时乘2，使用位运算取得哈希，效率高于取模。而后者为乘2加1，都是素数和奇数，这样取模哈希结果更均匀。
4. 哈希值的使用不同。HashTable直接使用对象的hashCode。而HashMap重新计算hash值。 

### 为什么不直接采用经过hashCode（）处理的哈希码 作为 存储数组table的下标位置？

- 结论：容易出现 哈希码 与 数组大小范围不匹配的情况，即 计算出来的哈希码可能 不在数组大小范围内，从而导致无法匹配存储位置

### 为什么采用 哈希码 与运算(&) （数组长度-1） 计算数组下标？

- 结论：根据HashMap的容量大小（数组长度），按需取 哈希码一定数量的低位 作为存储的数组下标位置，从而 解决 “哈希码与数组大小范围不匹配” 的问题

## TreeSet and TreeMap

### 总体介绍

### 红黑树介绍

#### 红黑树的由来

为了改善二叉树中节点的查找效率，提出二叉查找树。二叉查找树需满足如下条件：

1. 若左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2. 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值；
3. 左、右子树也分别为二叉排序树。

由于二叉查找树存在极端情况（即如果要插入的多个元素间是有序的，生成的二叉排序树成为一个链表），破坏了二叉查找树的效率。于是提出了红黑树。

#### 红黑树的定义

红黑树是一种自平衡的二叉查找树。红黑树除满足二叉查找树的要求外，还需要满足以下条件：
1. 每个节点的颜色必须是红色或黑色。
2. 根节点的颜色是黑色。
3. 所有NIL节点都是黑色节点。
4. **每个红色节点的子节点必须是黑色节点。**（从每个NIL节点到根节点的所有路径上不能有两个连续的红色节点。）
5. **从任一节点到其每个叶子节点的所有路径都必须包含相同数目的黑色节点**（简称黑高度）。

![img](https://img-my.csdn.net/uploads/201212/12/1355319681_6107.png)

红黑树是接近平衡的二叉树，因为**红黑树的特殊性质保证了从根节点到NIL节点的最长的可能路径不会多于最短的可能路径的两倍长**。原因如下：

当某条路径最短时，这条路径必然都是由黑色节点构成。当某条路径长度最长时，这条路径必然是由红色和黑色节点相间构成（性质4限定了不能出现两个连续的红色节点）。而性质5又限定了从任一节点到其每个叶子节点的所有路径必须包含相同数量的黑色节点。此时，在路径最长的情况下，路径上红色节点数量 = 黑色节点数量。该路径长度为两倍黑色节点数量，也就是最短路径长度的2倍。

红黑树的修正操作包括：左旋、右旋、节点颜色变换

**红黑树的时间复杂度为: O(lgn)**

#### 红黑树的优点

#### 节点的类型

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
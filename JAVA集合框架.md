# JAVA集合框架

## 概览

![âjava éåæ¡æ¶ æ¥å£âçå¾çæç´¢ç"æ](https://upload-images.jianshu.io/upload_images/3985563-e7febf364d8d8235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/939/format/webp)

## List接口

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

创建对象时可以指定ArrayList的初始大小：

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } 
}
```

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

*LinkedList*底层通过**双向链表**实现，其数据类型如下所示：

```Java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
Node<E> first;//指向链表头节点的指针
Node<E> last;//指向链表尾节点的指针
```

#### 方法剖析

#####  add(E e)方法

```Java
public boolean add(E e) {
    linkLast(e);
    return true;
} 
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);//创建一个新节点，该节点的prev指向l，next指向null
    last = newNode;//链表的尾指针指向新创建的节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

##### add(int index, E element)方法

```Java
public void add(int index, E element) {
    checkPositionIndex(index);
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));//node()方法用于找到该索引出的当前节点
}
void linkBefore(E e, Node<E> succ) {//在succ节点之前插入元素
    final Node<E> pred = succ.prev;
    //新创建一个节点，该节点的prev指向succ的前一个节点，next指向succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

##### get(int index)方法

```Java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
Node<E> node(int index) {
    if (index < (size >> 1)) {//从链表头部开始查找
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else { //从链表尾部开始查找
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

##### remove(int index)方法

```Java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    if (prev == null) {//当该节点不是头节点时
        first = next;
    } else {//当该节点不是头节点时
        prev.next = next;
        x.prev = null;
    }
    if (next == null) {//当该节点是尾节点时
        last = prev;
    } else {//当该节点不是尾节点时
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    size--;
    modCount++;
    return element;
}
```
##### remove(Object o)方法
```java
 //当LinkedList中有多个相同的元素o时，该方法只删除其中的一个。
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

##### set(int index, E element)方法

```Java
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

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

##### addFirst(E e)方法

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

##### addLast(E e)方法

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

##### pollFirst()方法

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

##### pollLast()方法

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

##### removeFirst()方法

```java
public E removeFirst() {
    E e = pollFirst();
    if (e == null)
        throw new NoSuchElementException();
    return e;
}
```

##### removeLast()方法

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
- HashMap*实现了*Map接口，允许放入`key`为`null`的元素，也允许插入`value`为`null`的元素
- HashMap不保证线程安全。
- HashMap不保证元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散。

将对象放入到*HashMap*或*HashSet*中时，有两个方法需要特别关心：`hashCode()`和`equals()`。**`hashCode()`方法决定了对象会被放到哪个`bucket`里，当多个对象的哈希值冲突时，`equals()`方法决定了这些对象是否是“同一个对象”**。所以，如果要将自定义的对象放入到`HashMap`或`HashSet`中，需要重写key对象的`hashCode()`和`equals()`方法。**在比较时，先比较key的hash和key的地址是否相同，再使用equals()方法比较两个key对象是否相同**。

### 重要字段

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; //table的默认初始容量，当未指定初始容量时使用。
static final float DEFAULT_LOAD_FACTOR = 0.75f;
Node<K,V>[] table;
int size;
```

### 方法剖析

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

此方法实现的思想为：1）原table为null的情况，如果为空，则开辟默认大小的空间。2）原table不为空的情况，则开辟原来空间的2倍。由于可能oldCap*2会大于最大容量，因此也对其这种溢出情况进行了处理。
分配空间之后，将原数组中的元素拷贝到新数组中。

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
4. 每个红色节点的子节点必须是黑色节点。（从每个NIL节点到根节点的所有路径上不能有两个连续的红色节点。）
5. 从任一节点到其每个叶子节点的所有路径都必须包含相同数目的黑色节点（简称黑高度）。

![img](https://img-my.csdn.net/uploads/201212/12/1355319681_6107.png)

红黑树是接近平衡的二叉树，因为**红黑树的特殊性质保证了从根节点到NIL节点的最长的可能路径不会多于最短的可能路径的两倍长**。原因如下：

当某条路径最短时，这条路径必然都是由黑色节点构成。当某条路径长度最长时，这条路径必然是由红色和黑色节点相间构成（性质4限定了不能出现两个连续的红色节点）。而性质5又限定了从任一节点到其每个叶子节点的所有路径必须包含相同数量的黑色节点。此时，在路径最长的情况下，路径上红色节点数量 = 黑色节点数量。该路径长度为两倍黑色节点数量，也就是最短路径长度的2倍。

#### 红黑树的修正操作

##### 左旋

##### 右旋

##### 节点颜色变换



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
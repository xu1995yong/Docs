# JAVA集合框架

##  1. ArrayList

###  总体介绍

1.	ArrayList允许放入`null`元素。
2.	ArrayList未实现多线程同步。
3.	ArrayList使用capacity字段表示底层数组的实际大小。
4.	ArrayList使用size字段表示实际存储的元素的数量。


###  方法剖析
**add()**
**addAll()**
**set()**
**get()**

**grow()**

当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。扩容操作最终是通过`grow()`方法完成的。

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//原来的1.5倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);//扩展空间并复制
}
```

**remove()**

`remove()`方法有两个版本，一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; //清除该位置的引用，让GC起作用
    return oldValue;
}
```



## 2. LinkedList

### 总体介绍

*LinkedList*同时实现了*List*接口和*Deque*接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（*Queue*），同时又可以看作一个栈（*Stack*）。

*LinkedList*底层**通过双向链表实现**，本节将着重讲解插入和删除元素时双向链表的维护过程，也即是之间解跟*List*接口相关的函数，而将*Queue*和*Stack*以及*Deque*相关的知识放在下一节讲。双向链表的每个节点用内部类*Node*表示。*LinkedList*通过`first`和`last`引用分别指向链表的第一个和最后一个元素。注意这里没有所谓的哑元，当链表为空的时候`first`和`last`都指向`null`。

```Java
//Node内部类
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
```

### 方法剖析

**add()**

```Java
public boolean add(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;//原来链表为空，这是插入的第一个元素
    else
        l.next = newNode;
    size++;
    return true;
}
```

`add(int index, E element)`的逻辑稍显复杂，可以分成两部分，1.先根据index找到要插入的位置；2.修改引用，完成插入操作。

```Java
//add(int index, E element)
public void add(int index, E element) {
	checkPositionIndex(index);//index >= 0 && index <= size;
	if (index == size)//插入位置是末尾，包括列表为空的情况
        add(element);
    else{
    	Node<E> succ = node(index);//1.先根据index找到要插入的位置
        //2.修改引用，完成插入操作。
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)//插入位置为0
            first = newNode;
        else
            pred.next = newNode;
        size++;
    }
}
```

上面代码中的`node(int index)`函数有一点小小的trick，因为链表双向的，可以从开始往后找，也可以从结尾往前找，具体朝那个方向找取决于条件`index < (size >> 1)`，也即是index是靠近前端还是后端。

**remove()**

`remove()`方法也有两个版本，一个是删除跟指定元素相等的第一个元素`remove(Object o)`，另一个是删除指定下标处的元素`remove(int index)`。

![LinkedList_remove.png](PNGFigures/LinkedList_remove.png)

两个删除操作都要1.先找到要删除元素的引用，2.修改相关引用，完成删除操作。在寻找被删元素引用的时候`remove(Object o)`调用的是元素的`equals`方法，而`remove(int index)`使用的是下标计数，两种方式都是线性时间复杂度。在步骤2中，两个`revome()`方法都是通过`unlink(Node<E> x)`方法完成的。这里需要考虑删除元素是第一个或者最后一个时的边界情况。

```Java
//unlink(Node<E> x)，删除一个Node
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    if (prev == null) {//删除的是第一个元素
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    if (next == null) {//删除的是最后一个元素
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;//let GC work
    size--;
    return element;
}
```

 **get()**

`get(int index)`得到指定下标处元素的引用，通过调用上文中提到的`node(int index)`方法实现。
```Java
public E get(int index) {
    checkElementIndex(index);//index >= 0 && index < size;
    return node(index).item;
}
```

**set()**

`set(int index, E element)`方法将指定下标处的元素修改成指定值，也是先通过`node(int index)`找到对应下表元素的引用，然后修改`Node`中`item`的值。

```Java
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;//替换新值
    return oldVal;
}
```



## 
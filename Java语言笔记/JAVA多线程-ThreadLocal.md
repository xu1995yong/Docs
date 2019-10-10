# ThreadLocal

ThreadLocal，即线程局部变量。为了实现ThreadLocal的功能，每一个线程中都有一个ThreadLocalMap对象，用于保存该线程的局部变量值。但是对该ThreadLocalMap对象的维护工作都是ThreadLocal类的对象进行的。

## ThreadLocal实现原理剖析

### ThreadLocalMap类

ThreadLocalMap是ThreadLocal的静态内部类。ThreadLocalMap用于保存线程与其局部变量的映射。



### get方法

```java
public T get() {
    Thread t = Thread.currentThread();  //获取当前线程
    ThreadLocalMap map = getMap(t);//获取当前线程中保存的ThreadLocalMap对象
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

### setInitialValue()方法

```java
private T setInitialValue() {
    T value = initialValue();//null值
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
    if (this instanceof TerminatingThreadLocal) {
        TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
    }
    return value;
}
```

### createMap(Thread t, T firstValue)方法

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```


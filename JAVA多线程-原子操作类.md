## JAVA中的原子操作类

### Atomic变量

AtomicInteger是Integer类型的线程安全原子类，可以原子的方式更新int值。

在JDK1.8之前，采用了volatile + 自旋 + CAS操作的方式实现。在JDK1.8中，直接调用Unsafe类操作value。

```java
private volatile int value;
```



### Atomic数组

能以原子的方式，操作数组中的元素。JDK提供了三种类型的原子数组：`AtomicIntegerArray`、`AtomicLongArray`、`AtomicReferenceArray`。
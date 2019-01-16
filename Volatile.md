# Volatile


### 1. Volatile对可见性的保证

1. 当写一个volatile变量后，JMM会立即把该线程对应的工作内存中的共享变量值刷新到主内存中。

2. 当读一个volatile变量后， JMM会把该线程对应的工作内存中的值置为无效，强制线程从主内存中读取共享变量的值。

    ##### Volatile对可见性保证的实现原理

    如果对声明了Volatile变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，lock前缀的指令在多核处理器下会引发了两件事情。
     - 将当前处理器缓存行的数据会写回到系统内存。
     - 这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

### 2. Volatile对有序性的保证

volatile关键字能禁止指令重排序，所以volatile能在一定程度上保证有序性。volatile关键字禁止指令重排序有两层意思：
 1. 当程序执行到volatile变量的读操作或者写操作时，在其前面的操作肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
 2. 在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。

### 3. Volatile与锁的区别

锁保证了原子性、可见性、有序性。而Volatile只保证了可见性、有序性，但没有保证原子性，所以volatile在多线程情况下不一定是安全的。

### 4. 使用Volatile必须要同时满足以下两条规则

### （即需要保证操作是原子性操作）

1. 运算结果并不依赖变量的当前值，或者能够保证只有单一的线程修改变量的值。

2. 变量不需要与其他的状态变量共同参与不变约束。

### 5. Volatile关键字的使用场景

1. 用于双重检查锁

   因为在new关键字创建对象时，会发生指令重排序现象（即设置instance引用指向创建的对象  和 Object对象初始化 两个步骤的指令重排序）。所以会发生instance引用已经指向新创建的对象，而该对象还没有完成初始化工作的现象。所以instance变量需要用volatile修饰，禁止指令重排序。

   ```java
   public class DoubleCheckLocking {
   	private volatile static Object instance;
   
   	public static Object getInstance() {
   		if (instance == null) {
   			synchronized (DoubleCheckLocking.class) {
   				if (instance == null) {
   					instance = new Object();
   				}
   			}
   		}
   		return instance;
   	}
   }
   ```

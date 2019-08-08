### JDK监控和故障处理命令

```bash
jps    #JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。
jstat  #用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
jmap   #JVM Memory Map命令用于生成heap dump文件。
jhat   #JVM Heap Analysis Tool命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看。
jstack #用于生成java虚拟机当前时刻的线程快照。
jinfo  #JVM Configuration info 这个命令作用是实时查看和调整虚拟机运行参数。
javap  #查看经javac之后产生的JVM字节码代码，自动解析.class文件, 避免了去理解class文件格式以及手动解析class文件内容。
jcmd   #几乎集合了jps、jstat、jinfo、jmap、jstack所有功能，一个多功能工具, 可以用来导出堆, 查看Java进程、导出线程信息、 执行GC、查看性能相关数据等。



-Xms	初始堆大小	物理内存的1/64(<1GB)	默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.
-Xmx	最大堆大小	物理内存的1/4(<1GB)	默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制
-Xmn	年轻代大小(1.4or lator)
 	注意：此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。
整个堆大小=年轻代大小 + 年老代大小 + 持久代大小.
增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8
-XX:NewSize	设置年轻代大小(for 1.3/1.4)	 	 
-XX:MaxNewSize	年轻代最大值(for 1.3/1.4)	 	 
-XX:PermSize	设置持久代(perm gen)初始值	物理内存的1/64	 
-XX:MaxPermSize	设置持久代最大值	物理内存的1/4	 
-Xss	每个线程的堆栈大小	 	JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右
一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。（校长）
和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"”
-Xss is translated in a VM flag named ThreadStackSize”
一般设置这个值就可以了。
-XX:ThreadStackSize	Thread Stack Size	 	(0 means use default stack size) [Sparc: 512; Solaris x86: 320 (was 256 prior in 5.0 and earlier); Sparc 64 bit: 1024; Linux amd64: 1024 (was 0 in 5.0 and earlier); all others 0.]
```

## OOM

### OOM介绍

OOM，全称“Out Of Memory”，当JVM因为没有足够的内存来为对象分配空间并且垃圾回收器也已经没有空间可回收时，就会抛出OOM Error。

### OOM的类型

按照JVM规范，除了程序计数器不会抛出OOM外，其他各个内存区域都可能会抛出OOM。 

最常见的OOM情况有以下三种：

- java.lang.OutOfMemoryError: Java heap space：java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。对于内存泄露，需要通过内存监控软件查找程序中的泄露代码，而堆大小可以通过虚拟机参数-Xms,-Xmx等修改。
- java.lang.OutOfMemoryError: PermGen space：java永久代溢出，即方法区溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。此种情况可以通过更改方法区的大小来解决，使用类似-XX:PermSize=64m -XX:MaxPermSize=256m的形式修改。另外，过多的常量尤其是字符串也会导致方法区溢出。
- java.lang.StackOverflowError： 不会抛OOM Error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数-Xss来设置栈的大小。

### OOM出现原因分析

1. 流量/数据量峰值：应用程序在设计之初均有用户量和数据量的限制，某一时刻，当用户数量或数据量突然达到一个峰值，并且这个峰值已经超过了设计之初预期的阈值，那么以前正常的功能将会停止，并触发java.lang.OutOfMemoryError: Java heap space异常。
2. 内存泄漏：特定的编程错误会导致你的应用程序不停的消耗更多的内存，每次使用有内存泄漏风险的功能就会留下一些不能被回收的对象到堆空间中，随着时间的推移，泄漏的对象会消耗所有的堆空间，最终触发java.lang.OutOfMemoryError: Java heap space错误。

### OOM排查流程

1. 使用**top**命令，实时显示系统中各个进程的资源占用状况。
2. 使用**ps -aux|grep java**找出当前Java进程的PID 
3. 使用**jstat -gcutil pid interval**用于查看当前GC的状态，它对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。 
4. 使用**jmap -histo:live pid**统计存活对象的分布情况，从高到低查看占据内存最多的对象。
5. 使用Java dump进行诊断。要dump堆的内存镜像，可以采用如下两种方式：
   - 设置JVM参数-XX:+HeapDumpOnOutOfMemoryError，设定当发生OOM时JVM自动dump出堆信息。
   - 使用JDK自带的jmap命令。"jmap -dump:format=b,file=heap.bin <pid>"   其中pid可以通过jps获取。
6. 对dump出的文件进行分析，从而找到OOM的原因。常用的工具有：
   - mat: eclipse memory analyzer, 基于eclipse RCP的内存分析工具。详细信息参见：http://www.eclipse.org/mat/，推荐使用。   
   - jhat：JDK自带的java heap analyze tool，可以将堆中的对象以html的形式显示出来，包括对象的数量，大小等等，并支持对象查询语言OQL，分析相关的应用后，可以通过http://localhost:7000来访问分析结果。不推荐使用，因为在实际的排查过程中，一般是先在生产环境 dump出文件来，然后拉到自己的开发机器上分析，所以，不如采用高级的分析工具比如前面的mat来的高效。
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


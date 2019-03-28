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
```


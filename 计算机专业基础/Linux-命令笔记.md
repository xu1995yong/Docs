查看端口占用：netstat -anp|grep 80 
			：lsof -i:端口号
查看端口被哪个软件占用：lsof -i:端口号

杀死进程：kill -9 进程号

kill和kill -9，两个命令在linux中都有杀死进程的效果，然而两命令的执行过程却大有不同，执行kill命令，系统会发送一个SIGTERM信号给对应的程序。当程序接收到该signal信号后，将会发生以下事情：程序立刻停止、当程序释放相应资源后再停止、程序可能仍然继续运行。大部分程序接收到SIGTERM信号后，会先释放自己的资源，然后再停止。但是也有程序可能接收信号后，做一些其他的事情，然而kill -9命令，系统给对应程序发送的信号是SIGKILL，即exit。exit信号不会被系统阻塞，所以kill -9能顺利杀掉进程。

## top命令

top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况。

前五行是当前系统情况整体的统计信息区。

第一行：任务队列信息，与 uptime 命令的执行结果相同。包括：当前系统时间、系统已经运行时长、当前已登录系统的用户数、平均负载情况。

load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。load average数据是每隔5秒钟检查一次活跃的进程数，然后按计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。

第二行：当前系统中任务（进程）情况

第三行：cpu状态信息
- 5.9%us — 用户空间占用CPU的百分比。
- 3.4% sy — 内核空间占用CPU的百分比。
- 0.0% ni — 改变过优先级的进程占用CPU的百分比
- 90.4% id — 空闲CPU百分比
- 0.0% wa — IO等待占用CPU的百分比
- 0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
- 0.2% si — 软中断（Software Interrupts）占用CPU的百分比

第四行：内存状态

第五行：swap交换分区信息

对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。

**多U多核CPU监控**

在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况：

**通过”shift + >”或”shift + <”可以向右或左改变排序列**


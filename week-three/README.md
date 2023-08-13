# JVM 虚拟机-实战

##  JVM相关工具

### jps

jps: Java Virtual Machine Process Status Tool

```
jps ：列出Java程序进程ID和Main函数名称
jps -q ：只输出进程ID
jps -m ：输出传递给Java进程（主函数）的参数
jps -l ：输出主函数的完整路径
jps -v ：显示传递给Java虚拟的参数
```

### jstat

jstat:JVM Statistics Monitoring Tool

jstat可以查看Java程序运行时相关信息，可以通过它查看运行时堆信息的相关情况。

**示例一：jstat -gc**

※ jps查看进程id

```shell
jstat -gc 7548 250 4
# 进程ID 34784 ，采样间隔250ms，采样数4
```

![jstat -gc](.\note\jstat -gc.png)

```
S0C：年轻代中第一个survivor（幸存区）的容量 （单位kb）
S1C：年轻代中第二个survivor（幸存区）的容量 (单位kb)
S0U ：年轻代中第一个survivor（幸存区）目前已使用空间 (单位kb)
S1U ：年轻代中第二个survivor（幸存区）目前已使用空间 (单位kb)
EC ：年轻代中Eden的容量 (单位kb)
EU ：年轻代中Eden目前已使用空间 (单位kb)
OC ：Old代的容量 (单位kb)
OU ：Old代目前已使用空间 (单位kb)
MC：metaspace的容量 (单位kb)
MU：metaspace目前已使用空间 (单位kb)
CCSC：压缩类空间大小
CCSU：压缩类空间使用大小
YGC ：从应用程序启动到采样时年轻代中gc次数
YGCT ：从应用程序启动到采样时年轻代中gc所用时间(s)
FGC ：从应用程序启动到采样时old代(全gc)gc次数
FGCT ：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)
```

**示例二：jstat -gcutil**

下面输出的是进程内存区域百分百 及 GC详细信息

```shell
jstat -gcutil 7548 1s 5
# 进程ID 30108，采样间隔1s，采样数5
```

![jstat -gcutil](.\note\jstat -gcutil.png)

```
S0 年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
S1 年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
E 年轻代中Eden（伊甸园）已使用的占当前容量百分比
O old代已使用的占当前容量百分比
M metaspace已使用的占当前容量百分比
CCS 压缩使用比例
YGC 从应用程序启动到采样时年轻代中gc次数
YGCT 从应用程序启动到采样时年轻代中gc所用时间(s)
FGC 从应用程序启动到采样时old代(全gc)gc次数
FGCT 从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT 从应用程序启动到采样时gc用的总时间(s)
```

### jinfo

jinfo：Java Configuration Info

jinfo可以用来查看正在运行的Java程序的扩展参数，甚至支持修改运行过程中的部分参数。

```
jinfo [option] <pid>
```

```
-flags 打印虚拟机 VM 参数
-flag <name> 打印指定虚拟机 VM 参数
-flag [+|-]<name> 打开或关闭虚拟机参数
-flag <name>=<value> 设置指定虚拟机参数的值
```

### jmap

jmap:Memory Map

jmap用来查看堆内存使用状况，一般结合jhat使用。

**示例一：jmap pid**

```
jmap 1850
```

查看进程的内存映像信息。使用不带选项参数的jmap打印共享对象映射，将会打印目标虚拟机中加载的每个共享对象的起始地址、映射大小以及共享对象文件的路径全称。

![jmap pid](.\note\jmap pid.jpg)

**示例二：jmap -heap pid**

```
jmap -heap 1850
```

显示Java堆详细信息：打印堆的摘要信息，包括使用的GC算法、堆配置信息和各内存区域内存使用信息

![jmap -heap pid](.\note\jmap -heap pid.jpg)

**示例三：jmap -histo:live pid**

```
jmap -histo:live 1850
```

显示堆中对象的统计信息：其中包括每个Java类、对象数量、内存大小(单位：字节)、完全限定的类名。打印的虚拟机内部的类名称将会带有一个’*’前缀。如果ja指定了live子选项，则只计算活动的对象。

![jmap -histolive pid](.\note\jmap -histolive pid.jpg)

**示例四：jmap -clstats pid**

打印类加载器信息：打印Java堆内存的方法区的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

**示例五：jmap -finalizerinfo pid**

打印等待终结的对象信息

**示例六：jmap -dump:format=b,file=heapdump.hprof pid**

生成堆转储快照dump文件：以二进制格式转储Java堆到指定文件中。如果指定了live子选项，堆中只有活动的对象会被转储。浏览heap dump 可以使用jhat 读取生成的文件，也可以使用MAT等堆内存分析工具。

注意：这个命令执行，JVM会将整个heap的信息dump写入到一个文件，heap如果比较大的话，就会导致这个过程比较耗时，并且执行的过程中为了保证dump的信息是可靠的，所以会暂停应用， 线上系统慎用！

### jhat

jhat:Java Heap Analysis Tool

- jhat 命令会解析Java堆转储文件，并启动一个 web server。然后用浏览器来查看/浏览 dump 出来的 heap二进制文件。
- jhat 命令支持预先设计的查询，比如：显示某个类的所有实例。还支持 对象查询语言（OQL）。 OQL有点类似SQL，专门用来查询堆转储。

**Java生成堆转储的方式有多种:**

1. 使用 jmap -dump 选项可以在JVM运行时获取 dump.
2. 使用 jconsole 选项通过 HotSpotDiagnosticMXBean 从运行时获得堆转储。
3. 在虚拟机启动时如果指定了 -XX:+HeapDumpOnOutOfMemoryError 选项，则抛出 OutOfMemoryError 时，会自动执行堆转储。

```
jhat [ options ] heap-dump-file
```

jhat 启动后显示的 html 页面中包含有:

- All classes including platform:显示出堆中所包含的所有的类
- Show all members of the rootset :从根集能引用到的对象
- Show instance counts for all classes (including platform/excluding platform):显示平台包括的所有类的实例数量
- Show heap histogram:堆实例的分布表
- Show finalizer summary:Finalizer 摘要
- Execute Object Query Language (OQL) query:执行对象查询语句（OQL）

### jstack

jstack：Java Stack Trace

jstack是Java虚拟机自带的一种堆栈跟踪工具，用于生成java虚拟机当前时刻的线程快照。

线程快照是当前Java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出**现长时间停顿的原因**，如线程间死锁、死循环、请求外部资源导致的长时间等待、等等。


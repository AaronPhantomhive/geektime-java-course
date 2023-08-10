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


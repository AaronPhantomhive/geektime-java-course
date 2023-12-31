# JVM 虚拟机

JVM广义上指的是一种规范。狭义上的是JDK中的JVM虚拟机。

##  **JVM**虚拟机概述

### 基本常识

平时我们所说的JVM广义上指的是一种规范。狭义上的是JDK中的JVM虚拟机。JVM的实现是由各个厂商来做的。比如现在流传最广泛的是hotspot。其他实现：BEA公司 JRocket、IBM j9、zing 号称世界最快JVM、taobao.vm。从广义上讲Java，Kotlin、Clojure、JRuby、Groovy等运行于Java虚拟机上的编程语言及其相关的程序都属于Java技术体系中的一员。

#### Java技术体系的四个方面

- Java程序设计语言

- Java类库API

- 来自商业机构和开源社区的第三方Java类库
  - Google
  - Apache

- Java虚拟机：各种硬件平台上的Java虚拟机实现

![Java](.\note\Java.jpg)

#### JVM架构图

![JVM架构图](.\note\JVM架构图.jpg)

#### Java和JVM的关系

![Java和JVM的关系](.\note\Java和JVM的关系.jpg)

### 类加载子系统

#### 类加载器

![类加载器](.\note\类加载器.png)

- 启动类加载器(Bootstrap ClassLoader)：
  - 负责加载 JAVA_HOME\lib 目录中的，或通过-Xbootclasspath参数指定路径中的，且被虚拟机认可（按文件名识别，如rt.jar）的类。由C++实现，不是ClassLoader的子类
- 扩展类加载器(Extension ClassLoader)：
  - 负责加载 JAVA_HOME\lib\ext 目录中的，或通过java.ext.dirs系统变量指定路径中的类库。
- 应用程序类加载器(Application ClassLoader)：
  - 负责加载用户路径classpath上的类库

- 自定义类加载器（User ClassLoader）：
  - 加载应用之外的类文件
  - 作用：JVM自带的三个加载器只能加载指定路径下的类字节码，如果某些情况下，我们需要加载应用程序之外的类文件呢？就需要用到自定义类加载器，就像是在汽车行驶的时候，为汽车更换轮子。
  - 比如本地D盘下的，或者去加载网络上的某个类文件，这种情况就可以使用自定义加载器了。
  - 举个栗子：JRebel

#### 加载顺序

自顶向下尝试加载，自底向上检查是否已加载。

![加载顺序](.\note\加载顺序.png)

- **检查顺序是自底向上**：加载过程中会先检查类是否被已加载，从Custom ClassLoader到BootStrapClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。
- **加载的顺序是自顶向下**：也就是由上层来逐层尝试加载此类。

#### 类加载的时机

- 四个时机
  - 遇到 new 、 getstatic 、 putstatic 和 invokestatic 这四条指令时，如果对应的类没有初始化，则要对对应的类先进行初始化。
  - 使用 java.lang.reflect 包方法时，对类进行**反射调用**的时候。
  - 初始化一个类的时候发现其父类还没初始化，要先初始化其父类
  - 当虚拟机开始启动时，用户需要指定一个主类（main），虚拟机会先执行这个主类的初始化。

#### 类加载的过程

![类的生命周期](.\note\类的生命周期.png)

类加载主要做三件事：

- 全限定名称 → 二进制字节流加载class文件

- 字节流的静态数据结构 → 方法区的运行时数据结构（永久代，元空间）
- 创建字节码Class对象（成员变量，方法等）

#### 类加载途径

加载途径：

- jar/war
- jsp生成的class
- 数据库中的二进制字节流
- 网络中的二进制字节流
- 动态代理生成的二进制字节流

### 双亲委派

**当一个类加载器收到类加载任务，会先交给其父类加载器去完成**。因此，最终加载任务都会传递到顶层的启动类加载器，**只有当父类加载器无法完成加载任务时，子类才会尝试执行加载任务**。

#### Oracle 官网文档描述

> **The Java Class Loading Mechanism**
>
> The Java platform uses a delegation model for loading classes. The basic idea is that everyclass loader has a "parent" class loader. When loading a class, a class loader first "delegates"the search for the class to its parent class loader before attempting to find the class itself.
>
> —— Oracel Document
>
> https://docs.oracle.com/javase/tutorial/ext/basics/load.html

### 需要双亲委派的原因

- 考虑到安全因素，双亲委派可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。
- 比如：加载位于rt.jar包中的类java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个Object对象。

### 双亲委派机制源码

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
     	// 首先，检查这类是否已经被加载过了
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // 如果存在父类加载器，则取找该类的父类加载器
                    c = parent.loadClass(name, false);
                } else {
                    // 返回由引导类加载器加载的类；如果未找到，则返回 null。
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                 // 如果父类加载器抛出ClassNotFoundException异常
                 // 则说明父类加载器无法完成加载请求
            }

            if (c == null) {
                 // 在父类加载器无法加载时
                 // 再调用本身的findClass方法来进行加载
                long t1 = System.nanoTime();
                c = findClass(name);

                 // s这是定义类加载器；记录统计数据
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

### 破坏双亲委派

#### 原因

- 在实际应用中，双亲委派解决了Java 基础类统一加载的问题，但是却存在着缺陷。JDK中的基础类作为典型的api被用户调用，但是也存在api调用用户代码的情况，典型的如：SPI代码。这种情况就需要打破双亲委派模式。

- 举个栗子：数据库驱动DriverManager。以Driver接口为例，Driver接口定义在JDK中，其实现由各个数据库的服务商来提供，由系统类加载器加载。这个时候就需要启动类加载器来委托子类来加载Driver实现，这就破坏了双亲委派。类似情况还有很多

#### 方式

**第一种方式**

- 在 jdk 1.2 之前，那时候还没有双亲委派模型，不过已经有了 ClassLoader 这个抽象类，所以已经有人继承这个抽象类，**重写** **loadClass** 方法来**实现用户自定义类加载器**。
- 而在 1.2 的时候要**引入双亲委派模型**，为了向前兼容， loadClass 这个方法还得保留着使之得以重写，新搞了个 findClass 方法让用户去重写，并呼吁大家不要重写 loadClass 只要重写 findClass。
- 这就是第一次对双亲委派模型的破坏，因为双亲委派的逻辑在loadClass上，但是又允许重写loadClass，重写了之后就可以破坏委派逻辑了。

**第二种方式：**

- 双亲委派机制是一种**自上而下的加载需求，越往上类越基础**。

- SPI代码打破了双亲委派

![破坏双亲委派](.\note\破坏双亲委派.png)

### 运行时数据区(内存区域)

**作业问题：请你用自己的语言介绍 Java 运行时数据区（内存区域）中的堆、虚拟机栈、本地方法栈、方法区（永久代、元空间）、运行时常量池（字符串常量池）、直接内存**

```
堆（Heap）：JVM启动时创建，用于存放对象实例、数组和运行时常量的内存分配，也是垃圾回收的主要内存区域。当堆中没有足够内存来创建对象时，会抛出OOM（OutOfMemoryError）异常。
JVM对堆进行了分代管理，分为新生代（Young Generation）和老年代（Old Generation）。

虚拟机栈（Java VM Stack）：虚拟机栈是用于存储和管理Java方法执行过程中的内存模型。每个方法调用和执行都会创建一个栈帧（Stack Frame）被称为当前栈帧，并且进行入栈和出栈。栈帧存储了方法的局部变量表（Local Variable Table）、操作数栈（Operand Stack）、动态连接（Dynamic Linking）和返回地址（Return Address）等信息。线程请求的栈深度大于-Xss虚拟机深度时，会抛出StackOverflowError异常。
栈是运行时的单位，堆是存储的单位。

本地方法栈（Native Method Stack）：用于储存和管理本地方法（Native Method）的调用和执行。本地方法是指非Java语言编写的方法，因此可以理解为一个本地方法（Native Method）就是一个Java调用非Java代码的接口。

方法区（Method Area）：JVM启动时创建，用于存储编译器已经编译后的类信息，运行时常量池，类型信息，字段信息，方法信息，类变量，静态变量，成员变量，方法数据，构造方法，字节码指令等。JDK7之前，被称为永久代（PermGen）。JDK8及之后，元空间（Metaspace）取代了永久代。永久代使用的是JVM进程使用的内存，而元空间使用的内存区域是物理内存区域。元空间改成只存储类的元信息，把静态变量和运行时常量池挪到了堆中。

运行时常量池（字符串常量池）（Runtime Constant Pool）：为了提高匹配速度，加快数据查找的速度，JVM启动时创建的一个全局存储字符串的常量池，存放的是字符串常量值。实现方式为StringTable，类似于哈希表的key value查找，根据字符串的hashcode找到其对应的entry并返回引用。

直接内存：为了提高I/O性能，提高读写速度，绕过Java虚拟机的内存空间。是直接向本地IO进行申请的内存，而不受虚拟机的管理。
```

#### 概念

整个JVM构成里面，由三部分组成：类加载系统、**运行时数据区**、执行引擎

![运行时数据区](.\note\运行时数据区.png)

按照线程使用情况和职责分成两大类

- 线程独享 （程序执行区域）
  - 不需要垃圾回收
  - 虚拟机栈、本地方法栈、程序计数器
- 线程共享 （数据存储区域）
  - 垃圾回收
  - 存储类的静态数据和对象数据
  - 堆和方法区

#### 堆

Java堆在JVM启动时创建内存区域去实现对象、数组与运行时常量的内存分配，它是虚拟机管理最大的，也是垃圾回收的主要内存区域 。

内存划分：

**核心逻辑就是三大假说，基于程序运行情况进行不断的优化设计。**

##### 堆内存为什么会存在新生代和老年代？

![堆内存的新生代和老年代](.\note\堆内存的新生代和老年代.png)

分代收集理论：当前商业虚拟机的垃圾收集器，大多数都遵循了“分代收集”（Generational
Collection）的理论进行设计，分代收集名为理论，实质是一套符合大多数程序运行实际情况的经验法则，它建立在两个分代假说之上：

- 弱分代假说（Weak Generational Hypothesis）：绝大多数对象都是朝生夕灭的。
- 强分代假说（Strong Generational Hypothesis）：熬过越多次垃圾收集过程的对象就越难以消亡。

这两个分代假说共同奠定了多款常用的垃圾收集器的一致的设计原则：收集器应该将Java堆划分出不同的区域，然后将回收对象依据其年龄（年龄即对象熬过垃圾收集过程的次数）分配到不同的区域之中存储。

- 如果一个区域中大多数对象都是朝生夕灭，难以熬过垃圾收集过程的话，那么把它们集中放在一起，每次回收时只关注如何保留少量存活而不是去标记那些大量将要被回收的对象，就能以较低代价回收到大量的空间；
- 如果剩下的都是难以消亡的对象，那把它们集中放在一块，虚拟机便可以使用较低的频率来回收这个区域。

这就同时兼顾了垃圾收集的时间开销和内存的空间有效利用。

**作业问题：为什么堆内存要分年轻代和老年代？**

回答：

```
主要目的是为了使JVM可以更好的管理和分配堆内存中的对象，提高程序的性能和效率，降低垃圾收集成本。

新生代约占堆内存的1/5，被划分为3个区域，Eden、Survivor From 和 Survivor To，其比例为8：1：1。新创建的对象会放在新生代的Eden里，当Eden区被占满时，会触发一次新生代的垃圾回收（MinorGC）。此时存活的对象会被记录其年龄并放到SurvivorFrom区，当Eden再次被占满触发垃圾回收（MinorGC）时，会扫描Eden和From区并进行垃圾回收（MinorGC），并将存活的对象放入SurvivorTo区。最后From区和To区互换，即谁空谁是To。如此互换15次（默认的MaxTenuringThreshold）后仍然存活的对象进入老年代。

而老年代是用来存放生命周期较长的对象的，约占堆内存的4/5。当一个对象在新生代经历过多次垃圾回收（MinorGC）时仍然存活，或对象由于过大（默认的PretenureSizeThreshold，1M）而无法放入新生代时，就会被放入老年代。而当老年代区被占满或由于对象过大放入不进新生代而放入老年代时，也同样会触发老年代的垃圾回收（MajorGC）。
```

**内存模型变迁：**

![JDK1.7](.\note\JDK1.7.png)

- JDK1.7
  - Young 年轻区 ：主要保存年轻对象，分为三部分，Eden区、两个Survivor区。
  - Tenured 年老区 ：主要保存年长对象，当对象在Young复制转移一定的次数后，对象就会被转移到Tenured区。
  - Perm 永久区 ：主要保存class、method、filed对象，这部份的空间一般不会溢出，除非一次性加载了很多的类，不过在涉及到热部署的应用服务器的时候，有时候会遇到OOM :PermGen space 的错误。
  - Virtual区： 最大内存和初始内存的差值，就是Virtual区

![JDK1.8](.\note\JDK1.8.png)

- JDK1.8
  - 由2部分组成，新生代（Eden + 2*Survivor ） + 年老代（OldGen )
  - JDK1.8中变化最大是，的Perm永久区用Metaspace进行了替换
  - 注意：Metaspace所占用的内存空间不是在虚拟机内部，而是在本地内存空间中。区别于JDK1.7

![JDK1.9](.\note\JDK1.9.png)

- JDK1.9
  - 取消新生代、老年代的物理划分
  - 将堆划分为若干个区域（Region），这些区域中包含了有逻辑上的新生代、老年代区域

#### 虚拟机栈

- 栈帧是什么？
  - 栈帧(Stack Frame)是用于支持虚拟机进行方法执行的数据结构。
  - 栈帧存储了方法的**局部变量表、操作数栈、动态连接和方法返回地址**等信息。每一个方法从调用至执行完成的过程，都对应着一个栈帧在虚拟机栈里从入栈到出栈的过程。
  - 栈内存为线程私有的空间，每个线程都会创建私有的栈内存，生命周期与线程相同，每个Java方法在执行的时候都会创建一个**栈帧（Stack Frame）**。栈内存大小决定了方法调用的深度，栈内存过小则会导致方法调用的深度较小，如递归调用的次数较少。

- **当前栈帧**
  - 一个线程中方法的调用链可能会很长，所以会有很多栈帧。只有位于JVM虚拟机栈栈顶的元素才是有效的，即称为当前栈帧，与这个栈帧相关连的方法称为当前方法，定义这个方法的类叫做当前类。
  - 执行引擎运行的所有字节码指令都只针对当前栈帧进行操作。如果当前方法调用了其他方法，或者当前方法执行结束，那这个方法的栈帧就不再是当前栈帧了。
- **什么时候创建栈帧**
  - 调用新的方法时，新的栈帧也会随之创建。并且随着程序控制权转移到新方法，新的栈帧成为了当前栈帧。方法返回之际，原栈帧会返回方法的执行结果给之前的栈帧(返回给方法调用者)，随后虚拟机将会丢弃此栈帧。
- **栈异常的两种情况**
  - 如果线程请求的栈深度大于虚拟机所允许的深度（Xss默认1m），会抛出StackOverflowError异常
  - 如果在创建新的线程时，没有足够的内存去创建对应的虚拟机栈，会抛出OutOfMemoryError异常【不一定】

#### 本地方法栈

**本地方法栈**和**虚拟机栈**相似，区别就是虚拟机栈为虚拟机执行**Java服务（字节码服务）**，而本地方法栈为虚拟机使用到的**Native方法（比如C++方法）服务**。

简单地讲，一个Native Method就是一个Java调用非Java代码的接口。

**为什么需要本地方法？**

Java是一门高级语言，不直接与操作系统资源、系统硬件打交道。如果想要直接与操作系统与硬件打交道，就需要使用到本地方法了。说白了，Java可以直接通过native方法调用cpp编写的接口。多线程底层就是这么实现的。

#### 方法区（永久代、元空间）

方法区（Method Area）是可供各个线程共享的运行时内存区域，方法区本质上是Java语言**编译后代码存储区域**，它存储每一个类的结构信息，例如：**运行时常量池**、成员变量、方法数据、构造方法和普通方法的字节码指令等内容。很多语言都有类似区域。

方法区的具体实现有两种：**永久代（PermGen）**、**元空间（Metaspace）**

**方法区存储什么数据**

![方法区存储数据](.\note\方法区存储数据.png)

主要有如下三种类型

- 第一：Class
  - 类型信息，比如Class（com.hero.User类）
  - 方法信息，比如Method（方法名称、方法参数列表、方法返回值信息）
  - 字段信息，比如Field（字段类型，字段名称需要特殊设置才能保存的住）
  - 类变量（静态变量）：JDK1.7之后，转移到堆中存储
  - 方法表（方法调用的时候） 在A类的main方法中去调用B类的method1方法，是根据B类的方法表去查找合适的方法，进行调用的。
- 第二：运行时常量池（字符串常量池）：从class中的常量池加载而来，JDK1.7之后，转移到堆中存储
  - 字面量类型
  - 引用类型 → 内存地址
- 第三：JIT编译器编译之后的代码缓存

##### **永久代和元空间的区别是什么**

JDK1.8之前使用的方法区实现是**永久代（PermGen）**，JDK1.8及以后使用的方法区实现是**元空间（Metaspace）**。

1. JDK1.8之前使用的方法区实现是**永久代**，JDK1.8及以后使用的方法区实现是**元空间**。

2. **存储位置不同：**
   - **永久代**所使用的内存区域是**JVM进程所使用的区域**，它的大小受整个JVM的大小所限制。
   - **元空间**所使用的内存区域是物理内存区域。那么元空间的使用大小只会受物理内存大小的限制。

3. **存储内容不同：**
   - 永久代存储的信息基本上就是上面方法区存储内容中的数据。
   - 元空间只存储类的元信息，而**静态变量和运行时常量池都挪到堆中**。

##### 为什么要使用元空间来替换永久代？

1. **字符串存在永久代中，容易出现性能问题和永久代内存溢出。**

2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。

3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

4. Oracle 计划将HotSpot 与 JRockit 合二为一。

##### 方法区实现变迁历史

![方法区实现变迁历史](.\note\方法区实现变迁历史.png)

#### 字符串常量池

##### 三种常量池的比较

- class常量池：一个class文件只有一个class常量池
  - 字面量：数值型（int、float、long、double）、双引号引起来的字符串值等
  - 符号引用：Class、Method、Field等
- 运行时常量池：一个class对象有一个运行时常量池
  - 字面量：数值型（int、float、long、double）、双引号引起来的字符串值等
  - 符号引用：Class、Method、Field等
- 字符串常量池：全局只有一个字符串常量池
  - 双引号引起来的字符串值

##### 字符串常量池如何存储数据

为了提高匹配速度， 即更快的查找某个字符串是否存在于常量池 Java 在设计字符串常量池的时候，还搞了一张StringTable， StringTable里面保存了字符串的引用。StringTable类似于HashTable（哈希表）。在JDK1.7+，StringTable可以通过参数指定 -XX:StringTableSize=99991

**哈希表**

哈希表（Hash table，也叫散列表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

- **哈希表本质上是一个数组+链表**
- key：散列函数，公式：hash（字符串）%数组size
- value：字符串的引用
- size：-XX:StringTableSize=65536

##### 字符串常量池如何查找字符串

- 根据字符串的hashcode找到对应entry
- 如果没有冲突，它可能只是一个entry
- 如何有冲突，它可能是一个entry的链表，然后Java再遍历链表，匹配引用对应的字符串
- 如果找到字符串，返回引用
- 如果找不到字符串，在使用intern()方法的时候，会将intern()方法调用者的引用放入到stringtable中

总结：

- 单独使用””引号创建的字符串都是常量，编译期就已经确定存储到String Pool中。
- 使用new String(“”)创建的对象会存储到heap中，是运行期新创建的。
- 使用只包含常量的字符串连接符如”aa”+”bb”创建的也是常量，编译期就能确定已经存储到StringPool中。
- 使用包含变量的字符串连接如”aa”+s创建的对象是运行期才创建的，存储到heap中。
- 运行期调用String的intern()方法可以向String Pool中动态添加对象。

#### 程序计数器

**程序计数器（Program Counter Register）**，也叫**PC寄存器**，是一块较小的内存空间，它可以看作是**当前线程所执行的字节码指令的行号指示器**。字节码解释器的工作就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。**分支，循环，跳转，异常处理，线程回复等都需要依赖这个计数器来完成。**

##### 为什么需要程序计数器

由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（针对多核处理器来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换（**系统上下文切换**）后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

##### 存储的什么数据

如果一个线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是一个Native方法，这个计数器的值则为空。

**异常：**此内存区域是唯一一个在Java的虚拟机规范中没有规定任何OutOfMemoryError异常情况的区域。

#### 直接内存

直接内存并不是虚拟机运行时数据区的一部分，也不是Java 虚拟机规范中定义的内存区域。在JDK1.4 中新加入了NIO(New Input/Output)类，引入了一种基于通道(Channel)与缓冲区（Buffer）的I/O 方式，它可以使用native 函数库直接分配堆外内存，然后通过一个存储在Java堆中的**DirectByteBuffer** 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

本机直接内存的分配不会受到Java 堆大小的限制，受到本机总内存大小限制。

##### 直接内存（堆外内存）与堆内存比较

- 直接内存申请空间耗费更高的性能，当频繁申请到一定量时尤为明显
- 直接内存IO读写的性能要优于普通的堆内存，在多次读写操作的情况下差异明显

从数据流的角度，来看

- 非直接内存作用链：本地IO –>直接内存–>非直接内存–>直接内存–>本地IO
- 直接内存作用链：本地IO–>直接内存–>本地IO

##### 直接内存的使用场景

- 有很大的数据需要存储，它的生命周期很长
- 适合频繁的IO操作，例如：网络并发场景

### 对象的创建流程与内存分配

作业问题：描述一个 Java 对象的生命周期

- 解释一个对象的创建过程

  ```
  当执行new指令的时候，JVM会先检查该对象的类是否已经被加载，如果没有，则会先加载该类的字节码对象到内存中，并为该对象分配一块内存空间。初始化内存空间的分配方式有两种：指针碰撞和空闲列表。之后为对象的属性进行初始化，设置元数据哈希码GC分代年龄等信息。
  ```

- 解释一个对象的内存分配

  ```
  初始化内存空间的分配方式有两种：指针碰撞和空闲列表。指针碰撞多以新生代为主，内存地址是连续的。空闲列表多以老年代为主，内存地址不连续，回收时采用标记整理算法。当分配内存遇到分配冲突时，会先从本地线程分配缓存（TLAB）（Thread Local Allocation Buffer）里分配，如果不能被分配到，则再使用CAS（Compare and Swap）乐观锁。
  ```

- 解释一个对象的销毁过程

  ```
  对象销毁是由垃圾回收机制完成的，使用引用计数法，根搜索算法等找到标记并回收内存中不再被引用的对象。如果一个对象没有任何变量指向它，或所属的引用链不可达，则会被清除。新生代通常采用复制算法，老年代使用标记清除、标记整理算法来实现垃圾回收。
  ```

- 对象的 2 种访问方式是什么？

  ```
  句柄和直接指针。
  句柄为从句柄池中找到对象对应实例数据的指针，从而找到实例池中对象的数据。优点为比较文档，可以直接修改句柄中的地址来实现对象的移动和修改。
  直接指针是直接在本地变量表中找到对象对应的指针，好处为访问速度快，不需要再从实例池中再进行一次查找，节省指针的开销。
  ```

- 为什么需要内存担保？

  ```
  在当新生代把对象转移到老年代时的机制，被称为内存担保。是为了避免在新生代分配内存时发生内存不足的情况，保证新生代的内存分配不会失败。
  ```

![对象的创建流程与内存分配](.\note\对象的创建流程与内存分配.png)

#### 对象内存分配方式

内存分配的方法有两种：不同垃圾收集器不一样

- 指针碰撞(Bump the Pointer)
- 空闲列表(Free List)

| 分配方法                   | 说明                       | 收集器                         |
| -------------------------- | -------------------------- | ------------------------------ |
| 指针碰撞(Bump the Pointer) | 内存地址是连续的（新生代） | Serial 和 ParNew 收集器        |
| 空闲列表(Free List)        | 内存地址不连续（老年代）   | CMS 收集器和 Mark-Sweep 收集器 |

#### 内存分配安全问题

在分配内存的时候，虚拟机给A线程分配内存过程中，指针未修改。此时B线程同时使用了同样一块内存，此时可能出现线程的安全性问题

在JVM中有两种解决办法：

1. CAS 是乐观锁的一种实现方式。虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。
2. TLAB本地线程分配缓冲(Thread Local Allocation Buffer即TLAB)：为每一个线程预先分配一块内存

JVM在第一次给线程中的对象分配内存时，首先使用CAS进行TLAB的分配。当对象大于TLAB中的剩余内存或TLAB的内存已用尽时，再采用上述的CAS进行内存分配。

#### 对象怎样才会进入老年代

对象内存分配：

- 新生代：新对象大多数都默认进入新生代的Eden区

- 进入老年代的条件：四种情况
  - 存活年龄太大，默认超过15次【-XX:MaxTenuringThreshold】
  - 动态年龄判断：  MinorGC之后，发现Survivor区中的一批对象的总大小大于了这块Survivor区 的50%，那么就会将此时大于等于这批对象年龄最大值的所有对象，直接进入老年代。
    - 举个栗子：  Survivor区中有一批对象， 年龄分别为年龄1+年龄2+年龄n的多个对象，对 象总和大小超过了Survivor区域的50%，此时就会把年龄n及以上的对象都放入老年代。
    - 为什么会这样？ 希望那些可能是长期存活的对象，尽早进入老年代。
    - -XX:TargetSurvivorRatio可以指定
  - 大对象直接进入老年代：  前提是Serial和ParNew收集器
    - 举个栗子：字符串或数组
    - -XX:PretenureSizeThreshold 一般设置为1M
    - 为什么会这样？ 为了避免大对象分配内存时的复制操作降低效率。避免了Eden和 Survivor区的复制
    - MinorGC后，存活对象太多无法放入Survivor

空间担保机制：当新生代无法分配内存的时候，我们想把新生代的老对象转移到老年代，然后把新对象 放入腾空的新生代。此种机制我们称之为内存担保。

- MinorGC前，判断老年代可用内存是否小于新时代对象全部对象大小，如果小于则继续判断
- 判断老年代可用内存大小是否小于之前每次MinorGC后进入老年代的对象平均大小
  - 如果是，则会进行一次FullGC，判断是否放得下，放不下OOM
  - 如果否，则会进行一些MinorGC：
    - MinorGC后，剩余存活对象小于Survivor区大小，直接进入Survivor区
    - MinorGC后，剩余存活对象大于Survivor区大小，但是小于老年代可用内存，直接进入老年代
    - MinorGC后，剩余存活对象大于Survivor区大小，也大于老年代可用内存，进行FullGC
    - FullGC之后，任然没有足够内存存放MinorGC的剩余对象，就会OOM

![老年代的担保示意图](.\note\老年代的担保示意图.png)

### 对象里的三个区

堆内存中，一个对象在内存中存储的布局可以分为三块区域：

1. **对象头（Header）：**Java对象头占8byte。如果是数组则占12byte。因为JVM里数组size需要使用4byte存储。
   - **标记字段MarkWord**：
     - 用于存储对象自身的运行时数据，它是synchronized实现轻量级锁和偏向锁的关键。
     - 默认存储：对象HashCode、GC分代年龄、锁状态等等信息。
     - 为了节省空间，也会随着锁标志位的变化，存储数据发生变化。
   - **类型指针KlassPoint：**
     - 是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例
     - 开启指针压缩存储空间4byte，不开启8byte。
     - JDK1.6+默认开启
   - **数组长度：**如果对象是数组，则记录数组长度，占4个byte，如果对象不是数组则不存在。
   - **对齐填充：**保证数组的大小永远是8byte的整数倍。

2. **实例数据（Instance Data）：**生成对象的时候，对象的非静态成员变量也会存入堆空间

3. **对齐填充（Padding）：**JVM内对象都采用8byte对齐，不够8byte的会自动补齐。

![对象里的三个区](.\note\对象里的三个区.png)

### 如何访问一个对象

**句柄和直接指针。**

句柄为从句柄池中找到对象对应实例数据的指针，从而找到实例池中对象的数据。优点为比较文档，可以直接修改句柄中的地址来实现对象的移动和修改。

直接指针是直接在本地变量表中找到对象对应的指针，好处为访问速度快，不需要再从实例池中再进行一次查找，节省指针的开销。

![句柄](.\note\句柄.png)

![直接指针](.\note\直接指针.png)

## JVM垃圾收集器

### GC基本原理

#### 垃圾回收

##### 引用计数法（Reference Counting）

当这个对象引用都消失了，消失一个计数减一，当引用都消失了，计数就会变为0。此时这个对象就会变成垃圾。

##### 根可达算法（GCRoots Tracing）

又叫根搜索算法。在主流的商用程序语言中（Java和C#），都是使用根搜索算法判定对象是否存活的。

基本思路就是通过一系列的名为“GCRoot”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GCRoot没有任何引用链相连时，则证明此对象是不可用的，也就是不可达的。

**可作GCRoots的对象：**

- 虚拟机栈中，栈帧的本地变量表引用的对象。
- 方法区中，类静态属性引用的对象。
- 方法区中，常量引用的对象。
- 本地方法栈中，JNl引用的对象。

##### 回收过程

即使在可达性分析算法中不可达的对象，也并非是“非死不可”。被判定不可达的对象处于“缓刑”阶段。

要真正宣告死亡，至少要经历两次标记过程：

- 第一次标记：如果对象可达性分析后，发现没有与GC Roots相连接的引用链，那它将会被第一次标记；
- 第二次标记：第一次标记后，接着会进行一次筛选。筛选条件：此对象是否有必要执行finalize() 方法。在 finalize() 方法中没有重新与引用链建立关联关系的，将被进行第二次标记。

第二次标记成功的对象将真的会被回收，如果失败则继续存活

#### 清除垃圾

作业问题：垃圾收集算法有哪些？垃圾收集器有哪些？他们的特点是什么？

```
垃圾收集算法有三种：
标记清除算法（Mark-Sweep），
复制算法（Copying），
标记整理算法（Mark-Compact）。

标记清除算法分为两个步骤，首先标记出所有需要回收的对象，然后清除掉这些对象所占的空间。缺点是效率不够高，且会产生大量不连续的内存碎片。

复制算法是将内存分为两块相等的区域，每次都只使用其中一块。当一块被占满时，把还存活的对象复制到另一块上，然后清除掉原来的区域。此算法比较适用于年轻代。优点是效率会大幅度提升且不会内存空间碎片化，缺点是内存利用率很低存在空间浪费。

标记整理算法是对标记清除算法的改进，在标记出需要回收的对象后将所有的存活对象向一端移动。优点是没有空间浪费，没有内存碎片化的问题。缺点是移动对象开销很大，性能很低。比较适用于老年代。

垃圾回收器有8种，分别用于不同分代的垃圾回收。
新生代回收器：Serial、ParNew、Parallel Scavenge
老年代回收器：Serial Old、Parallel Old、CMS
整堆回收器：G1、ZGC
```

- ParallelScavenge 收集器

  ```
  新生代使用并行收集器，采用复制算法。老年代使用串行收集器，使用标记整理算法。主要关注系统吞吐量，即 运行用户代码时间/(运行用户代码时间+运行垃圾收集时间) 越趋近1意味着吞吐量越好越大，垃圾收集需要暂停用户线程。
  ```

- ParallelOld 收集器

  ```
  老年代的并行收集器，ParallelScavenge的老年代版本。同样吞吐量优先，使用多线程进行标记整理算法。垃圾收集同样需要暂停用户线程，对cpu敏感。
  ```

- ParNew 收集器

  ```
  Serial的多线程版本，单核cpu不建议使用，使用复制算法。响应优先，即减少用户线程停顿时间。需要指定垃圾收集的线程数。新生代使用并行ParNew，老年代使用串行SerialOld。
  ```

- CMS（Concurrent Mark Sweep GC）收集器

  ```
  主要用在老年代并发收集器，使用标记清除算法，同样响应优先。低延时，减少STW对用户的影响。用户线程与收集线程一起执行，cpu敏感。
  ```

- G1（Garbage First）收集器

  ```
  全新的面向服务端应用的全功能型的垃圾收集器。吞吐量和低延时皆可行的整堆垃圾收集器。适用于新生代和老年代。全局使用多线程进行标记整理算法，将内存空间划分为多个小块，称为区域（Region），在逻辑上有Eden、Survivor、Old、Humongous（当对象的容量超过Region的50%，则被认为是巨型对象）。局部采用复制算法收集。
  ```


- Mark-Sweep 标记清除算法
  - 最基本的算法，主要分为**标记**和**清除**2个阶段。首先**标记出所有需要回收的对象**，在**标记完成后统一回收掉所有被标记的对象**
  - 缺点: 
    - **效率不高**，**标记和清除**过程的效率都不高
    - **空间碎片**，会产生大量不连续的内存碎片，会导致大对象可能无法分配，提前触发GC 。
- Copying 拷贝算法
  - **为解决效率。**它将可用内存按容量划分为相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。
  - **现在商业虚拟机都是采用这种收集算法来回收新生代**，当回收时，将Eden和Survivor中还存活着的对象拷贝到另外一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor的空间。
  - HotSpot虚拟机默认Eden和Survivor的大小比例是8：1，也就是每次新生代中可用内存空间为整个新生代容量的90%（80%+10%），只有10%的内存是会被“浪费”的。当Survivor空间不够用时，需要依赖其他内存（这里指老年代）进行**分配担保（Handle Promotion）**。
  - 优点：没有碎片化，所有的有用的空间都连接在一起，所有的空闲空间都连接在一起
  - 缺点：存在空间浪费
- Mark-Compact 标记压缩算法
  - **老年代没有人担保，不能用复制回收算法**。可以用**标记整理**算法，标记过程仍然与“标记-清除”算法一样，然后让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存
  - 缺点：性能较低，因为除了拷贝对象以外，还需要对象内存空间进行压缩，所以性能较低

#### 垃圾回收器

- 新生代回收器：Serial、ParNew、Parallel Scavenge
- 老年代回收器：Serial Old、Parallel Old、CMS
- 整堆回收器：G1、ZGC

两个垃圾回收器之间有连线表示它们可以搭配使用，可选的搭配方案如下

![垃圾回收器](.\note\垃圾回收器.png)

| **新生代**        | **老年代**   |
| ----------------- | ------------ |
| Serial            | Serial Old   |
| Serial            | CMS          |
| ParNew            | Serial Old   |
| ParNew            | CMS          |
| Parallel Scavenge | Serial Old   |
| Parallel Scavenge | Parallel Old |

### 串行收集器

#### 基本概念

使用单线程进行垃圾回收的收集器，每次回收时，串行收集器只有一个工作线程，对于并行能力较弱的计算机来说，串行收集器性能会更好。

串行收集器可以在新生代和老年代中使用，根据作用于不同的堆空间，分为新生代串行收集器和老年代收集器。

配置参数 -XX:+UseSerialGC ：年轻串行（Serial），老年串行（Serial Old）

**Serial收集器：年轻串行**

Serial收集器是新生代收集器，单线程执行，使用复制算法。

进行垃圾收集时，必须暂停其他所有的工作线程。

对于单个CPU的环境来说，Serial收集器由于没有线程交互的开销，收集效率更高

**Serial Old收集器：老年串行**

Serial收集器的老年代版本，它同样是一个单线程收集器，使用“标记-整理”算法。

### 并行收集器

#### Parallel Scavenge收集器

配置参数： -XX:+UseParallelGC

目标是达到一个可控制的吞吐量（Throughput）

特点：

- 吞吐量优先收集器
- 新生代使用并行回收收集器，采用复制算法
- 老年代使用串行收集器

```
吞吐量 = 运行用户代码时间 /（运行用户代码时间+垃圾收集时间）
```

#### Parallel Old收集器

配置参数： -XX:+UseParallelOldGC

特点：

- Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。
- 在注重**吞吐量，**CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

#### ParNew收集器

配置参数： -XX:+UseParNewGC

配置参数： -XX:ParallelGCThreads=n 设置并行收集器收集时使用的并行收集线程数。一般最好和计算机的CPU相当

特点：

- 新生代并行（ParNew），老年代串行（Serial Old）
- Serial收集器的多线程版本
- 注意：单CPU性能并不如Serial，因为存在线程交互的开销

#### CMS收集器

配置参数： -XX:+UseConcMarkSweepGC 应用CMS收集器

尽管CMS收集器采用的是并发回收，但是在其初始标记和重新标记这两个阶段中仍然需要执行“STW”暂停程序中的工作线程，不过暂停时间并不会太长，目前所有的垃圾收集器都做不到完全不需要“STW”，只是尽可能地缩短暂停时间。

由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的。另外，由于在垃圾收集阶段用户线程没有中断，所以在CMS回收过程中，还应该确保应用程序用户线程有足够的内存可用。

特点:

- 低延迟：减少STW对用户体验的影响【低延迟要求高】
- 并发收集：可以同时执行用户线程
- CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是当堆内存使用率达到某一阈值时，便开始进行回收。
- CMS收集器的垃圾收集算法采用的是标记-清除算法。
- 会产生内存碎片，导致并发清除后，用户线程可用的空间不足。
- CMS收集器对CPU资源非常敏感。

CMS整个过程比之前的收集器要复杂，整个过程分为4个主要阶段：

**初始标记（Initial-Mark）阶段**：

- 本阶段任务：标记出GCRoots能直接关联到的对象。
- 一旦标记完成之后就会恢复之前被暂停的所有应用线程。
- 由于直接关联对象比较小，所以这里的速度非常快。
- 会STW

**并发标记（Concurrent-Mark）阶段**：

- 本阶段任务：从GC Roots的直接关联对象遍历整个对象图
- 这个过程耗时较长
- 不会STW

**重新标记（Remark）阶段**：

- 本阶段任务：修正并发标记期间，因用户程序继续运作产生的新的对象记录
- 这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短。
- 会STW

**并发清除（Concurrent-Sweep）阶段**：

- 本阶段任务：清理删除掉标记阶段判断的已经死亡的对象，释放内存空间。
- 由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的。

#### G1（Garbage-First）收集器

Garbage First（简称G1）收集器是垃圾收集器技术发展历史上的里程碑式的成果，它开创了收集器**面向**

**局部收集**的设计思路和**基于Region的内存布局**形式。

JDK 8以后G1收集器才被Oracle官方称为**全功能的垃圾收集器**。

G1是一款**面向服务端应用的垃圾收集器**，**大内存**企业配置的垃圾收集器大多都是G1。

JDK 9发布之日G1宣告取代Parallel Scavenge加Parallel Old组合，成为服务端模式下的默认垃圾收

集器，而CMS则被声明为不推荐使用（Deprecate）。G1最大堆内存是 32MB * 2048=64G ，G1最小堆内存 1MB * 2048=2GB ，低于此值建议使用其它收集器。

特点：

1. 并行与并发：充分利用多核环境下的硬件优势

2. 多代收集：不需要其他收集器配合就能独立管理整个GC堆

3. **空间整合**：“标记-整理”算法实现的收集器，局部上基于“复制”算法不会产生内存空间碎片

4. **可预测的停顿**：能让使用者明确指定消耗在垃圾收集上的时间。当然，更短的GC时间的代价是回收空间的效率降低。

G1收集器的运作大致可划分为以下几个步骤：

1. **初始标记：**标记一下GC Roots能直接关联到的对象，需要停顿线程，但耗时很短

2. **并发标记：**是从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段耗时较长，但可与用户程序并发执行

3. **最终标记：**修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录

4. **筛选回收：**对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划

G1中有三种模式垃圾回收模式，**Young GC、Mixed GC 和Full GC**，在不同的条件下被触发。

#### ZGC（Z Garbage Collector）

Z Garbage Collector，也称为ZGC，在 JDK11 中引入的一种可扩展的低延迟垃圾收集器，在 JDK15 中发布稳定版。

ZGC的目标：

- **< 1ms** 最大暂停时间（jdk < 16 是 10ms，jdk >=16 是 <1ms ）
- 暂停时间不会随着堆、live-set 或 root-set 的大小而增加
- 适用内存大小从 8MB 到16**TB** 的堆

ZGC 具有以下特征：

- 并发
- 基于 region
- 压缩
- NUMA 感知
- 使用彩色指针
- 使用负载屏障

ZGC 收集器是一款基于 Region 内存布局的， 不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现**可并发的标记整理算法的，以低延迟为首要目标的一款垃圾收集器**。ZGC 的核心是一个并发垃圾收集器，这意味着所有繁重的工作都在Java 线程继续执行的同时完成。这极大地限制了垃圾收集对应用程序响应时间的影响。

### Minor GC 、Major GC和 Full GC 有什么区别

新生代收集（Minor GC/Young GC）：指目标只是新生代的垃圾收集。Minor GC 非常频繁，回收速度比较快。

老年代收集（Major GC/Old GC）：指目标只是老年代的垃圾收集， Major GC 一般比 Minor GC慢 10 倍以上。目前只有CMS收集器会有单独收集老年代的行为。另外请注意“Major GC”这个说法现在有点混淆，在不同资料上常有不同所指，需按上下文区分到底是指老年代的收集还是整堆收集。

整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集。

混合收集（Mixed GC）：指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有G1收集器会有这种行为。

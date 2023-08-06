# Week 2 Homework

题目 01- 请你用自己的语言向我介绍 Java 运行时数据区（内存区域）

- 堆、虚拟机栈、本地方法栈、方法区（永久代、元空间）、运行时常量池（字符串常量池）、直接内存

  ```
  堆（Heap）：JVM启动时创建，用于存放对象实例、数组和运行时常量的内存分配，也是垃圾回收的主要内存区域。当堆中没有足够内存来创建对象时，会抛出OOM（OutOfMemoryError）异常。
  JVM对堆进行了分代管理，分为新生代（Young Generation）和老年代（Old Generation）。
  
  虚拟机栈（Java VM Stack）：虚拟机栈是用于存储和管理Java方法执行过程中的内存模型。每个方法调用和执行都会创建一个栈帧（Stack Frame）被称为当前栈帧，并且进行入栈和出栈。栈帧存储了方法的局部变量表（Local Variable Table）、操作数栈（Operand Stack）、动态连接（Dynamic Linking）和返回地址（Return Address）等信息。线程请求的栈深度大于-Xss虚拟机深度时，会抛出StackOverflowError异常。
  栈是运行时的单位，堆是存储的单位。
  
  本地方法栈（Native Method Stack）：用于储存和管理本地方法（Native Method）的调用和执行。本地方法是指非Java语言编写的方法，因此可以理解为一个本地方法（Native Method）就是一个Java调用非Java代码的接口。
  ```

- 为什么堆内存要分年轻代和老年代？

  ```
  主要目的是为了使JVM可以更好的管理和分配堆内存中的对象，提高程序的性能和效率，降低垃圾收集成本。
  新生代占堆内存的1/3，被划分为3个区域，Eden、Survivor From 和 Survivor To，其比例为8：1：1。新创建的对象会放在新生代的Eden里，当Eden区被占满时，会触发一次新生代的垃圾回收（MinorGC）。此时存活的对象会被记录其年龄并放到SurvivorFrom区，当Eden再次被占满触发垃圾回收（MinorGC）时，会扫描Eden和From区并进行垃圾回收（MinorGC），并将存活的对象放入SurvivorTo区。最后From区和To区互换，即谁空谁是To。如此互换15次（默认的MaxTenuringThreshold）后仍然存活的对象进入老年代。
  而老年代是用来存放生命周期较长的对象的，占堆内存的2/3。当一个对象在新生代经历过多次垃圾回收（MinorGC）时仍然存活，或对象由于过大而无法放入新生代时，就会被放入老年代。而当老年代区被占满时，也同样会触发老年代的垃圾回收（MajorGC）。
  ```

题目 02- 描述一个 Java 对象的生命周期

- 解释一个对象的创建过程

  ```
  
  ```

- 解释一个对象的内存分配

  ```
  
  ```

- 解释一个对象的销毁过程

  ```
  
  ```

- 对象的 2 种访问方式是什么？

  ```
  
  ```

- 为什么需要内存担保？

  ```
  
  ```

题目 03- 垃圾收集算法有哪些？垃圾收集器有哪些？他们的特点是什么？

- ParNew 收集器

  ```
  
  ```

- ParallelScavenge 收集器

  ```
  
  ```

- ParallelOld 收集器

  ```
  
  ```

- CMS 收集器

  ```
  
  ```

- G1 收集器

  ```
  
  ```

  
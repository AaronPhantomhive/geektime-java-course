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

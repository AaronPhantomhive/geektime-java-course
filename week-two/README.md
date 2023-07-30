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

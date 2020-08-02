# 运行时数据区

<img src="https://upload-images.jianshu.io/upload_images/10006199-a4108d8fb7810a71.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp" alt="img" style="zoom:67%;" />

方法区：类信息、常量、静态变量、即使编译器编译后的代码

 运行时常量池，用于存放编译器生成的各种字面量和符号引用。编译器和运行期都可以将常量放入池中。内存有限，无法申请时会抛出OutOfMemoryError

虚拟机栈：Java方法（局部变量表、操作数栈、动态链接、方法出口）

本地方法栈

堆：对象和数组

程序计数器：虚拟机字节码指令的地址或Undefind



直接内存

非虚拟机运行时数据区的部分



# 堆和栈的区别



| 区域名称   | 特性                                                         |
| ---------- | ------------------------------------------------------------ |
| 程序计数器 | 指示当前程序执行到了哪一行，执行JAVA方法时纪录正在执行的虚拟机字节码指令地址；执行本地方法时，计数器值为undefined |
| 虚拟机栈   | 用于执行JAVA方法。栈帧存储局部变量表、操作数栈、动态链接、方法返回地址和一些额外的附加信息。程序执行时栈帧入栈；执行完成后栈帧出栈 |
| 本地方法栈 | 用于执行本地方法，其它和虚拟机栈类似                         |
| JAVA堆     | JAVA虚拟机管理的内存中最大的一块，所有线程共享，几乎所有的对象实例和数组都在这类分配内存。**GC主要就是在JAVA堆中进行的**。 |
| 方法区     | 用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。但是已经被最新的 JVM 取消了。现在，被加载的类作为元数据加载到底层操作系统的本地内存区。 |

# HotSop虚拟机

## 对象的创建

new的时候会执行类加载器

## 类加载器



# ClassLoader类加载器

**一类是系统提供的(启动类加载器)**

引导类加载器（bootstrapclass loader）：它用来加载 Java 的核心库，是用原生代码而不是java来实现的，并不继承自java.lang.ClassLoader，除此之外基本上所有的类加载器都是java.lang.ClassLoader类的一个实例。

扩展类加载器（extensionsclass loader）：它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录（一般为%JRE_HOME%/lib/ext）。该类加载器在此目录里面查找并加载 Java 类。

系统类加载器（systemclass loader或 App class loader）：它根据当前Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader() 来获取它。

**另外一类则是由 Java 应用开发人员编写的**

开发人员可以通过继承java.lang.ClassLoader 类的方式实现自己的类加载器，以满足一些特殊的需求

# 类加载过程

![image-20200622232854675](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200622232854675.png)

# 双亲委派模型

应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器

下图展示了类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系（Inheritance）。

1. 工作过程
一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

2. 好处
使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。



# GC算法

## 复制算法

优点：
没有标记和清除的过程，实现简单，运行高效
复制过去以后保证空间的连续性，不会出现碎片问题

缺点:
此算法的缺点也是很明显的，就是需要两倍的内存空间
频繁的递归调用函数. 对栈的压力比较大

特别的：
如果存活的对象比较多，哪岂不是每次复制的量级也很大
（不过我感觉这个应用于年轻代中就不存在这个问题  因为年轻的对象都是朝生夕死 不会经常存在大量的对象  哪对于老年代来说就不适用了)



## 标记-清除算法

## 标记-压缩算法





# jvm调优

## JVM 调优的工具

JDK 自带了很多监控工具，都位于 JDK 的 bin 目录下，其中最常用的是 jconsole 和 jvisualvm 这两款视图监控工具。

jconsole：用于对 JVM 中的内存、线程和类等进行监控；
jvisualvm：JDK 自带的全能分析工具，可以分析：内存快照、线程快照、程序死锁、监控内存的变化、gc 变化等。

## JVM 调优的参数

-Xms2g：初始化推大小为 2g；
-Xmx2g：堆最大内存为 2g；
-XX:NewRatio=4：设置年轻的和老年代的内存比例为 1:4；
-XX:SurvivorRatio=8：设置新生代 Eden 和 Survivor 比例为 8:2；
–XX:+UseParNewGC：指定使用 ParNew + Serial Old 垃圾回收器组合；
-XX:+UseParallelOldGC：指定使用 ParNew + ParNew Old 垃圾回收器组合；
-XX:+UseConcMarkSweepGC：指定使用 CMS + Serial Old 垃圾回收器组合；
-XX:+PrintGC：开启打印 gc 信息；
-XX:+PrintGCDetails：打印 gc 详细信息





场景




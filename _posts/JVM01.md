---
title: Java虚拟机（一）
date: 2017-06-04 19:20:29
tags:
- java虚拟机
- 堆、栈、方法区
- 虚拟机参数
description: 今天进入Java虚拟机知识的学习，该内容共分为两部分学习，在第一部分中我们将介绍以下内容：（1）java虚拟机概述和基本概念 （2）堆、栈、方法区 （3）了解虚拟机参数

categories: JVM
---
# java虚拟机概述和基本概念
## java虚拟机的原理
&ensp;&emsp;&emsp;所谓**虚拟机**，就是一台虚拟的机器。它是一款软件，用来执行一系列虚拟计算指令，大体上虚拟机可以分为**系统虚拟机**和**程序虚拟机**，大名鼎鼎的**Visual Box**、**VMare**就属于**系统虚拟机**，他们完全是对物理计算机的仿真，提供了一个可运行完整操作系统的软件平台。**程序虚拟机**典型代表就是**Java虚拟机**，它专门为执行单个计算机程序而设计，在java虚拟机中执行的指令我们称为**java字节码指令**。无论是系统虚拟机还是程序虚拟机，在上面运行的软件都被限制于虚拟机提供的资源中。Java发展至今，出现过很多虚拟机，最初**Sun**使用的一款叫**Classic**的java虚拟机，到现在引用最广泛的是**HotSpot虚拟机**，除了Sun以外，还有**BEA的JRockit**，目前JRockit和HotSpot都被Oracle收入旗下，大有整合的趋势。
## 认识java虚拟机的基本结构

![JVM01](/JVM01/JVM01.png)

## 基本概念说明
1. **类加载子系统：**负责从文件系统或者网络中加载Class信息，加载的信息存放在一块称之为**方法区的内存空间**。
2. **方法区：**就是存放**类信息**、**常量信息**、**常量池信息**、包括**字符串字面量**和**数字常量**等。
3. **java堆：**在*java虚拟机启动的时候*建立java堆，它是java程序最主要的内存工作区域，几乎所有的**对象实例**都存放到java堆中，**堆空间是所有线程共享的**。
4. **直接内存：**java的NIO库允许java程序使用直接内存，从而提高性能，**通常直接内存速度会优于java堆**。**读写频繁**的场合可能会考虑使用。
5. **java栈：****每个虚拟机线程都有一个私有的栈**，一个线程的java栈在线程创建的时候被创建，java栈中保存着**局部变量**、**方法参数**、同时**java的方法调用**、**返回值**等。
6. **本地方法栈：**和java栈非常类似，最大不同为**本地方法栈用于本地方法调用**。java虚拟机允许java直接调用本地方法（通常使用C编写）。
7. **垃圾收集系统：**java的核心，也是必不可少的，java有一套自己进行垃圾清理的机制，开发人员无需手工清理。
8. **PC（Program Counter）寄存器**：也是每个线程私有的空间，java虚拟机会为每个线程创建PC寄存器，在任意时刻，一个java线程总是在执行一个方法，这个方法被称为当前方法，如果当前方法不是本地方法，PC寄存器就会执行当前正在被执行的指令，如果是本地方法，则PC寄存器值为undefined,寄存器存放如**当前执行环境指针**、**程序计数器**、**操作栈指针**、**计算的变量指针**等信息。
9. **执行引擎：**虚拟机最核心的组件就是执行引擎了，它负责执行虚拟机的字节码（.Class文件）。一般先进行编译成机器码后执行。

# 堆、栈、方法区
## 堆、栈、方法区概念和联系
* **堆**解决的是数据存储的问题，即数据怎么放、放在哪儿。
* **栈**解决程序的运行问题，即程序如何运行，或者说如何处理数据。
* **方法区**则是辅助堆栈的快永久区（Perm），解决堆栈信息的产生，是先决条件。

&ensp;&emsp;&emsp;我们创建一个新的对象，User：那么User类的一些信息（**类信息、静态信息都存在于方法区中**）；而**User类被实例化出来之后，被存储到java堆中的一块内存空间**；当我们去使用的时候，都是使用User对象的引用，形如**User user=new User()**，**这里的user就是存放在java栈中**的，即User真实对象的一个引用。

![JVM02](/JVM01/JVM02.png)

## 辨清java堆
&ensp;&emsp;&emsp;**java堆**是和java应用程序关系最密切的内存空间，几乎所有的对象都存放在其中，并且java堆完全是自动化管理的，通过垃圾回收机制，垃圾对象会自动管理，不需要显示地释放。
&ensp;&emsp;&emsp;根据垃圾回收机制不同，Java堆有可能拥有不同的结构。**最为常见的就是将整个java堆分为新生代和老年代**。其中新生代存放新生的对象或者年龄不大的对象，老年代则存放老年对象。
&ensp;&emsp;&emsp;**新生代**分为**eden区**、**s0区**、**s1区**，**s0和s1也被称为from和to区域**，他们是两块大小相等并且可以互换角色的空间（复制算法）。
&ensp;&emsp;&emsp;绝大多数情况下，对象首先分配在eden区，在一次新生代回收后，如果对象还存活，则会进入s0或者s1区（s0和s1不能同时使用），之后每经过一次新生代回收，如果对象存活则它的年龄就加1，当对象达到一定的年龄后，则进入老年代。

![JVM03](/JVM01/JVM03.png)

## java栈
&ensp;&emsp;&emsp;java栈是一块线程私有的内存空间，一个栈，一般由三部分组成：**局部变量表**、**操作数栈**和**帧数据区**。

* **局部变量表：**用于**报错函数的参数**及**局部变量**。
* **操作数栈：**主要**保存计算过程的中间结果**，同时作为计算过程中变量临时的存储空间。
* **帧数据区：**除了局部变量表和操作数栈以外，栈还需要一些数据来支持**常量池的解析**，这里帧数据区保存着访问常量池的指针，方便程序访问常量池，另外，当函数返回或者出现异常时，虚拟机必须有一个**异常处理表**，方便发送异常的时候找到异常的代码，因此异常处理表也是帧数据区的一部分。

![JVM04](/JVM01/JVM04.png)

## java方法区
&ensp;&emsp;&emsp;**java方法区**和堆一样，方法区是一块所有线程共享的内存区域，它保存系统的类信息，比如类的**字段、方法、常量池**等。方法区的大小决定了系统可以保存多少个类，如果系统定义太多的类，导致方法区溢出。虚拟机同样会抛出内存溢出错误。方法区可以理解为**永久区（Perm）**。

# 虚拟机参数
&ensp;&emsp;&emsp;在虚拟机运行的过程中，如果可以跟踪系统的运行状态，那么对于问题的故障排查会有一定的帮助，为此，虚拟机提供了一些跟踪系统状态的参数，使用给定的参数执行java虚拟机，就可以在系统运行时打印相关日志，用于分析实际问题。我们进行虚拟机参数配置，其实**主要就是围绕着堆、栈、方法区进行配置**。

## 堆分配参数（一）
**-XX:+PrintGC**使用这个参数，虚拟机启动后，只要遇到GC就会打印日志。
**-XX:+UseSerialGC**配置串行回收器
**-XX:+PrintGCDetails**可以查看详细信息，包括各个区的情况
**-Xms:**设置java程序启动时初始化堆大小
**-Xmx:**设置java程序能获得的最大堆大小
**-Xmx20m -Xms5m -XX:+PrintCommandLineFlags:**可以将隐式或者显示传给虚拟机的参数输出

_Test01.java_

```java
public class Test01 {

	public static void main(String[] args) {

		//-Xms5m -Xmx20m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:+PrintCommandLineFlags
		
		//查看GC信息
		System.out.println("max memory:" + Runtime.getRuntime().maxMemory());
		System.out.println("free memory:" + Runtime.getRuntime().freeMemory());
		System.out.println("total memory:" + Runtime.getRuntime().totalMemory());
		
		byte[] b1 = new byte[1*1024*1024];
		System.out.println("分配了1M");
		System.out.println("max memory:" + Runtime.getRuntime().maxMemory());
		System.out.println("free memory:" + Runtime.getRuntime().freeMemory());
		System.out.println("total memory:" + Runtime.getRuntime().totalMemory());
		
		byte[] b2 = new byte[4*1024*1024];
		System.out.println("分配了4M");
		System.out.println("max memory:" + Runtime.getRuntime().maxMemory());
		System.out.println("free memory:" + Runtime.getRuntime().freeMemory());
		System.out.println("total memory:" + Runtime.getRuntime().totalMemory());
		
	}	
}
```
 总结：
&ensp;&emsp;&emsp;**在实际工作中，我们可以直接将初始的堆大小与最大堆大小设置相等，这样的好处是可以减少程序运行时的垃圾回收次数，从而提高性能。**

## 堆分配参数（二）
***新生代的配置***
**-Xmn:**可以设置**新生代**的大小，设置一个比较大的新生代会减少老年代的大小，这个参数对系统性能以及GC行为有很大的影响，**新生代大小一般会设置整个堆空间的1/3到1/4左右**。
**-XX:SurvivorRatio:**用来设置**新生代中eden空间和from/to空间的比例**。含义：-XX:SurvivorRatio=eden/from=eden/to

_Test02.java_

```java
public class Test02 {

	public static void main(String[] args) {
		
		//第一次配置
		//-Xms20m -Xmx20m -Xmn1m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC
		
		//第二次配置
		//-Xms20m -Xmx20m -Xmn7m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC
		
		//第三次配置
		//-XX:NewRatio=老年代/新生代
		//-Xms20m -Xmx20m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC
		
		byte[] b = null;
		//连续向系统申请10MB空间
		for(int i = 0 ; i <10; i ++){
			b = new byte[1*1024*1024];
		}
	}
}
```

总结：
&ensp;&emsp;&emsp;不同的堆分布情况，对系统执行会产生一定的影响，在实际工作中，应该根据系统的特点作出合理的配置，基本策略：**尽可能将对象预留在新生代，减少老年代的GC次数**。
&ensp;&emsp;&emsp;除了可以设置新生代的绝对大小（-Xmn），还可以使用**（-XX:NewRatio）设置新生代和老年代的比例**：-XX:NewRatio=老年代／新生代

## 堆溢出处理
&ensp;&emsp;&emsp;在java程序的运行过程中，如果堆空间不足，则会抛出内存溢出的错误（Out of Memory）OOM，一旦这类问题发生在生产环境，可能引起严重的业务中断，java虚拟机提供了**-XX:+HeapDumpOnOutOfMemoryError**,使用该参数可以在内存溢出时导出整个堆信息，与之配合使用的还有参数，**-XX:HeapDumpPath**,可以设置导出堆的存放路径。
内存分析工具：Memory Analyzer 1.5.0
地址：[http://download.eclipse.org/mat/1.5/update-site/](http://download.eclipse.org/mat/1.5/update-site/)

_Test03.java_

```java
public class Test03 {

	public static void main(String[] args) {
		
		//-Xms1m -Xmx1m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:/Test03.dump
		//堆内存溢出
		Vector v = new Vector();
		for(int i=0; i < 5; i ++){
			v.add(new Byte[1*1024*1024]);
		}
		
	}
}
```

## 栈配置
&ensp;&emsp;&emsp;java虚拟机提供了参数**-Xss**来指定线程的最大栈空间，整个参数也直接决定了函数可调用的最大深度。

_Test04.java_

```java
public class Test04 {

	//-Xss1m  
	//-Xss5m
	
	//栈调用深度
	private static int count;
	
	public static void recursion(){
		count++;
		recursion();
	}
	public static void main(String[] args){
		try {
			recursion();
		} catch (Throwable t) {
			System.out.println("调用最大深入：" + count);
			t.printStackTrace();
		}
	}
}
```

## 方法区
&ensp;&emsp;&emsp;和java堆一样，方法区是一块所有线程共享的内存区域，它用于保存系统的类信息，方法区（永久区）可以保存多少信息可以对其进行配置，在默认情况下**，-XX:MaxPermSize为64MB**，如果系统运行时生产大量的类，就需要设置一个相对合适的方法区，以免出现永久区内存溢出的问题。
-XX:PermSize=64M -XX:MaxPermSize=64M

## 直接内存配置
&ensp;&emsp;&emsp;直接内存也是java程序中非常重要的组成部分，特别是广泛用在NIO中，直接内存跳过了java堆，使java程序可以直接访问原生堆空间，因此在一定程度上加快了内存空间的访问速度。但是说直接内存一定就可以提高内存访问速度也不见得，具体情况具体分析。
&ensp;&emsp;&emsp;相关配置参数：**-XX：MaxDirectMemorySize**，如果不设置默认值为最大堆空间，即-Xmx。直接内存使用达到上限时，就会触发垃圾回收，如果不能有效的释放空间，也会引起系统的OOM。

## Client和Server虚拟机工作模式（适用jdk早期版本,不使用jdk1.7以后版本）
&ensp;&emsp;&emsp;目前java虚拟机支持Client和Server两种运行模式，使用参数-client可以制定使用Client模式，使用-server即使用Server模式。可以直接在命令行查看当前计算机系统自动选择的运行模式。java -version 即可。
&ensp;&emsp;&emsp;二者区别：**Client模式相对Server启动较快，如果不追求系统的长时间使用性能仅仅是测试，可以使用Client模式。而Server模式则启动比较慢，原因是会对其进行复杂的系统性能信息收集和使用更复杂的算法对程序进行优化，一般我们的生产环境都会使用Server模式，长期运行其性能要远远快于Client模式。**

JVM不错的博客：
[http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)




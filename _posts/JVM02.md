---
title: Java虚拟机（二）
date: 2017-06-05 18:09:05
tags:
- 垃圾回收算法
- 垃圾回收器
- Tomcat性能影响实验
- 性能监控工具

description: 今天继续Java虚拟机知识的学习，该内容共分为两部分学习，在第二部分中我们将介绍以下内容：（1）垃圾回收概念和算法、及对象的分代转换 （2）垃圾回收器 （3）Tomcat性能影响实验 （4）性能监控工具

categories: JVM
---
# 垃圾回收概念和算法及对象的分代转换
&ensp;&emsp;&emsp;谈到**垃圾回收**（Garbage Collection，简称GC），需要先澄清什么是垃圾，类比日常生活中的垃圾，我们会把他们丢入垃圾桶，然后倒掉。GC中的垃圾，特指存于内存中、不会再被使用的对象，而回收就是相当于把垃圾“倒掉”。垃圾回收有很多种算法：如**引用计数法**、**标记压缩法**、**复制算法**、**分代**、**分区**的思想。
## 垃圾收集算法（一）
**引用计数法：**这是个比较古老而经典的垃圾收集算法，其核心就是在对象被其他所引用时计数器加1，而当引用失效时则减1，但是这种方式有**非常严重的问题**：**无法处理循环引用的情况、还有就是每次进行加减操作比较浪费系统性能**。

**标记清除法：**就是分为标记和清除两个阶段进行处理内存中的对象，当然这种方式也有非常大的弊端，就是**空间碎片问题**，垃圾回收后的空间不是连续的，不连续的内存空间的工作效率要低于连续的内存空间。

**复制算法：**其核心思想就是将内存空间分为两块，每次只使用其中一块，在垃圾回收时，将正在使用的内存中的存留对象复制到未被使用的内存块中去，之后去清除之前正在使用的内存块中所有的对象，反复去交换两个内存的角色，完成垃圾收集。（**java中新生代的from和to空间就是使用这个算法**）

**标记压缩法：**标记压缩法在标记清除法基础之上做了优化，把存活的对象压缩到内存一端，而后进行垃圾清理。（**java中老年代使用的就是标记压缩法**）

考虑一个问题：**为什么新生代和老年代使用不同的算法**？
答案：因为新生代GC进行频繁，里面对象不稳定，老年代里的对象稳定，默认为永久存活在内存中，GC次数就不频繁，一次回收个数不多。

## 垃圾收集算法（二）
**分代算法：**就是根据对象的特点把内存分为N块，而后根据每个内存的特点使用不同的算法。
&ensp;&emsp;&emsp;**对于新生代和老年代来说，新生代回收频率很高，但是每次回收耗时都很短，而老年代回收频率较低，但耗时会相对较长，所以应该尽量减少老年代的GC。**

**分区算法：**其主要就是将整个内存分为N多个小的独立空间，每个小空间都可以独立使用，这样细粒度的控制一次回收几个小空间和哪些个小空间，而不是对整个空间进行GC，从而提升性能，并**减少GC的停顿时间**。（jdk1.7后开始，但还没有普及）

## 垃圾回收时的停顿现象
&ensp;&emsp;&emsp;垃圾回收器的任务是识别和回收垃圾对象进行内存清理，为了让垃圾回收器可以高效的执行，大部分情况下，会要求系统进入一个停顿的状态。**停顿的目的是终止所有应用线程，只有这样系统才不会有新的垃圾产生，同时停顿保证了系统状态在某一个瞬间的一致性，也有益于更好地标记垃圾对象**。因此在垃圾回收时，都会产生应用程序的停顿。

## 对象如何进入老年代
&ensp;&emsp;&emsp;一般而言对象首次创建会被放置在新生代的eden区，如果没有GC介入，则对象不会离开eden区，那么eden区的对象如何进入老年代呢？一般来讲，只要对象的年龄达到一定的大小，就会自动离开年轻代进入老年代，对象年龄是由对象经历数次GC决定的，在新生代每次GC之后如果对象没有被回收则年龄加1。**虚拟机提供了一个参数来控制新生代对象的最大年龄，当超过这个年龄范围就会晋升老年代。**
**-XX:MaxTenuringThreshold,默认情况下为15。**

_Test05.java_

```java
public class Test05 {

	public static void main(String[] args) {
		//初始的对象在eden区
		//参数：-Xmx64M -Xms64M -XX:+PrintGCDetails
		for(int i=0; i< 5; i++){
			byte[] b = new byte[1024*1024];
		}
		

		//测试进入老年代的对象
		//
		//参数：-Xmx1024M -Xms1024M -XX:+UseSerialGC -XX:MaxTenuringThreshold=15 -XX:+PrintGCDetails 
		//-XX:+PrintHeapAtGC
//		Map<Integer, byte[]> m = new HashMap<Integer, byte[]>();
//		for(int i =0; i <5 ; i++) {
//			byte[] b = new byte[1024*1024];
//			m.put(i, b);
//		}
//		
//		for(int k = 0; k<20; k++) {
//			for(int j = 0; j<300; j++){
//				byte[] b = new byte[1024*1024]; 
//			}
//		}
	}
}
```
总结：
&ensp;&emsp;&emsp;根据设置**MaxTenuringThreshold参数**，可以指定新生代对象经过多少次回收后进入老年代。
&ensp;&emsp;&emsp;另外，**大对象**（新生代eden区无法装入时，也会直接进入老年代）。JVM里有个参数可以设置对象的大小超过在指定的大小之后，直接晋升老年代。
**-XX:PretenureSizeThreshold**

_Test06.java_

```java
public class Test06 {

	public static void main(String[] args) {
		
		//参数：-Xmx30M -Xms30M -XX:+UseSerialGC -XX:+PrintGCDetails -XX:PretenureSizeThreshold=1000
		//这种现象原因为：虚拟机对于体积不大的对象 会优先把数据分配到TLAB区域中，因此就失去了在老年代分配的机会
		//参数：-Xmx30M -Xms30M -XX:+UseSerialGC -XX:+PrintGCDetails -XX:PretenureSizeThreshold=1000 -XX:-UseTLAB
		Map<Integer, byte[]> m = new HashMap<Integer, byte[]>();
		for(int i=0; i< 5*1024; i++){
			byte[] b = new byte[1024];
			m.put(i, b);
		}
	}
}
```
总结：
&ensp;&emsp;&emsp;使用**PretenureSizeThreshold**可以进行指定进入老年代的对象大小，但是要注意**TLAB区域优先分配空间**。

## TLAB

&ensp;&emsp;&emsp;**TLAB全称是Thread Local Allocation Buffer**即**线程本地分配缓存**，从名字上看是一个线程专用的内存分配区域，是为了加速对象分配而生的。每一个线程都会产生一个TLAB，该线程独享的工作区域，**java虚拟机使用这种TLAB区来避免多线程冲突问题**，提高了对象分配的效率。TLAB空间一般不会太大，当大对象无法在TLAB分配时，则会直接分配到堆上。
**-XX:+UseTLAB** 使用TLAB
**-XX:+TLABSize** 设置TLAB大小（jdk1.7后可以自动调整，不要轻易修改）
**-XX:TLABRefillWasteFraction**设置维护进入TLAB空间的单个对象大小，他是一个比例值，默认为64，即如果对象大于整个空间的1/64,则在堆创建对象。
**-XX:+PrintTLAB** 查看TLAB信息
**-XX:ResizeTLAB** 自调整TLABRefillWasteFraction阈值

_Test07.java_

```java
public class Test07 {

	public static void alloc(){
		byte[] b = new byte[2];
	}
	public static void main(String[] args) {
		
		//TLAB分配
		//参数：-XX:+UseTLAB -XX:+PrintTLAB -XX:+PrintGC -XX:TLABSize=102400 -XX:-ResizeTLAB -XX:TLABRefillWasteFraction=100 -XX:-DoEscapeAnalysis -server
		for(int i=0; i<10000000;i++){
			alloc();
		}
	}
}
```

## 对象创建流程图
&ensp;&emsp;&emsp;一个对象创建在什么位置，我们的jvm会有一个比较细节的流程，根据数据的大小，参数的设置，决定如何创建分配，以及其位置。

![JVM01](/JVM02/JVM01.png)

# 垃圾收集器
&ensp;&emsp;&emsp;在java虚拟机中，垃圾回收器不仅仅只有一种，什么情况下该使用哪种，对性能又有什么影响，这都是我们需要了解的。

* 串行垃圾回收器
* 并行垃圾回收器
* CMS回收器
* G1回收器

## 串行回收器
&ensp;&emsp;&emsp;**串行回收器**是指使用**单线程**进行垃圾回收的回收器。每次回收时，串行回收器只有一个工作线程，对于并发能力较强的计算机来说，串行回收器的专注性和独占性往往有更好的性能表现。串行回收器可以在新生代和老年代使用，**根据作用于不同的堆空间**，分为**新生代串行回收器**和**老年代串行回收器**。
&ensp;&emsp;&emsp;使用**-XX:+UseSerialGC**参数可以设置使用新生代串行回收器和老年代串行回收器。

## 并行回收器（ParNew回收器）
&ensp;&emsp;&emsp;**并行回收器**在串行回收器基础上做了改进，他可以使用**多个线程**同时进行垃圾回收，对于计算能力强的计算机而言，可以有效的缩短垃圾回收所需的实际时间。
&ensp;&emsp;&emsp;**ParNew回收器**是一个**工作在新生代**的垃圾收集器，他只是简单的将串行回收器多线程化，他的**回收策略和算法和串行回收器一样**。
&ensp;&emsp;&emsp;使用**-XX:+UseParNewGC** **新生代ParNew回收器，老年代则使用串行回收器**。
&ensp;&emsp;&emsp;**ParNew回收器工作时的线程数量**可以使用**-XX:ParallelGC Threads**参数指定，一般最好和计算机的CPU相当 ，避免过多的线程影响性能。

## 并行回收器（ParallelGC回收器）
&ensp;&emsp;&emsp;**新生代ParallelGC回收器**，使用了**复制算法**的收集器，也是多线程独占形式的收集器，但ParallelGC回收器有个非常重要的特点，就是它非常关注系统的吞吐量。
&ensp;&emsp;&emsp;提供了两个非常关键的参数控制系统的吞吐量

* **-XX:MaxGCPauseMills:** **设置最大垃圾收集停顿时间**，可用把虚拟机在GC停顿的时间控制在MaxGCPauseMills范围内，如果希望减少GC停顿时间可以将MaxGCPauseMills设置的很小，但是会导致GC频繁，从而增加了GC的总时间，降低了吞吐量。所以需要根据实际情况设置该值。
* **-XX:GCTimeRatio:** **设置吞吐量大小**，它是一个0到100之间的整数，**默认情况下它的取值是99**，那么系统将花费不超过1/（1+n）的时间用于垃圾回收，也就是1/(1+99)=1%的时间。

&ensp;&emsp;&emsp;另外还可以指定**-XX:+UseAdaptiveSizePolicy**打开自适应模式，在这种模式下，新生代的大小、eden、from\to的比例，以及晋升老年代的对象年龄参数会被自动调整，以达到堆大小、吞吐量和停顿时间之间的平衡点。

## 并行回收器（ParallelOldGC回收器）
&ensp;&emsp;&emsp;**老年代ParallelOldGC回收器**也是一种**多线程**的回收器，和新生代的ParallelGC回收器一样，也是一种**关注吞吐量**的回收器，它使用了**标记压缩算法**进行实现。

* **-XX:+UseParallelOldGC**进行设置
* **-XX:+ParallelGCThreads**也可以设置垃圾收集时的线程数量

## CMS回收器
&ensp;&emsp;&emsp;**CMS全称为：Concurrent Mark Sweep**意为**并发标记清除**，他使用的是**标记清除法**，主要**关注系统停顿时间**。
&ensp;&emsp;&emsp;使用**-XX:+UseConcMarkSweepGC**进行设置。
&ensp;&emsp;&emsp;使用**-XX:ConcGCThreads**设置并发线程数量。
&ensp;&emsp;&emsp;CMS并不是独占（GC停顿）的回收器，也就说CMS回收的过程中，应用程序仍然在不停的工作，又会有新的垃圾不断产生，所以在使用CMS的过程中应该确保应用程序的内存足够可用。CMS不会等到应用程序饱和的时候才去回收垃圾，而是**在某一阀值的时候开始回收**，回收阀值可用指定的参数进行配置，**-XX:CMSInitiatingOccupancyFraction**未指定，**默认为68**,也就是说当老年代的空间使用率达到68%的时候，会执行CMS回收。如果内存使用率增长的很快，在CMS执行的过程中，已经从出现了内存不足的情况，此时CMS回收就会失败，虚拟机将启动老年代串行回收器进行垃圾回收，这会导致应用程序中断，直到垃圾回收完成后才会正常工作，这个过程GC的停顿时间可能较长，所以**-XX:CMSInitiatingOccupancyFraction**的设置要根据实际的情况。
&ensp;&emsp;&emsp;之前我们在学习算法的时候说过，标记清除法有个缺点就是存在内存碎片的问题，那么CMS有个参数设置**-XX:+UseCMSCompactAtFullCollection**可以**使CMS回收完成之后进行一次碎片整理**，**-XX:CMSFullGCBeforeCompaction**参数可以设置进行多少次CMS回收之后，对内存进行一次压缩。

## G1回收器
&ensp;&emsp;&emsp;**G1回收器(Garbage-First)**实现jdk1.7中提出的垃圾回收器，从长期目标来看是为了取代CMS回收器，G1回收器拥有独特的垃圾回收策略，**G1属于分代垃圾回收器**，区分新生代和老年代，依然有eden和from/to区，它并不要求整个eden区或者新生代、老年代的空间都连续，它使用了**分区算法**。
**并行性：**G1回收期间可多线程同时工作。
**并发性：**G1拥有与应用程序交替执行能力，部分工作可与应用程序同时执行，在整个GC期间不会完全阻塞应用程序。
**分代GC：**G1依然是一个分代的收集器，但是它是兼顾新生代和老年代一起工作，之前的垃圾收集器它们或者在新生代工作，或者在老年代工作，因此这是一个很大的不同。
**空间整理：**G1在回收过程中，不会像CMS那样在若干次GC后需要进行碎片整理，G1采用了**有效复制对象**的方式，减少空间碎片。
**可预见性：**由于分区的原因，G1可以只选取部分区域进行回收，缩小了回收的范围，提升了性能。
使用**-XX:+UseG1GC**应用G1收集器
使用**-XX:MaxGCPauseMilllis**指定最大停顿时间
使用**-XX:ParallelGCThreads**设置并行回收的线程数量

# Tomcat性能影响实验
**配置环境说明：**
&ensp;&emsp;&emsp;Tomcat7
&ensp;&emsp;&emsp;一个JSP网站
&ensp;&emsp;&emsp;测试网站吞吐量（1个指标、停顿时间、内存的使用情况，包括回收的效率...）
**工具：**
&ensp;&emsp;&emsp;Apache JMeter
&ensp;&emsp;&emsp;下载地址：[http://jmeter.apache.org/download_jmeter.cgi](http://jmeter.apache.org/download_jmeter.cgi)
**实验原理：**
&ensp;&emsp;&emsp;通过JMeter对Tomcat增加压力，不同的虚拟机参数应该会有不同的表现
**目的：**
&ensp;&emsp;&emsp;观察不同配置参数对吞吐量的影响

## 测试串行回收器
-XX:+PrintGCDetails -Xmx32M -Xms32M
-XX:+HeapDumpOnOutOfMemoryError
-XX:+UseSerialGC
-XX:PermSize=32M

![Config](/JVM02/Config.png)
测试结果显示吞吐量为：1165    116

![Res1](/JVM02/Res1.png)
其中JMeter聚合报告（Aggregate Report）生成的两个关键量：

* **Throughput：**吞吐量——默认情况下表示每秒完成的请求数（Request per Second），当使用了 Transaction Controller 时，也可以表示类似 LoadRunner 的 Transaction per Second 数
* **KB/Sec：**每秒从服务器端接收到的数据量，相当于LoadRunner中的Throughput/Sec

## 扩大堆内存以提升系统性能
-XX:+PrintGCDetails -Xmx512M -Xms32M
-XX:+HeapDumpOnOutOfMemoryError
-XX:+UseSerialGC
-XX:PermSize=32M
-Xloggc:e:/gc.log
测试结果显示吞吐量为：1374   137

![Res2](/JVM02/Res2.png)

## 调整初始堆大小
-XX:+PrintGCDetails -Xmx512M -Xms128M
-XX:+HeapDumpOnOutOfMemoryError
-XX:+UseSerialGC
-XX:PermSize=32M
-Xloggc:e:/gc1.log

测试结果显示吞吐量为：1682    167

![Res3](/JVM02/Res3.png)

## 测试ParNew回收器的表现
-XX:+PrintGCDetails -Xmx512M -Xms128M
-XX:+HeapDumpOnOutOfMemoryError
-XX:+UseParNewGC
-XX:PermSize=32M
-Xloggc:e:/gc2.log

测试结果显示吞吐量为：1698    169

![Res4](/JVM02/Res4.png)
**这个性能测试跟个人计算机有关**

## 使用ParallelOldGC回收器
-XX:+PrintGCDetails -Xmx512M -Xms128M
-XX:+HeapDumpOnOutOfMemoryError
-XX:+UseParallelGC
-XX:+UseParallelOldGC
-XX:ParallelGCThreads=4
-XX:PermSize=32M
-Xloggc:e:/gc3.log

测试结果显示吞吐量为：1891    188

![Res5](/JVM02/Res5.png)
压测需要测试多次，取平均值
注：**吞吐量最高，因Parallel专注于吞吐量。**

## 测试CMS回收器的性能
-XX:+PrintGCDetails -Xmx512M -Xms128M
-XX:+HeapDumpOnOutOfMemoryError
-XX:+UseConcMarkSweepGC
-XX:ConcGCThreads=4
-XX:PermSize=32M
-Xloggc:e:/gc4.log

测试结果显示吞吐量为：1815    181

![Res6](/JVM02/Res6.png)

# 性能监控工具
&ensp;&emsp;&emsp;**JConsole**是JDK自带的图形化性能监控工具，通过JConsole可以查看JAVA的应用程序运行概况，可以监控堆信息、永久区使用情况、类加载情况等。




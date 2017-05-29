---
title: 垃圾回收（Garbage Collection）
date: 2017-03-25 13:57:46
tags: 
- 垃圾回收
- GC

description: 在Java语言中，垃圾回收（Garbage Collection,GC）是一个非常重要的概念，它的主要作用是回收程序中不再使用的内存。

categories: JVM
---

# 垃圾回收机制的意义
&ensp;&emsp;&emsp;垃圾回收器的存在,一方面把开发人员从释放内存的复杂工作中解脱出来，提高了开发人员的生产效率；另一方面,对开发人员屏蔽了释放内存的方法，可以避免因开发人员错误地操作内存而导致应用程序的崩溃，保证了程序的稳定性。由于有个垃圾回收机制，Java中的对象不再有“作用域”的概念，**只有对象的引用才有“作用域”**。垃圾回收可以有效的防止内存泄露，有效的使用空闲的内存。
ps:**内存泄露**是指该内存空间使用完毕之后未回收，在不涉及复杂数据结构的一般情况下，Java 的内存泄露表现为一个内存对象的生命周期超出了程序需要它的时间长度，我们有时也将其称为**“对象游离”**。
# 垃圾回收机制中的算法
&ensp;&emsp;&emsp;Java语言规范没有明确地说明JVM使用哪种垃圾回收算法，但是任何一种垃圾回收算法一般要做2件基本的事情：
（1）发现无用信息对象；
（2）回收被无用对象占用的内存空间，使该空间可被程序再次使用。
## 引用计数算法（Reference Counting Collector）
**1.1 算法分析**
&ensp;&emsp;&emsp;**引用计数**是垃圾收集器中的早期策略。在这种方法中，堆中每个对象实例都有一个**引用计数**。当一个对象被创建时，且将该对象实例分配给一个变量，该变量计数设置为1。当任何其它变量被赋值为这个对象的引用时，计数加1（a = b,则b引用的对象实例的计数器+1），但当一个对象实例的某个引用超过了生命周期或者被设置为一个新值时，对象实例的引用计数器减1。任何引用计数器为0的对象实例可以被当作垃圾收集。当一个对象实例被垃圾收集时，它引用的任何对象实例的引用计数器减1。
**1.2 优缺点**
优点：
&ensp;&emsp;&emsp;引用计数收集器可以很快的执行，交织在程序运行中。对程序需要不被长时间打断的实时环境比较有利。
缺点：
&ensp;&emsp;&emsp;无法检测出循环引用。如父对象有一个对子对象的引用，子对象反过来引用父对象。这样，他们的引用计数永远不可能为0。
**1.3 引用计数算法无法解决循环引用问题**
例如：

```java
public class Main {
    public static void main(String[] args) {
        MyObject object1 = new MyObject();
        MyObject object2 = new MyObject();
          
        object1.object = object2;
        object2.object = object1;
          
        object1 = null;
        object2 = null;
    }
}
```
&ensp;&emsp;&emsp;最后面两句将object1和object2赋值为null，也就是说object1和object2指向的对象已经不可能再被访问，但是由于它们互相引用对方，导致它们的引用计数器都不为0，那么垃圾收集器就永远不会回收它们。
&ensp;&emsp;&emsp;下面的动画模拟的是根据内存对象的引用次数来清理垃圾。闪烁的红色表示正在进行引用计数。引用计数的最大好处在于它可以很快地检测出垃圾，从动画中你会发现，有些红色闪过之后的区域立即变成黑色。
![REF_COUNT_GC](/Garbage-Collection/REF_COUNT_GC.gif)

## Tracing算法(Tracing Collector) 或 标记-清除算法(mark and sweep)

2.1 根搜索算法
![Tracing-Collector](/Garbage-Collection/Tracing-Collector.png)
&ensp;&emsp;&emsp;**根搜索算法**是从离散数学中的图论引入的，程序把所有的引用关系看作一张图，从一个节点GC ROOT开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点。
Java中可作为**GC Root**的对象有

1. 虚拟机栈中引用的对象（本地变量表）
2. 方法区中静态属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈中引用的对象（Native对象）

2.2 Tracing算法的示意图

![Tracing-Collector2](/Garbage-Collection/Tracing-Collector2.png)

2.3 标记-清除算法分析
&ensp;&emsp;&emsp;**标记-清除算法**采用从根集合进行扫描，对存活的对象对象标记，标记完毕后，再扫描整个空间中未被标记的对象，进行回收，如上图所示。
优点：
&ensp;&emsp;&emsp;（1）**标记-清除算法**不需要进行对象的移动，并且仅对不存活的对象进行处理，在存活对象比较多的情况下极为高效；
&ensp;&emsp;&emsp;（2）解决了循环引用问题，而且开销要小得多，因为它不需要维护计数器。
缺点：
&ensp;&emsp;&emsp;由于标记-清除算法直接回收不存活的对象，因此会造成内存碎片。
![MARK_SWEEP_GC](/Garbage-Collection/MARK_SWEEP_GC.gif)
&ensp;&emsp;&emsp;从动画中可以看到，在一小段时间内没有出现红色闪烁，之后又突然出现了很多红色闪烁，说明它正在标记存活的对象。在完成标记之后，所有的垃圾被清除，释放出内存。我们还能从动画中看到，有些区域一下子全部转成黑色，而不是像引用计数算法那样慢慢地随时间扩散开来。

## compacting算法 或 标记-整理算法
![Compacting-Collector](/Garbage-Collection/Compacting-Collector.png)
3.1 **算法分析**
&ensp;&emsp;&emsp;标记-整理算法采用标记-清除算法一样的方式进行对象的标记，但在清除时不同，在回收不存活的对象占用的空间后，会将所有的存活对象往左端空闲空间移动，并更新对应的指针。
3.2 **优缺点**
优点：
&ensp;&emsp;&emsp;解决了内存碎片的问题。
缺点：
&ensp;&emsp;&emsp;在标记-清除算法的基础上，又进行了对象的移动，因此成本更高。

&ensp;&emsp;&emsp;在之前的动画里，对象不会发生移动。一旦对象分配到了内存，它就会待在原地不动，即使内存出现了很多碎片。后面的两个算法将会改变这种状况，它们使用不同的方式来实现回收。
![MARK_COMPACT_GC](/Garbage-Collection/MARK_COMPACT_GC.gif)
&ensp;&emsp;&emsp;标记整理是一种复杂的算法，需要多次遍历所有分配到内存的对象。从动画里我们可以看到，在红色闪烁之后，在计算对象的目标地址时发生了很多读写操作，然后对象被移动到目标地址上，对象的引用也被指向新的地址。这种复杂性所带来的主要好处就是极低的内存开销。Oracle的Hotspot虚拟机使用了多种垃圾回收算法，其中老年代空间使用的是标记整理算法。
在基于Compacting算法的收集器的实现中，一般增加句柄和句柄表。

## copying算法（Copying Collector）

![Copying-Collector](/Garbage-Collection/Copying-Collector.png)
&ensp;&emsp;&emsp;最后一种算法是大部分高性能垃圾回收系统的基础。它有点类似标记整理算法，不过相比之下要简单很多。它把内存分为两个部分，然后在这两个内存空间之间移动对象。在实际应用当中，一般会有多个内存空间，每个空间分配给不同年代的对象。先是在其中一个内存空间创建新对象，如果它们存活下来，就把它们复制到另一个内存空间。最后，如果这些对象的寿命足够长，它们会被复制到老年代空间。如果一个垃圾回收器被贴上分代或者朝生夕灭的标签，那它极有可能是多空间的复制回收器。
&ensp;&emsp;&emsp;除了简单性和灵活性，这种算法的最大优势在于，它只处理存活的对象。这种算法并不存在标记阶段，存活的对象直接被复制到新的地址上，对象引用也随之指向新的地址。
![COPY_GC](/Garbage-Collection/COPY_GC.gif)
&ensp;&emsp;&emsp;从动画中我们可以看到，有些对象集合几乎是整块地被复制到另一个内存空间里，这是比较糟糕的情况，这也就是为什么我们需要对垃圾回收器进行调优。如果我们能够通过调整内存大小和控制内存分配，确保大部分对象在垃圾回收开始之前死亡，那么，我们就得到了函数式编程和高性能的一个完美组合。

## generation算法(Generational Collector)
![Generational-Collector](/Garbage-Collection/Generational-Collector.png)
&ensp;&emsp;&emsp;分代的垃圾回收策略，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的回收算法，以便提高回收效率。
### 年轻代（Young Generation）
1. 所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。

2. 新生代内存按照8:1:1的比例分为一个eden区和两个survivor(survivor0,survivor1)区。一个Eden区，两个 Survivor区(一般而言)。大部分对象在Eden区中生成。回收时先将eden区存活对象复制到一个survivor0区，然后清空eden区，当这个survivor0区也存放满了时，则将eden区和survivor0区存活对象复制到另一个survivor1区，然后清空eden和这个survivor0区，此时survivor0区是空的，然后将survivor0区和survivor1区交换，即保持survivor1区为空， 如此往复。

3. 当survivor1区不足以存放 eden和survivor0的存活对象时，就将存活对象直接存放到老年代。若是老年代也满了就会触发一次Full GC，也就是新生代、老年代都进行回收

4. 新生代发生的GC也叫做Minor GC，MinorGC发生频率比较高(不一定等Eden区满了才触发)

### 年老代（Old Generation)
1. 在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。

2. 内存比新生代也大很多(大概比例是1:2)，当老年代内存满时触发Major GC即Full GC，Full GC发生频率比较低，老年代对象存活时间比较长，存活率标记高。

### 持久代（Permanent Generation）
&ensp;&emsp;&emsp;用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class，例如Hibernate 等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。

# GC（垃圾收集器）
**新生代**收集器使用的收集器：Serial、PraNew、Parallel Scavenge

**老年代**收集器使用的收集器：Serial Old、
Parallel Old、CMS
![Generation-Collector](/Garbage-Collection/Generation-Collector.png)

**Serial收集器（复制算法)**

新生代单线程收集器，标记和清理都是单线程，优点是简单高效。

**Serial Old收集器(标记-整理算法)**

老年代单线程收集器，Serial收集器的老年代版本。

**ParNew收集器(停止-复制算法)**　

新生代收集器，可以认为是Serial收集器的多线程版本,在多核CPU环境下有着比Serial更好的表现。

**Parallel Scavenge收集器(停止-复制算法)**

并行收集器，追求高吞吐量，高效利用CPU。吞吐量一般为99%， 吞吐量= 用户线程时间/(用户线程时间+GC线程时间)。适合后台应用等对交互相应要求不高的场景。

**Parallel Old收集器(停止-复制算法)**

Parallel Scavenge收集器的老年代版本，并行收集器，吞吐量优先

**CMS(Concurrent Mark Sweep)收集器（标记-清理算法）**

高并发、低停顿，追求最短GC回收停顿时间，cpu占用比较高，响应时间快，停顿时间短，多核cpu 追求高响应时间的选择

# GC的执行机制
由于对象进行了分代处理，因此垃圾回收区域、时间也不一样。GC有两种类型：**Scavenge GC**和**Full GC**。

**Scavenge GC**

&ensp;&emsp;&emsp;一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。

**Full GC**

&ensp;&emsp;&emsp;对整个堆进行整理，包括Young、Tenured和Perm。Full GC因为需要对整个堆进行回收，所以比Scavenge GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于FullGC的调节。
有如下原因可能导致Full GC：

1. 年老代（Tenured）被写满

2. 持久代（Perm）被写满

3. System.gc()被显示调用

4. 上一次GC之后Heap的各域分配策略动态变化

# Java有了GC同样会出现内存泄露问题
1. 静态集合类像HashMap、Vector等的使用最容易出现内存泄露，这些静态变量的生命周期和应用程序一致，所有的对象Object也不能被释放，因为他们也将一直被Vector等应用着。
2. 各种连接，数据库连接，网络连接，IO连接等没有显示调用close关闭，不被GC回收导致内存泄露。

3. 监听器的使用，在释放对象的同时没有相应删除监听器的时候也可能导致内存泄露。






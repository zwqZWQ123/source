---
title: 互联网并发编程中级篇（一）
date: 2017-05-26 15:02:14
tags: 
- 同步类容器
- 并发类容器
- Concurrent集合类
- CopyOnWrite集合类
- 各类并发Queue

description: 今天进入互联网并发编程知识的学习中级篇，在中级篇第一部分我们将掌握以下方面的内容：（1）同步类容器讲解 （2）并发类容器讲解（3）Concurrent集合类讲解与底层原理实现 （4）CopyOnWrite集合类讲解与底层原理实现（5）各类并发Queue详细讲解

categories: 
- 多线程&并发编程
---
# 同步类容器
&ensp;&emsp;&emsp;**同步类容器**都是**线程安全**的，**但在某些场景下可能需要加锁来保护复合操作**。复合类操作如：**迭代**（反复访问元素，遍历完容器中所有的元素）、**跳转**（根据指定的顺序找到当前元素的下一个元素）、以及**条件运算**。这些复合操作在多线程并发地修改容器时，可能会表现出意外的行为，最经典的便是**ConcurrentModificationException**，原因是当容器迭代的过程中，被并发的修改了内容，这是由于早期迭代器设计的时候并没有考虑并发修改的问题。
&ensp;&emsp;&emsp;同步类容器：如古老的Vector、HashTable。这些容器的同步功能其实都是有JDK的Collections.synchronized***等工厂方法去创建实现的。其底层的机制无非就是用传统的synchronized关键字对每个公用的方法都进行同步，使得每次只能有一个线程访问容器的状态。这很明显不满足我们今天互联网时代高并发的需求，在保证线程安全的同时，也必须要有足够好的性能。

_Tickets.java_

```java
public class Tickets {

	public static void main(String[] args) {
		//初始化火车票池并添加火车票:避免线程同步可采用Vector替代ArrayList  HashTable替代HashMap
		
		final Vector<String> tickets = new Vector<String>();
		
		//Map<String, String> map = Collections.synchronizedMap(new HashMap<String, String>());

		for(int i = 1; i<= 1000; i++){
			tickets.add("火车票"+i);
		}
		
//		for (Iterator iterator = tickets.iterator(); iterator.hasNext();) {
//			String string = (String) iterator.next();
//			tickets.remove(20);
//		}
		
		for(int i = 1; i <=10; i ++){
			new Thread("线程"+i){
				public void run(){
					while(true){
						if(tickets.isEmpty()) break;
						System.out.println(Thread.currentThread().getName() + "---" + tickets.remove(0));
					}
				}
			}.start();
		}
	}
}
```

# 并发类容器
&ensp;&emsp;&emsp;jdk5.0以后提供了多种并发类容器来替代同步类容器从而改善性能。**同步类容器都是串行化的。他们虽然实现了线程安全，但是严重降低了并发性，在多线程环境时，严重降低了应用程序的吞吐量**。
&ensp;&emsp;&emsp;**并发类容器**是专门针对并发设计的,使用**ConcurrentHashMap来代替给予散列的传统的HashTable**，而且在ConcurrentHashMap中，添加了一些常见复合操作的支持。以及使用了**CopyOnWriteArrayList代替Vector**，**并发的CopyOnWriteArraySet**,以及**并发的Queue**，***ConcurrentLinkedQueue***和***LinkedBlockingQueue***,前者是高性能队列，后者是以阻塞形式的队列，具体实现的Queue还有很多，例如ArrayBlockingQueue、PriorityBlockingQueue、SynchronousQueue等。

## ConcurrentMap
* ConcurrentHashMap
* ConcurrentSkipListMap(支持并发排序功能，弥补ConcurrentHashMap)

&ensp;&emsp;&emsp;**ConcurrentHashMap**内部使用**段（Segment）**来表示这些不同的部分，每个段其实就是一个小的**HashTable**,它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。把一个整体分成了16个段（Segment）。也就是**最高支持16个线程的并发修改操作**。这也是在多线程场景时**减小锁的粒度从而降低锁竞争**的一种方案。并且代码中大多共享变量使用volatile关键字声明，目的是第一时间获取修改的内容，性能非常好。

_UseConcurrentMap.java_

```java
public class UseConcurrentMap {

	public static void main(String[] args) {
		ConcurrentHashMap<String, Object> chm = new ConcurrentHashMap<String, Object>();
		chm.put("k1", "v1");
		chm.put("k2", "v2");
		chm.put("k3", "v3");
		chm.putIfAbsent("k4", "vvvv");
		//System.out.println(chm.get("k2"));
		//System.out.println(chm.size());
		
		for(Map.Entry<String, Object> me : chm.entrySet()){
			System.out.println("key:" + me.getKey() + ",value:" + me.getValue());
		}	
	}
}
```
## Copy-On-Write容器 
&ensp;&emsp;&emsp;Copy-On-Write简称COW，是一种用于程序设计中的优化策略。
&ensp;&emsp;&emsp;JDK里的COW容器有两种：
&ensp;&emsp;&emsp;**CopyOnWriteArrayList**和**CopyOnWriteArraySet**，COW容器非常有用，可以在非常多的并发场景中使用到。（使用场景：读多写少）
&ensp;&emsp;&emsp;什么是CopyOnWrite容器？
&ensp;&emsp;&emsp;**CopyOnWrite容器**即**写时复制**的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy,复制出一个新的容器，然后向新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的**好处**是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

_UseCopyOnWrite.java_

```java
public class UseCopyOnWrite {

	public static void main(String[] args) {
		CopyOnWriteArrayList<String> cwal = new CopyOnWriteArrayList<String>();
		CopyOnWriteArraySet<String> cwas = new CopyOnWriteArraySet<String>();
	}
}
```

# 并发Queue
&ensp;&emsp;&emsp;在并发队列上JDK提供了两套实现，一个是以**ConcurrentLinkedQueue**为代表的**高性能队列**，一个是以**BlockingQueue接口**为代表的**阻塞队列** ，无论哪种都继承自Queue。
![cp3](/Concurrent-Programming03/cp3.png)

## ConcurrentLinkedQueue
&ensp;&emsp;&emsp;**ConcurrentLinkedQueue**是一个适用于高并发场景下的队列，通常**无锁**的方式，实现了高并发状态下的高性能，通常**ConcurrentLinkedQueue性能好于BlockingQueue**。它是一个**基于链接节点的无界线程安全队列**。该队列的元素遵循先进先出的原则。头是最先加入的，尾是最近加入的，该队列**不允许null元素**。
&ensp;&emsp;&emsp;ConcurrentLinkedQueue重要方法：

*     add()和offer()都是加入元素的方法（在ConcurrentLinkedQueue中，这两个方法没有任何区别）
*     poll()和peek()都是取头元素节点，区别在于前者会删除元素，后者不会。

## BlockingQueue接口
**ArrayBlockingQueue：**基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，其内部没有实现读写分离，也就意味着生产和消费不能完全并行，长度是需要定义的，可以制定先进先出或者先进后出，也叫有界队列，在很多场合非常适合使用。

**LinkedBlockingQueue：**基于链表的阻塞队列，同ArrayBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），LinkedBlockingQueue之所以能够高效的处理并发数据，是因为其内部实现采用分离锁（读写分离两个锁），从而实现生产者和消费者操作的完全
并行运行。他是一个无界队列。

**SynchronousQueue：**一种没有缓冲的队列，生产者产生的数据直接会被消费者获取并消费。（不允许添加元素）

**PriorityBlockingQueue：**基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定，也就是说传入队列的对象必须实现Comparable接口），在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁，他也是一个无界的队列。

**DelayQueue：**带有延迟时间的Queue,其中的元素只有当其制定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue中的元素必须实现Delayed接口,DelayQueue是一个没有大小限制的队列，应用场景很多，比如对缓存超时的数据进行移除、任务超时处理、空闲连接的关闭等等。

_UseQueue.java_

```java
public class UseQueue {

	public static void main(String[] args) throws Exception {
		
		//高性能无阻塞无界队列：ConcurrentLinkedQueue
		/**
		ConcurrentLinkedQueue<String> q = new ConcurrentLinkedQueue<String>();
		q.offer("a");
		q.offer("b");
		q.offer("c");
		q.offer("d");
		q.add("e");
		
		System.out.println(q.poll());	//a 从头部取出元素，并从队列里删除
		System.out.println(q.size());	//4
		System.out.println(q.peek());	//b
		System.out.println(q.size());	//4
		*/
		//阻塞队列
		/**
		ArrayBlockingQueue<String> array = new ArrayBlockingQueue<String>(5);
		array.put("a");
		array.put("b");
		array.add("c");
		array.add("d");
		array.add("e");
		array.add("f");
		//System.out.println(array.offer("a", 3, TimeUnit.SECONDS));
		*/
		
		
		/**
		LinkedBlockingQueue<String> q = new LinkedBlockingQueue<String>();
		q.offer("a");
		q.offer("b");
		q.offer("c");
		q.offer("d");
		q.offer("e");
		q.add("f");
		//System.out.println(q.size());
		
//		for (Iterator iterator = q.iterator(); iterator.hasNext();) {
//			String string = (String) iterator.next();
//			System.out.println(string);
//		}
		
		List<String> list = new ArrayList<String>();
		System.out.println(q.drainTo(list, 3));
		System.out.println(list.size());
		for (String string : list) {
			System.out.println(string);
		}
		*/
		
		
		final SynchronousQueue<String> q = new SynchronousQueue<String>();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println(q.take());
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		t1.start();
		Thread t2 = new Thread(new Runnable() {
			
			@Override
			public void run() {
				q.add("asdasd");
			}
		});
		t2.start();		
	}
}
```

_Task.java_

```java
public class Task implements Comparable<Task>{
	
	private int id ;
	private String name;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	
	@Override
	public int compareTo(Task task) {
		return this.id > task.id ? 1 : (this.id < task.id ? -1 : 0);  
	}
	
	public String toString(){
		return this.id + "," + this.name;
	}
}
```

_UsePriorityBlockingQueue.java_

```java
public class UsePriorityBlockingQueue {
	public static void main(String[] args) throws Exception{
		PriorityBlockingQueue<Task> q = new PriorityBlockingQueue<Task>();
		
		Task t1 = new Task();
		t1.setId(3);
		t1.setName("id为3");
		Task t2 = new Task();
		t2.setId(4);
		t2.setName("id为4");
		Task t3 = new Task();
		t3.setId(1);
		t3.setName("id为1");
		
		//return this.id > task.id ? 1 : 0;
		q.add(t1);	//3
		q.add(t2);	//4
		q.add(t3);  //1
		
		// 1 3 4
		System.out.println("容器：" + q);
		System.out.println(q.take().getId());
		System.out.println("容器：" + q);
//		System.out.println(q.take().getId());
//		System.out.println(q.take().getId());
	}
}
```

_TimeUinitTest.java_

```java
public class TimeUinitTest {
     private TimeUnit timeUnit = TimeUnit.DAYS;
 
     public static void main(String[] TimeUinitTest) {
    	 TimeUinitTest tut = new TimeUinitTest();
    	 tut.outInfo();
     }
 
     public void outInfo() {
         System.out.println(timeUnit.name());
         System.out.println(timeUnit.toDays(1));
         System.out.println(timeUnit.toHours(1));
         System.out.println(timeUnit.toMinutes(1));
         System.out.println(timeUnit.toSeconds(1));
         System.out.println(timeUnit.toMillis(1));
         System.out.println(timeUnit.toMicros(1));
         System.out.println(timeUnit.toNanos(1));
         System.out.println((timeUnit.convert(1, TimeUnit.DAYS)) + timeUnit.name());
         System.out.println((timeUnit.convert(24, TimeUnit.HOURS)) + timeUnit.name());
         System.out.println((timeUnit.convert(1440, TimeUnit.MINUTES)) + timeUnit.name());
         System.out.println("-------------------");
     }
 }
```

_WangBa.java_

```java
public class WangBa implements Runnable {  
    
    private DelayQueue<Wangmin> queue = new DelayQueue<Wangmin>();  
    
    public boolean yinye =true;  
      
    public void shangji(String name,String id,int money){  
        Wangmin man = new Wangmin(name, id, 1000 * money + System.currentTimeMillis());  
        System.out.println("网名"+man.getName()+" 身份证"+man.getId()+"交钱"+money+"块,开始上机...");  
        this.queue.add(man);  
    }  
      
    public void xiaji(Wangmin man){  
        System.out.println("网名"+man.getName()+" 身份证"+man.getId()+"时间到下机...");  
    }  
  
    @Override  
    public void run() {  
        while(yinye){  
            try {  
                Wangmin man = queue.take();  
                xiaji(man);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
      
    public static void main(String args[]){  
        try{  
            System.out.println("网吧开始营业");  
            WangBa siyu = new WangBa();  
            Thread shangwang = new Thread(siyu);  
            shangwang.start();  
              
            siyu.shangji("路人甲", "123", 1);  
            siyu.shangji("路人乙", "234", 10);  
            siyu.shangji("路人丙", "345", 5);  
        }  
        catch(Exception e){  
            e.printStackTrace();
        }  
  
    }  
}
```

_Wangmin.java_

```java
public class Wangmin implements Delayed {  
    
    private String name;  
    //身份证  
    private String id;  
    //截止时间  
    private long endTime;  
    //定义时间工具类
    private TimeUnit timeUnit = TimeUnit.SECONDS;
      
    public Wangmin(String name,String id,long endTime){  
        this.name=name;  
        this.id=id;  
        this.endTime = endTime;  
    }  
      
    public String getName(){  
        return this.name;  
    }  
      
    public String getId(){  
        return this.id;  
    }  
      
    /** 
     * 用来判断是否到了截止时间 
     */  
    @Override  
    public long getDelay(TimeUnit unit) { 
        //return unit.convert(endTime, TimeUnit.MILLISECONDS) - unit.convert(System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    	return endTime - System.currentTimeMillis();
    }  
  
    /** 
     * 相互批较排序用 
     */  
    @Override  
    public int compareTo(Delayed delayed) {  
    	Wangmin w = (Wangmin)delayed;  
        return this.getDelay(this.timeUnit) - w.getDelay(this.timeUnit) > 0 ? 1:0;  
    }   
}
```


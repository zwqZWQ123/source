---
title: 互联网并发编程基础篇（二）
date: 2017-05-26 14:15:38
tags: 
- 多线程间通信
- ThreadLocal
- 单例&多线程

description: 互联网并发编程知识我们将进行第二部分的学习，在基础篇第二部分我们将掌握以下方面的内容：（1）并发编程下的多线程间通信概念wait、notify，线程经典面试题讲解 （2）模拟底层阻塞队列（queue）实现讲解 （3）单例和多线程如何结合使用

categories: 
- 多线程&并发编程
---

# 线程之间的通信

**线程通信概念：**线程是操作系统中独立的个体，但这些个体如果不经过特殊的处理就不能成为一个整体，线程间的通信就成为整体的必用方式之一。当线程存在通信指挥，系统间的交互性会更强大，在提高CPU利用率的同时还会使开发人员对线程任务在处理的过程中进行有效的把控与监督。

&ensp;&emsp;&emsp;使用**wait／notify**方法实现线程间的通信。（注意这两个方法都是Object类的方法，换句话说java为所有的对象都提供了这两个方法）。

1. wait和notify必须配合synchronized关键字使用
2. wait方法释放锁，notify方法不释放锁

_ListAdd1.java_

```java
public class ListAdd1 {

	
	private volatile static List list = new ArrayList();	
	
	public void add(){
		list.add("bjsxt");
	}
	public int size(){
		return list.size();
	}
	
	public static void main(String[] args) {
		
		final ListAdd1 list1 = new ListAdd1();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					for(int i = 0; i <10; i++){
						list1.add();
						System.out.println("当前线程：" + Thread.currentThread().getName() + "添加了一个元素..");
						Thread.sleep(500);
					}	
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				while(true){
					if(list1.size() == 5){
						System.out.println("当前线程收到通知：" + Thread.currentThread().getName() + " list size = 5 线程停止..");
						throw new RuntimeException();
					}
				}
			}
		}, "t2");		
		
		t1.start();
		t2.start();
	}	
}
```

_ListAdd2.java_

```java
public class ListAdd2 {
	private volatile static List list = new ArrayList();	
	
	public void add(){
		list.add("bjsxt");
	}
	public int size(){
		return list.size();
	}
	
	public static void main(String[] args) {
		
		final ListAdd2 list2 = new ListAdd2();
		
		// 1 实例化出来一个 lock
		// 当使用wait 和 notify 的时候 ， 一定要配合着synchronized关键字去使用
		//final Object lock = new Object();
		
		final CountDownLatch countDownLatch = new CountDownLatch(1);
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					//synchronized (lock) {
						for(int i = 0; i <10; i++){
							list2.add();
							System.out.println("当前线程：" + Thread.currentThread().getName() + "添加了一个元素..");
							Thread.sleep(500);
							if(list2.size() == 5){
								System.out.println("已经发出通知..");
								countDownLatch.countDown();
								//lock.notify();
							}
						}						
					//}
				} catch (InterruptedException e) {
					e.printStackTrace();
				}

			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				//synchronized (lock) {
					if(list2.size() != 5){
						try {
							//System.out.println("t2进入...");
							//lock.wait();
							countDownLatch.await();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
					System.out.println("当前线程：" + Thread.currentThread().getName() + "收到通知线程停止..");
					throw new RuntimeException();
				//}
			}
		}, "t2");
		t2.start();
		t1.start();
	}
}
```
**java.util.concurrent.CountDownLatch 实现线程间实时通信。
wait —— countDownLatch.await()
notify—— countDownLatch.notify()**

# 使用wait/notify模拟Queue
**BlockingQueue:**顾名思义，首先它是一个队列，并支持阻塞的机制，阻塞的放入和得到数据。我们要实现LinkedBlockingQueue下面两个简单的方法put和take。
**put(anObject):**把anObject加到BlockingQueue里，如果BlockingQueue没有空间，则调用此方法的线程被阻断，直到BlockingQueue里面有空间再继续。
**take:**取走BlockingQueue里排在首位的对象，若BlockingQueue为空，阻断进入等待状态直到BlockingQueue有新的数据被加入。

_MyQueue.java_

```java
public class MyQueue {
	
	//1 需要一个承装元素的集合 
	private LinkedList<Object> list = new LinkedList<Object>();
	
	//2 需要一个计数器
	private AtomicInteger count = new AtomicInteger(0);
	
	//3 需要制定上限和下限
	private final int minSize = 0;
	
	private final int maxSize ;
	
	//4 构造方法
	public MyQueue(int size){
		this.maxSize = size;
	}
	
	//5 初始化一个对象 用于加锁
	private final Object lock = new Object();
	
	
	//put(anObject): 把anObject加到BlockingQueue里,如果BlockQueue没有空间,则调用此方法的线程被阻断，直到BlockingQueue里面有空间再继续.
	public void put(Object obj){
		synchronized (lock) {
			while(count.get() == this.maxSize){
				try {
					lock.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			//1 加入元素
			list.add(obj);
			//2 计数器累加
			count.incrementAndGet();
			//3 通知另外一个线程（唤醒）
			lock.notify();
			System.out.println("新加入的元素为:" + obj);
		}
	}
	
	
	//take: 取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到BlockingQueue有新的数据被加入.
	public Object take(){
		Object ret = null;
		synchronized (lock) {
			while(count.get() == this.minSize){
				try {
					lock.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			//1 做移除元素操作
			ret = list.removeFirst();
			//2 计数器递减
			count.decrementAndGet();
			//3 唤醒另外一个线程
			lock.notify();
		}
		return ret;
	}
	
	public int getSize(){
		return this.count.get();
	}
	
	
	public static void main(String[] args) {
		
		final MyQueue mq = new MyQueue(5);
		mq.put("a");
		mq.put("b");
		mq.put("c");
		mq.put("d");
		mq.put("e");
		
		System.out.println("当前容器的长度:" + mq.getSize());
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				mq.put("f");
				mq.put("g");
			}
		},"t1");
		
		t1.start();
		
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				Object o1 = mq.take();
				System.out.println("移除的元素为:" + o1);
				Object o2 = mq.take();
				System.out.println("移除的元素为:" + o2);
			}
		},"t2");
		
		
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}	
}
```

# ThreadLocal
**ThreadLocal概念：**线程局部变量，是一种多线程间并发访问变量的解决方案 。与其synchronized等加锁的方式不同，**ThreadLocal完全不提供锁，而使用以空间换时间的手段，为每个线程提供变量的独立副本，以保证线程安全**。

&ensp;&emsp;&emsp;从性能上讲，ThreadLocal不具有绝对的优势，**在并发不是很高的时候，加锁的性能会更好**，但作为一套与锁完全无关的线程安全解决方案，在高并发量或者竞争激烈的场景，使用ThreadLocal可以在一定程度上减少锁竞争。

_ConnThreadLocal.java_

```java
public class ConnThreadLocal {

	public static ThreadLocal<String> th = new ThreadLocal<String>();
	
	public void setTh(String value){
		th.set(value);
	}
	public void getTh(){
		System.out.println(Thread.currentThread().getName() + ":" + this.th.get());
	}
	
	public static void main(String[] args) throws InterruptedException {
		
		final ConnThreadLocal ct = new ConnThreadLocal();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				ct.setTh("张三");
				ct.getTh();
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(1000);
					ct.getTh();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "t2");
		
		t1.start();
		t2.start();
	}
}
```
# 单例&多线程
**单例模式**，最常见的就是**饥饿模式**和**懒汉模式**，一个直接实例化对象，一个在调用方法时进行实例化对象。在多线程模式中，考虑到性能和线程安全问题，我们一般选择下面两种比较经典的单例模式，在性能提高的同时，又保证了线程安全。
**dubble check instance（双重if）**

_DubbleSingleton.java_

```java
public class DubbleSingleton {

	private static DubbleSingleton ds;
	
	public  static DubbleSingleton getDs(){
		if(ds == null){
			try {
				//模拟初始化对象的准备时间...
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			synchronized (DubbleSingleton.class) {
				if(ds == null){
					ds = new DubbleSingleton();
				}
			}
		}
		return ds;
	}
	
	public static void main(String[] args) {
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t2");
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t3");
		
		t1.start();
		t2.start();
		t3.start();
	}
}
```
**static inner class （静态内部类）**

_Singletion.java_

```java
public class Singletion {
	
	private static class InnerSingletion {
		private static Singletion single = new Singletion();
	}
	
	public static Singletion getInstance(){
		return InnerSingletion.single;
	}
	
}
```


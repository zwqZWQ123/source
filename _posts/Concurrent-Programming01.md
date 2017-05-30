---
title: 互联网并发编程基础篇（一）
date: 2017-05-25 18:47:37
tags: 
- 线程
- 对象锁
- 脏读
- Synchronized
- Volatile

description: 今天进入互联网并发编程知识的学习，在基础篇第一部分我们将掌握以下方面的内容：（1）线程基础概念、线程安全概念、多个线程多个锁概念（2）对象锁的同步和异步（3）脏读概念、脏读业务场景（4）Synchronized概念、Synchronized代码块、Synchronized其他细节（5）olatile关键字概念、线程优化执行流程、内部原理讲解、Volatile关键字的非原子性

categories: 
- 多线程&并发编程
---
# 线程安全

**线程安全：**当多个线程访问某一个类（对象或方法）时，这个类始终都能表现出正确的行为，那么这个类（对象或方法）就是线程安全的。
**synchronized:**可以在任意对象或方法上加锁，而加锁的这段代码称为“互斥区”或“临界区”。
_**MyThread.java**_

```java
public class MyThread extends Thread{
	
	private int count = 5 ;
	
	//synchronized加锁
	public void run(){
		count--;
		System.out.println(this.currentThread().getName() + " count = "+ count);
	}
	
	public static void main(String[] args) {
		/**
		 * 分析：当多个线程访问myThread的run方法时，以排队的方式进行处理（这里排对是按照CPU分配的先后顺序而定的），
		 * 		一个线程想要执行synchronized修饰的方法里的代码：
		 * 		1 尝试获得锁
		 * 		2 如果拿到锁，执行synchronized代码体内容；拿不到锁，这个线程就会不断的尝试获得这把锁，直到拿到为止，
		 * 		   而且是多个线程同时去竞争这把锁。（也就是会有锁竞争的问题）
		 */
		MyThread myThread = new MyThread();
		Thread t1 = new Thread(myThread,"t1");
		Thread t2 = new Thread(myThread,"t2");
		Thread t3 = new Thread(myThread,"t3");
		Thread t4 = new Thread(myThread,"t4");
		Thread t5 = new Thread(myThread,"t5");
		t1.start();
		t2.start();
		t3.start();
		t4.start();
		t5.start();
	}
}
```
**代码分析：**
&ensp;&emsp;&emsp;当多个线程访问myThread的run方法时，以排队的方式进行处理（这里排对是按照CPU分配的先后顺序而定的），一个线程想要执行synchronized修饰的方法里的代码：
1. 尝试获得锁
2. 如果拿到锁，执行synchronized代码体内容；拿不到锁，这个线程就会不断的尝试获得这把锁，直到拿到为止，而且是多个线程同时去竞争这把锁。（也就是会有锁竞争的问题）
**注：避免激烈的有锁竞争情况，因为当有锁竞争严重的时候，内存占用率过高，严重的情况容易产生宕机。**

# 多个线程多个锁
**多个线程多个锁：**多个线程，每个线程都可以拿到自己指定的锁，分别获得锁之后，执行synchronized方法体的内容。
_MultiThread.java_

```java
public class MultiThread {

	private int num = 0;
	
	/** static */
	public synchronized void printNum(String tag){
		try {
			
			if(tag.equals("a")){
				num = 100;
				System.out.println("tag a, set num over!");
				Thread.sleep(1000);
			} else {
				num = 200;
				System.out.println("tag b, set num over!");
			}
			
			System.out.println("tag " + tag + ", num = " + num);
			
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	//注意观察run方法输出顺序
	public static void main(String[] args) {
		
		//俩个不同的对象
		final MultiThread m1 = new MultiThread();
		final MultiThread m2 = new MultiThread();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				m1.printNum("a");
			}
		});
		
		Thread t2 = new Thread(new Runnable() {
			@Override 
			public void run() {
				m2.printNum("b");
			}
		});		
		
		t1.start();
		t2.start();
	}	
}
```
**代码分析：**
&ensp;&emsp;&emsp;关键字synchronized取得的锁都是对象锁，而不是把一段代码（方法）当作锁，所以代码中哪个线程先执行synchronized关键字的方法，哪个线程就持有该方法所属对象的锁（Lock）。
&ensp;&emsp;&emsp;有一种情况则是相同的锁，在**静态方法**上加synchronized关键字，表示锁定.class类，类一级别的锁（独占.class类）。

# 对象锁的同步和异步
**同步（synchronized）**
&ensp;&emsp;&emsp;同步的概念就是共享，如果不是共享的资源，就没有必要进行同步。
**异步（asynchronized）**
&ensp;&emsp;&emsp;异步的概念就是独立，相互之间不受到任何制约。就好像我们学习http的时候，在页面发起的Ajax请求，我们还可以继续浏览或操作页面的内容，二者之间没有任何关系。
&ensp;&emsp;&emsp;同步的目的就是为了线程安全，其实对于其他线程安全来说，需要满足两个特性：

* **原子性（同步）**
* **可见性**

_MyObject.java_

```java
public class MyObject {

	public synchronized void method1(){
		try {
			System.out.println(Thread.currentThread().getName());
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	/** synchronized */
	public void method2(){
			System.out.println(Thread.currentThread().getName());
	}
	
	public static void main(String[] args) {
		
		final MyObject mo = new MyObject();
		
		/**
		 * 分析：
		 * t1线程先持有object对象的Lock锁，t2线程可以以异步的方式调用对象中的非synchronized修饰的方法
		 * t1线程先持有object对象的Lock锁，t2线程如果在这个时候调用对象中的同步（synchronized）方法则需等待，也就是同步
		 */
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				mo.method1();
			}
		},"t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				mo.method2();
			}
		},"t2");
		
		t1.start();
		t2.start();
		
	}
	
}
```
**代码分析：**
&ensp;&emsp;&emsp;t1线程先持有object对象的Lock锁，t2线程可以以异步的方式调用对象中的非synchronized修饰的方法
&ensp;&emsp;&emsp;t1线程先持有object对象的Lock锁，t2线程如果在这个时候调用对象中的同步（synchronized）方法则需等待，也就是同步

# 脏读
&ensp;&emsp;&emsp;对于对象的同步和异步的方法，我们在设计自己的程序的时候，一定要考虑问题的整体，不然就会出现数据不一致的错误，很经典的错误就是**脏读（dirtyread）**。

_DirtyRead.java_

```java
public class DirtyRead {

	private String username = "bjsxt";
	private String password = "123";
	
	public synchronized void setValue(String username, String password){
		this.username = username;
		
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		this.password = password;
		
		System.out.println("setValue最终结果：username = " + username + " , password = " + password);
	}
	
	public void getValue(){
		System.out.println("getValue方法得到：username = " + this.username + " , password = " + this.password);
	}
	
	
	public static void main(String[] args) throws Exception{
		
		final DirtyRead dr = new DirtyRead();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				dr.setValue("z3", "456");		
			}
		});
		t1.start();
		Thread.sleep(1000);
		
		dr.getValue();
	}
}
```
&ensp;&emsp;&emsp;在我们对一个对象的方法加锁的时候，需要考虑业务的整体性，即为setValue/getValue方法同时加锁**synchronized**同步关键字，保证业务（service）的原子性，不然会出现业务错误（也从侧面保证业务的一致性）。

关系型数据库事务的四种特性ACID:

* **原子性**
* **一致性**
* **隔离性**
* **永久性**
举例一致性（consistancy）:

![cp1](/Concurrent-Programming01/cp1.png)
&ensp;&emsp;&emsp;Oracle数据库——undo（类似于日志） 如果从undo中找不到值，则会报出snapshotToOld异常。

# synchronized其他概念

&ensp;&emsp;&emsp;关键字synchronized拥有**锁重入**的功能,也就是在使用synchronized时，当一个线程得到一个对象的锁后，再次请求此对象时是可以再次得到该对象的锁。
_SyncDubbo1.java_

```java
public class SyncDubbo1 {

	public synchronized void method1(){
		System.out.println("method1..");
		method2();
	}
	public synchronized void method2(){
		System.out.println("method2..");
		method3();
	}
	public synchronized void method3(){
		System.out.println("method3..");
	}
	
	public static void main(String[] args) {
		final SyncDubbo1 sd = new SyncDubbo1();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				sd.method1();
			}
		});
		t1.start();
	}
}
```

_SyncDubbo2.java_

```java
public class SyncDubbo2 {

	static class Main {
		public int i = 10;
		public synchronized void operationSup(){
			try {
				i--;
				System.out.println("Main print i = " + i);
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	static class Sub extends Main {
		public synchronized void operationSub(){
			try {
				while(i > 0) {
					i--;
					System.out.println("Sub print i = " + i);
					Thread.sleep(100);		
					this.operationSup();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				Sub sub = new Sub();
				sub.operationSub();
			}
		});
		
		t1.start();
	}
}
```

&ensp;&emsp;&emsp;若出现异常，锁自动释放。
_SyncException.java_

```java
public class SyncException {

	private int i = 0;
	public synchronized void operation(){
		while(true){
			try {
				i++;
				Thread.sleep(100);
				System.out.println(Thread.currentThread().getName() + " , i = " + i);
				if(i == 20){
					//Integer.parseInt("a");
					throw new RuntimeException();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		
		final SyncException se = new SyncException();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				se.operation();
			}
		},"t1");
		t1.start();
	}
}
```
&ensp;&emsp;&emsp;适用于处理多任务，需要同时成功或失败，回滚；或者不需要同时成功或失败，失败的需要日志。

# synchronized代码块
&ensp;&emsp;&emsp;使用**synchronized**声明的方法在某些情况下是有弊端的，比如A线程调用同步的方法执行一个很长时间的任务，那么B线程就必须等待比较长的时间才能执行，这样的情况下可以使用synchronized代码块去优化代码执行时间，也就是通常所说的**减小锁的粒度**。
_Optimize.java_

```java
public class Optimize {

	public void doLongTimeTask(){
		try {
			
			System.out.println("当前线程开始：" + Thread.currentThread().getName() + 
					", 正在执行一个较长时间的业务操作，其内容不需要同步");
			Thread.sleep(2000);
			
			synchronized(this){
				System.out.println("当前线程：" + Thread.currentThread().getName() + 
					", 执行同步代码块，对其同步变量进行操作");
				Thread.sleep(1000);
			}
			System.out.println("当前线程结束：" + Thread.currentThread().getName() +
					", 执行完毕");
			
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		final Optimize otz = new Optimize();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				otz.doLongTimeTask();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				otz.doLongTimeTask();
			}
		},"t2");
		t1.start();
		t2.start();
	}
}
```
&ensp;&emsp;&emsp;**synchronized**可以使用任意的object进行加锁，用法比较灵活。
_ObjectLock.java_

```java
public class ObjectLock {

	public void method1(){
		synchronized (this) {	//对象锁
			try {
				System.out.println("do method1..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public void method2(){		//类锁
		synchronized (ObjectLock.class) {
			try {
				System.out.println("do method2..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	private Object lock = new Object();
	public void method3(){		//任何对象锁
		synchronized (lock) {
			try {
				System.out.println("do method3..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	
	public static void main(String[] args) {
		
		final ObjectLock objLock = new ObjectLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method1();
			}
		});
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method2();
			}
		});
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method3();
			}
		});
		t1.start();
		t2.start();
		t3.start();
	}
}
```

&ensp;&emsp;&emsp;另一个特别注意一个问题，就是不要使用String的常量加锁，会出现死循环问题。

_StringLock.java_

```java
public class StringLock {

	public void method() {
		//new String("字符串常量")
		synchronized ("字符串常量") {
			try {
				while(true){
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
					Thread.sleep(1000);		
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		final StringLock stringLock = new StringLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				stringLock.method();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				stringLock.method();
			}
		},"t2");
		
		t1.start();
		t2.start();
	}
}
```
&ensp;&emsp;&emsp;锁对象的改变问题，当使用一个对象进行加锁的时候，要注意对象本身发生改变的时候，那么持有的锁就不同。如果对象本身不发生改变，那么依然是同步的，即使是对象的属性发生了改变。

_ChangeLock.java_

```java
public class ChangeLock {

	private String lock = "lock";
	
	private void method(){
		synchronized (lock) {
			try {
				System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
				lock = "change lock";
				Thread.sleep(2000);
				System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
	
		final ChangeLock changeLock = new ChangeLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				changeLock.method();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				changeLock.method();
			}
		},"t2");
		t1.start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
}
```

_ModifyLock.java_

```java
public class ModifyLock {
	
	private String name ;
	private int age ;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public synchronized void changeAttributte(String name, int age) {
		try {
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 开始");
			this.setName(name);
			this.setAge(age);
			
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 修改对象内容为： " 
					+ this.getName() + ", " + this.getAge());
			
			Thread.sleep(2000);
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 结束");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		final ModifyLock modifyLock = new ModifyLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				modifyLock.changeAttributte("张三", 20);
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				modifyLock.changeAttributte("李四", 21);
			}
		},"t2");
		
		t1.start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
}
```
&ensp;&emsp;&emsp;死锁问题，在设计程序时就应该避免双方持有对方的锁的情况：

_DeadLock.java_

```java  
public class DeadLock implements Runnable{

	private String tag;
	private static Object lock1 = new Object();
	private static Object lock2 = new Object();
	
	public void setTag(String tag){
		this.tag = tag;
	}
	
	@Override
	public void run() {
		if(tag.equals("a")){
			synchronized (lock1) {
				try {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock1执行");
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				synchronized (lock2) {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock2执行");
				}
			}
		}
		if(tag.equals("b")){
			synchronized (lock2) {
				try {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock2执行");
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				synchronized (lock1) {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock1执行");
				}
			}
		}
	}
	
	public static void main(String[] args) {
		
		DeadLock d1 = new DeadLock();
		d1.setTag("a");
		DeadLock d2 = new DeadLock();
		d2.setTag("b");
		 
		Thread t1 = new Thread(d1, "t1");
		Thread t2 = new Thread(d2, "t2");
		 
		t1.start();
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
}
```

# volatile关键字

&ensp;&emsp;&emsp;**voletile关键字**的主要作用是使变量在多个线程间可见。

_RunThread.java_

```java
public class RunThread extends Thread{

	private volatile boolean isRunning = true;
	private void setRunning(boolean isRunning){
		this.isRunning = isRunning;
	}
	
	public void run(){
		System.out.println("进入run方法..");
		int i = 0;
		while(isRunning == true){
			//..
		}
		System.out.println("线程停止");
	}
	
	public static void main(String[] args) throws InterruptedException {
		RunThread rt = new RunThread();
		rt.start();
		Thread.sleep(1000);
		rt.setRunning(false);
		System.out.println("isRunning的值已经被设置了false");
	}
}
```
&ensp;&emsp;&emsp;JVM为线程单独保留一块内存，储存主线程中的变量，即示例中的isRunning。而加了**volatile关键字实现了isRunning在线程和主线程之间的可见性**。
&ensp;&emsp;&emsp;**详解：**在java中，每一个线程都会有一块工作内存区，其中存放着所有线程共享的主内存中的变量值的拷贝。当线程执行时，他在自己的工作内存区中操作这些变量。为了存取一个共享的变量，一个线程通常先获取锁定并去清除它的内存工作区，把这些共享变量从所有线程的共享内存区中正确的装入到他自己的工作内存区中，当线程解锁时保证该工作内存区中变量的值写回到共享内存中。
&ensp;&emsp;&emsp;一个线程可以执行的操作有**使用（use）、赋值（assign）、装载（load）、存储（store）、锁定（lock）、解锁（unlock）**。
&ensp;&emsp;&emsp;而主内存可以执行的操作有**读（read）、写（write）、锁定（lock）、解锁（unlock）**,每个操作都是原子性的。
&ensp;&emsp;&emsp;volatile的作用就是强制线程到主内存（共享内存）里去读取变量，而不去线程工作内存区里去读取，从而实现了多个线程间的变量可见，也就是满足线程安全的可见性。

**Volatile关键字（线程执行流程图）**

![cp2](/Concurrent-Programming01/cp2.png)

**注：**volatile关键字具有**可见性**，但不具有synchronized关键字的**原子性（同步）**。
&ensp;&emsp;&emsp;volatile关键字虽然拥有多个线程的可见性，但是却不具备同步性（也就是原子性），可以算上是一个轻量级的synchronized，性能要比synchronized强很多，不会造成阻塞（在很多开源的架构里，比如netty的底层代码就是大量使用volatile，可见netty性能一定是非常不错的）这里需要注意：一般volatile用于只针对多个线程可见的变量操作，并不能代替synchronized的同步功能。

_VolatileNoAtomic.java_

```java
public class VolatileNoAtomic extends Thread{
	//private static volatile int count;
	private static AtomicInteger count = new AtomicInteger(0);
	private static void addCount(){
		for (int i = 0; i < 1000; i++) {
			//count++ ;
			count.incrementAndGet();
		}
		System.out.println(count);
	}
	
	public void run(){
		addCount();
	}
	
	public static void main(String[] args) {
		
		VolatileNoAtomic[] arr = new VolatileNoAtomic[100];
		for (int i = 0; i < 10; i++) {
			arr[i] = new VolatileNoAtomic();
		}
		
		for (int i = 0; i < 10; i++) {
			arr[i].start();
		}
	}
}
```
&ensp;&emsp;&emsp;volatile关键字只具有可见性，没有原子性。要实现原子性建议使用atomic类的系列对象，支持原子性操作（注意atomic类只保证本身方法原子性，并不保证多次操作的原子性）

_AtomicUse.java_

```java
public class AtomicUse {

	private static AtomicInteger count = new AtomicInteger(0);
	
	//多个addAndGet在一个方法内是非原子性的，需要加synchronized进行修饰，保证4个addAndGet整体原子性
	/**synchronized*/
	public synchronized int multiAdd(){
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			count.addAndGet(1);
			count.addAndGet(2);
			count.addAndGet(3);
			count.addAndGet(4); //+10
			return count.get();
	}
	
	
	public static void main(String[] args) {
		
		final AtomicUse au = new AtomicUse();

		List<Thread> ts = new ArrayList<Thread>();
		for (int i = 0; i < 100; i++) {
			ts.add(new Thread(new Runnable() {
				@Override
				public void run() {
					System.out.println(au.multiAdd());
				}
			}));
		}

		for(Thread t : ts){
			t.start();
		}		
	}
}
```

&ensp;&emsp;&emsp;多个addAndGet在一个方法内是非原子性的，需要加synchronized进行修饰，保证4个addAndGet整体原子性。


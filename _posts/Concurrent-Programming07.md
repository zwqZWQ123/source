---
title: 互联网并发编程高级篇（三）
date: 2017-05-29 16:16:24
tags:
- ReentrantLock(重入锁)
- ReentrantReadWriteLock（读写锁）

description: 继续学习互联网并发编程知识的高级篇，在高级篇第三部分我们将掌握以下方面的内容：（1）并发编程中的重入锁使用 （2）并发编程中的读写分离锁使用

categories: 
- 多线程&并发编程
---
# 锁
&ensp;&emsp;&emsp;在java多线程中，我们知道可以使用**synchronized关键字**来实现线程间的同步互斥工作，那么其实还有一个更优秀的机制去完成这个“同步互斥”工作，他就是**Lock对象**，我们主要学习两种锁，**重入锁**和**读写锁**。他们具有比synchronized更为强大的功能，并且有**嗅探锁定**、**多路分支**等功能。

# 重入锁
&ensp;&emsp;&emsp;**重入锁**，在需要进行同步的代码部分加上锁定，但不要忘记最后一定要释放锁定，不然会造成锁永远无法释放，其他线程永远进不来的结果。
_UseReentrantLock.java_

```java
public class UseReentrantLock {
	
	private Lock lock = new ReentrantLock();
	
	public void method1(){
		try {
			lock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入method1..");
			Thread.sleep(1000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出method1..");
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			
			lock.unlock();
		}
	}
	
	public void method2(){
		try {
			lock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入method2..");
			Thread.sleep(2000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出method2..");
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			
			lock.unlock();
		}
	}
	
	public static void main(String[] args) {

		final UseReentrantLock ur = new UseReentrantLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				ur.method1();
				ur.method2();
			}
		}, "t1");

		t1.start();
		try {
			Thread.sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		//System.out.println(ur.lock.getQueueLength());
	}
}
```

# 锁与等待／通知
&ensp;&emsp;&emsp;还记得我们在使用synchronized的时候,如果需要多线程间进行协作工作则需要Object的wait()和notify()、notifyAll()方法进行配合工作。
&ensp;&emsp;&emsp;那么同样，我们在使用Lock的时候，可以使用一个新的等待／通知的类，它就是**Condition**。这个Condition一定是针对具体某一把锁的。也就是在只有锁的基础之上才会产生Condition。

_UseCondition.java_

```java
public class UseCondition {

	private Lock lock = new ReentrantLock();
	private Condition condition = lock.newCondition();
	
	public void method1(){
		try {
			lock.lock();
			System.out.println("当前线程：" + Thread.currentThread().getName() + "进入等待状态..");
			Thread.sleep(3000);
			System.out.println("当前线程：" + Thread.currentThread().getName() + "释放锁..");
			condition.await();	// Object wait
			System.out.println("当前线程：" + Thread.currentThread().getName() +"继续执行...");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void method2(){
		try {
			lock.lock();
			System.out.println("当前线程：" + Thread.currentThread().getName() + "进入..");
			Thread.sleep(3000);
			System.out.println("当前线程：" + Thread.currentThread().getName() + "发出唤醒..");
			condition.signal();		//Object notify
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public static void main(String[] args) {
		
		final UseCondition uc = new UseCondition();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				uc.method1();
			}
		}, "t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				uc.method2();
			}
		}, "t2");
		t1.start();

		t2.start();
	}
}
```

# 多Condition
&ensp;&emsp;&emsp;我们可以通过一个Lock对象产生多个Condition进行多线程间的交互，非常的灵活。可以使得部分需要唤醒的线程唤醒，其他线程则继续等待通知。

_UseManyCondition.java_

```java
public class UseManyCondition {

	private ReentrantLock lock = new ReentrantLock();
	private Condition c1 = lock.newCondition();
	private Condition c2 = lock.newCondition();
	
	public void m1(){
		try {
			lock.lock();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "进入方法m1等待..");
			c1.await();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "方法m1继续..");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void m2(){
		try {
			lock.lock();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "进入方法m2等待..");
			c1.await();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "方法m2继续..");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void m3(){
		try {
			lock.lock();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "进入方法m3等待..");
			c2.await();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "方法m3继续..");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void m4(){
		try {
			lock.lock();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "唤醒..");
			c1.signalAll();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void m5(){
		try {
			lock.lock();
			System.out.println("当前线程：" +Thread.currentThread().getName() + "唤醒..");
			c2.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public static void main(String[] args) {
		
		
		final UseManyCondition umc = new UseManyCondition();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				umc.m1();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				umc.m2();
			}
		},"t2");
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				umc.m3();
			}
		},"t3");
		Thread t4 = new Thread(new Runnable() {
			@Override
			public void run() {
				umc.m4();
			}
		},"t4");
		Thread t5 = new Thread(new Runnable() {
			@Override
			public void run() {
				umc.m5();
			}
		},"t5");
		
		t1.start();	// c1
		t2.start();	// c1
		t3.start();	// c2
		

		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

		t4.start();	// c1
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t5.start();	// c2	
	}
}
```
# Lock/Condition其他方法和用法
**公平锁和非公平锁：**
Lock lock = new ReentrantLock(boolean isFair);
**lock用法：**
tryLock():尝试获得锁，获得结果用true/false返回。
tryLock():在给定的时间内尝试获得锁，获得结果用true/false返回。
isFair():是否是公平锁。
isLocked():是否锁定。
getHoldCount():查询当前线程保持此锁的个数，也就是调用lock()次数。
lockInterruptibly()：优先响应中断的锁。
getQueueLength():返回正在等待获取此锁定的线程数。
getWaitQueueLength():返回等待与锁定相关的给定条件Condition的线程数。
hasQueuedThread(Thread thread):查询指定的线程是否正在等待此锁。
hasQueueThreads():查询是否有线程正在等待此锁。
hasWaiters():查询是否有线程正在等待与此锁定有关的condition条件。

_TestHoldCount.java_

```java
public class TestHoldCount {

	//重入锁
	private ReentrantLock lock = new ReentrantLock();
	
	public void m1(){
		try {
			lock.lock();
			System.out.println("进入m1方法，holdCount数为：" + lock.getHoldCount());
			
			//调用m2方法
			m2();
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void m2(){
		try {
			lock.lock();
			System.out.println("进入m2方法，holdCount数为：" + lock.getHoldCount());
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	
	public static void main(String[] args) {
		TestHoldCount thc = new TestHoldCount();
		thc.m1();
	}
}
```

# ReentrantReadWriteLock（读写锁)
&ensp;&emsp;&emsp;**读写锁ReentrantReadWriteLock**，其核心就是实现读写分离的锁。在高并发访问下，尤其是**读多写少**的情况下，性能要远高于重入锁。
&ensp;&emsp;&emsp;之前学synchronized、ReentrantLock时，我们知道，同一时间内，只能有一个线程进行访问被锁定的代码，那么读写锁则不同，其本质是分成两个锁，即读锁、写锁。在读锁下，多个线程可以并发的进行访问，但在写锁的时候，只能一个一个的顺序访问。
&ensp;&emsp;&emsp;口诀：**读读共享，写写互斥，读写互斥**。

_UseReentrantReadWriteLock.java_

```java
public class UseReentrantReadWriteLock {

	private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
	private ReadLock readLock = rwLock.readLock();
	private WriteLock writeLock = rwLock.writeLock();
	
	public void read(){
		try {
			readLock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入...");
			Thread.sleep(3000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出...");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			readLock.unlock();
		}
	}
	
	public void write(){
		try {
			writeLock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入...");
			Thread.sleep(3000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出...");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			writeLock.unlock();
		}
	}
	
	public static void main(String[] args) {
		
		final UseReentrantReadWriteLock urrw = new UseReentrantReadWriteLock();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.read();
			}
		}, "t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.read();
			}
		}, "t2");
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.write();
			}
		}, "t3");
		Thread t4 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.write();
			}
		}, "t4");		
		
//		t1.start();
//		t2.start();
		
//		t1.start(); // R 
//		t3.start(); // W
		
		t3.start();
		t4.start();		
	}
}
```

# 锁优化总结
1. 避免死锁
2. 减小锁的持有时间
3. 减少锁的粒度
4. 锁的分离
5. 尽量使用无锁的操作，如原子操作（Atomic系列类）,volatile关键字




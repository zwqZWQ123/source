---
title: 互联网并发编程高级篇（一）
date: 2017-05-29 11:35:26
tags:
- Executor框架
- 自定义线程池

description: 今天进入互联网并发编程知识的学习高级篇，在高级篇第一部分我们将掌握以下方面的内容：（1）JDK多任务执行框架底层讲解与内部实现 （2）默认线程池说明、底层代码讲解 （3）自定义线程池说明、底层代码讲解（4）线程池拒绝策略讲解

categories: 
- 多线程&并发编程
---
# Executor框架
&ensp;&emsp;&emsp;为了更好的控制多线程，JDK提供了一套**线程框架Executor**，帮助开发人员有效地进行线程控制。它们都在**java.util.concurrent包**中,是JDK并发包的核心。其中又有一个比较重要的类：**Executors**，他扮演着**线程工厂**的角色，我们**通过Executors可以创建特定功能的线程池**。
&ensp;&emsp;&emsp;***Executors创建线程池方法***：

1. **newFixedThreadPool()**方法，该方法返回一个固定数量的线程池，该方法的线程数始终不变，当有一个任务提交时，若线程池中空闲，则立即执行，若没有，则会被暂缓在一个任务队列中等待有空闲的线程去执行。
2. **newSingleThreadExecutor()**方法，创建一个线程的线程池，若空闲则执行，若没有空闲线程则暂缓在任务队列中。
3. **newCachedThreadPool()**方法，返回一个可根据实际情况调整线程个数的线程池，不限制最大线程数量，若用空闲的线程则执行任务，若无任务则不创建线程。并且每一个空闲线程会在60秒后自动回收。
4. **newScheduleThreadPool()**方法，该方法返回一个ScheduledExecutorService对象，但该线程池可以指定线程的数量。

&ensp;&emsp;&emsp;源码中可见都是由**ThreadPoolExecutor**构建的。除了 **newScheduleThreadPool()**,是由**ScheduleThreadPoolExecutor**构建的，但源码中可见也是由**ThreadPoolExecutor**构建的。

_ScheduledJob.java_

```java
class Temp extends Thread {
    public void run() {
        System.out.println("run");
    }
}

public class ScheduledJob {
	
    public static void main(String args[]) throws Exception {
    
    	Temp command = new Temp();
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        
        ScheduledFuture<?> scheduleTask = scheduler.scheduleWithFixedDelay(command, 5, 1, TimeUnit.SECONDS);
    
    }
}
```
# 自定义线程池
&ensp;&emsp;&emsp;若**Executors工厂类**无法满足我们的需求，可以自己去创建自定义的线程池，其实Executors工厂类里面的创建线程方法其内部实现均是用了**ThreadPoolExecutor**这个类，这个类可以自定义线程。构造方法如下：
public ThreadPoolExecutor(
**int corePoolSize**,//当前核心线程数,表示时这个线程池创建时初始化的数量
                                            **int maximumPoolSize**,//最大线程数
                                            **long  keepAliveTime**,//线程池中每个线程保持空闲的时间
                                            **TimeUnit unit**,//时间戳 
                                            **BlockingQueue<Runnable> workQueue**,
                                            **ThreadFactory threadFactory**,
                                            **RejectedExecutionHandler handler**
                                            ){...}
                                            
# 自定义线程池使用详细
&ensp;&emsp;&emsp;这个构造方法对于队列是什么类型的比较关键：
&ensp;&emsp;&emsp;**在使用有界队列时：**若有新的任务需要执行，如果线程池实际线程数小于corePoolSize，则优先创建线程，若大于corePoolSize，则会将任务加入队列，若队列已满，则在总线程数不大于maximumPoolSize的前提下，创建新的线程，若线程数大于maximumPoolSize，则执行拒绝策略。或其它自定义方式。
&ensp;&emsp;&emsp;**无界的任务队列时：**LinkedBlockingQueue。与有界队列相比，除非系统资源耗尽，否则无界的任务队列不存在任务入队失败的情况。当有新任务到来，系统的线程数小于corePoolSize时，则新建线程执行任务。当达到corePoolSize后，就不会继续增加。若后续仍有新的任务加入，而又没有空闲的线程资源，则任务直接进入队列等待。若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存。
&ensp;&emsp;&emsp;**JDK拒绝策略**：

* **AbortPolicy:**直接抛出异常阻止系统正常工作。
* **CallerRunsPolicy:**只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。
* **DiscardOldestPolicy:**丢弃最老的一个请求，尝试再次提交当前任务。
* **DiscardPolicy:**丢弃无法处理的任务，不给予任何处理。
&ensp;&emsp;&emsp;如果需要自定义拒绝策略可以实现RejectedExecutionHandler接口。

_MyRejected.java_

```java
public class MyRejected implements RejectedExecutionHandler{

	
	public MyRejected(){
	}
	
	@Override
	public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
		System.out.println("自定义处理..");
		System.out.println("当前被拒绝任务为：" + r.toString());
	}
}
```                   

_MyTask.java_

```java
public class MyTask implements Runnable {

	private int taskId;
	private String taskName;
	
	public MyTask(int taskId, String taskName){
		this.taskId = taskId;
		this.taskName = taskName;
	}
	
	public int getTaskId() {
		return taskId;
	}

	public void setTaskId(int taskId) {
		this.taskId = taskId;
	}

	public String getTaskName() {
		return taskName;
	}

	public void setTaskName(String taskName) {
		this.taskName = taskName;
	}

	@Override
	public void run() {
		try {
			System.out.println("run taskId =" + this.taskId);
			Thread.sleep(5*1000);
			//System.out.println("end taskId =" + this.taskId);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}		
	}
	
	public String toString(){
		return Integer.toString(this.taskId);
	}

}
```

_UseThreadPoolExecutor1.java_

```java
public class UseThreadPoolExecutor1 {


	public static void main(String[] args) {
		/**
		 * 在使用有界队列时，若有新的任务需要执行，如果线程池实际线程数小于corePoolSize，则优先创建线程，
		 * 若大于corePoolSize，则会将任务加入队列，
		 * 若队列已满，则在总线程数不大于maximumPoolSize的前提下，创建新的线程，
		 * 若线程数大于maximumPoolSize，则执行拒绝策略。或其他自定义方式。
		 * 
		 */	
		ThreadPoolExecutor pool = new ThreadPoolExecutor(
				1, 				//coreSize
				2, 				//MaxSize
				60, 			//60
				TimeUnit.SECONDS, 
				new ArrayBlockingQueue<Runnable>(3)			//指定一种队列 （有界队列）
				//new LinkedBlockingQueue<Runnable>()
				, new MyRejected()
				//, new DiscardOldestPolicy()
				);
		
		MyTask mt1 = new MyTask(1, "任务1");
		MyTask mt2 = new MyTask(2, "任务2");
		MyTask mt3 = new MyTask(3, "任务3");
		MyTask mt4 = new MyTask(4, "任务4");
		MyTask mt5 = new MyTask(5, "任务5");
		MyTask mt6 = new MyTask(6, "任务6");
		
		pool.execute(mt1);
		pool.execute(mt2);
		pool.execute(mt3);
		pool.execute(mt4);
		pool.execute(mt5);
		pool.execute(mt6);
		
		pool.shutdown();
	}
}
```

_UseThreadPoolExecutor2.java_

```java
public class UseThreadPoolExecutor2 implements Runnable{

	private static AtomicInteger count = new AtomicInteger(0);
	
	@Override
	public void run() {
		try {
			int temp = count.incrementAndGet();
			System.out.println("任务" + temp);
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) throws Exception{
		//System.out.println(Runtime.getRuntime().availableProcessors());
		BlockingQueue<Runnable> queue = 
				//new LinkedBlockingQueue<Runnable>();
				new ArrayBlockingQueue<Runnable>(10);
		ExecutorService executor  = new ThreadPoolExecutor(
					5, 		//core
					10, 	//max
					120L, 	//2fenzhong
					TimeUnit.SECONDS,
					queue);
		
		for(int i = 0 ; i < 20; i++){
			executor.execute(new UseThreadPoolExecutor2());
		}
		Thread.sleep(1000);
		System.out.println("queue size:" + queue.size());		//10
		Thread.sleep(2000);
	}
}
```




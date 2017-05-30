---
title: 互联网并发编程高级篇（二）
date: 2017-05-29 15:54:19
tags: 
- CyclicBarrier
- CountDownLatch
- Callable
- Future
- Semaphore信号量

description: 继续学习互联网并发编程知识的高级篇，在高级篇第二部分我们将掌握以下方面的内容：（1）并发编程中的CountDownLatch与CyclicBarrier的使用 （2）并发编程中Future和Callable使用 （3）互联网进行限流策略的Semaphore信号量使用

categories: 
- 多线程&并发编程
---
# Concurrent.util常用类

## CyclicBarrier使用
&ensp;&emsp;&emsp;假设有只有一个场景：每个线程代表一个跑步运动员，当运动员都准备好后，才一起出发，只要有一个人没有准备好，大家都等待。

_UseCyclicBarrier.java_

```java
public class UseCyclicBarrier {

	static class Runner implements Runnable {  
	    private CyclicBarrier barrier;  
	    private String name;  
	    
	    public Runner(CyclicBarrier barrier, String name) {  
	        this.barrier = barrier;  
	        this.name = name;  
	    }  
	    @Override  
	    public void run() {  
	        try {  
	            Thread.sleep(1000 * (new Random()).nextInt(5));  
	            System.out.println(name + " 准备OK.");  
	            barrier.await();  
	        } catch (InterruptedException e) {  
	            e.printStackTrace();  
	        } catch (BrokenBarrierException e) {  
	            e.printStackTrace();  
	        }  
	        System.out.println(name + " Go!!");  
	    }  
	} 
	
    public static void main(String[] args) throws IOException, InterruptedException {  
        CyclicBarrier barrier = new CyclicBarrier(3);  // 3 
        ExecutorService executor = Executors.newFixedThreadPool(3);  
        
        executor.submit(new Thread(new Runner(barrier, "zhangsan")));  
        executor.submit(new Thread(new Runner(barrier, "lisi")));  
        executor.submit(new Thread(new Runner(barrier, "wangwu"))); 
        executor.shutdown();  
    }   
} 
```

## CountDownLatch使用
&ensp;&emsp;&emsp;它经常用于监听某些初始化操作，等初始化执行完毕后，通知主线程继续工作。

_UseCountDownLatch.java_

```java
public class UseCountDownLatch {

	public static void main(String[] args) {
		
		final CountDownLatch countDown = new CountDownLatch(2);
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("进入线程t1" + "等待其他线程处理完成...");
					countDown.await();
					System.out.println("t1线程继续执行...");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		},"t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("t2线程进行初始化操作...");
					Thread.sleep(3000);
					System.out.println("t2线程初始化完毕，通知t1线程继续...");
					countDown.countDown();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("t3线程进行初始化操作...");
					Thread.sleep(4000);
					System.out.println("t3线程初始化完毕，通知t1线程继续...");
					countDown.countDown();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		
		t1.start();
		t2.start();
		t3.start();
	}
}
```

***注意：CyclicBarrier和CountDownLatch的区别。***

* **CountDownLatch：**一个线程等待，n个线程发出通知，一个线程执行。
* **CyclicBarrier：**几个线程都参与阻塞

## Callable和Future使用
&ensp;&emsp;&emsp;以下例子其实就是我们之前实现的Future模式。jdk给予我们一个实现的封装，使用非常简单。

_UseFuture.java_

```java
public class UseFuture implements Callable<String>{
	private String para;
	
	public UseFuture(String para){
		this.para = para;
	}
	
	/**
	 * 这里是真实的业务逻辑，其执行可能很慢
	 */
	@Override
	public String call() throws Exception {
		//模拟执行耗时
		Thread.sleep(5000);
		String result = this.para + "处理完成";
		return result;
	}
	
	//主控制函数
	public static void main(String[] args) throws Exception {
		String queryStr = "query";
		//构造FutureTask，并且传入需要真正进行业务逻辑处理的类,该类一定是实现了Callable接口的类
		FutureTask<String> future = new FutureTask<String>(new UseFuture(queryStr));
		
		FutureTask<String> future2 = new FutureTask<String>(new UseFuture(queryStr));
		//创建一个固定线程的线程池且线程数为1,
		ExecutorService executor = Executors.newFixedThreadPool(2);
		//这里提交任务future,则开启线程执行RealData的call()方法执行
		//submit和execute的区别： 第一点是submit可以传入实现Callable接口的实例对象， 第二点是submit方法有返回值
		
		Future f1 = executor.submit(future);		//单独启动一个线程去执行的
		Future f2 = executor.submit(future2);
		System.out.println("请求完毕");
		
		try {
			//这里可以做额外的数据操作，也就是主程序执行其他业务逻辑
			System.out.println("处理实际的业务逻辑...");
			Thread.sleep(1000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		//调用获取数据方法,如果call()方法没有执行完成,则依然会进行等待
		System.out.println("数据：" + future.get());
		System.out.println("数据：" + future2.get());
		
		executor.shutdown();
	}

}
```

&ensp;&emsp;&emsp;**Future模式非常适合在处理很耗时很长的业务逻辑时进行使用，可以有效的减小系统的响应时间，提高系统的吞吐量。**

&ensp;&emsp;&emsp;**线程池Executor的submit和execute的区别**：

* 第一点是submit可以传入实现Callable接口的实例对象;
* 第二点是submit方法有返回值。

## Semaphore信号量使用
&ensp;&emsp;&emsp;在**Semaphore信号量**非常适合**高并发访问**，新系统在上线之前，要对系统的访问量进行评估，当然这个值肯定不是随便拍拍脑袋就能想出来的，是经过以往的经验、数据、历年的访问量，已经推广力度进行一个合理的评估，当然评估标准不能太大也不能太小，太大的话投入的资源达不到实际效果，纯粹浪费资源，太小的话，某个时间点一个高峰值的访问量上来直接可以压垮系统。
&ensp;&emsp;&emsp;相关概念：

**PV（page view）网站的总访问量**，页面浏览量或点击量，用户每刷新一次就会被记录一次。

**UV（unique Visitor）访问网站的一台电脑客户端为一个访客**。一般来讲，时间上以00:00-24:00之间形同ip的客户端只记录一次。

**QPS（query per second）即每秒查询数**，qps很大程度上代表了系统业务上的繁忙程度，每次请求的背后，可能对应着多次磁盘I/O，多次网络请求，多个cpu时间片等。我们通过qps可以非常直观的了解当前系统业务情况，一旦当前qps超过所设定的预警阈值，可以考虑增加机器对集群扩容，以免压力过大导致宕机，可以根据前期的压力测试得到估值，在结合后期综合运维情况，估算出阈值。

**RT（response time）即请求的响应时间**，这个指标非常关键，直接说明前端用户的体验，因此任何系统设计师都想降低rt时间。

当然还涉及cpu、内存、网络、磁盘等情况，更细节的问题很多，如select、update、delete/ps等数据库层面的统计。

面试常问试题：**如何解决高并发问题？**（解决高并发问题不在于技术，在于业务,实现业务模块化）
1、网络端：

2、服务器:nginx负载均衡 (前端：lvs/haproxy 可以适应更高并发——粗粒度)
    实现分流
3、Java：限流（使用Semaphore信号量） （使用Redix可以实现限流）


**容量评估：**
&ensp;&emsp;&emsp;一般来说通过开发、运维、测试、以及业务等相关人员，综合出系统的一系列阀值，然后我们根据关键阀值如qps、rt等，对系统进行有效的变更。
&ensp;&emsp;&emsp;一般来讲，我们进行多轮压力测试以后，可以对系统进行峰值评估，采用所谓的80/20原则，即80%的访问请求将在20%的时间内达到。这样我们可以根据系统对应的PV计算出峰值qps。
&ensp;&emsp;&emsp;**峰值qps=(总PV x 80%)／(60 x 60 x 24 x 20%)**
&ensp;&emsp;&emsp;然后在总的峰值qps除以单台机器所能承受的最高的qps值，就是所需要机器的数量：
&ensp;&emsp;&emsp;**机器数=总的峰值qps/压测得到的单机极限qps**
&ensp;&emsp;&emsp;当然不排除系统在上线前进行大型促销活动，或者双十一、双十二热点事件、遭受到DDos攻击等情况，系统的开发和运维人员急需要了解当前系统运行的状态和负载情况，一般都会有后台系统去维护。
&ensp;&emsp;&emsp;Semaphore可以控制系统的流量：
&ensp;&emsp;&emsp;拿到信号量的线程可以进入，否则就等待。通过acquire()和release()获取和释放访问许可。

_UseSemaphore.java_

```java
public class UseSemaphore {  
  
    public static void main(String[] args) {  
        // 线程池  
        ExecutorService exec = Executors.newCachedThreadPool();  
        // 只能5个线程同时访问  
        final Semaphore semp = new Semaphore(5);  
        // 模拟20个客户端访问  
        for (int index = 0; index < 20; index++) {  
            final int NO = index;  
            Runnable run = new Runnable() {  
                public void run() {  
                    try {  
                        // 获取许可  
                        semp.acquire();  
                        System.out.println("Accessing: " + NO);  
                        //模拟实际业务逻辑
                        Thread.sleep((long) (Math.random() * 10000));  
                        // 访问完后，释放  
                        semp.release();  
                    } catch (InterruptedException e) {  
                    }  
                }  
            };  
            exec.execute(run);  
        } 
        
        try {
			Thread.sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
        
        //System.out.println(semp.getQueueLength());
        // 退出线程池  
        exec.shutdown();  
    }  
} 
```


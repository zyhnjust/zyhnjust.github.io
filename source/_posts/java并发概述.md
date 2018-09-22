
---
title: java并发概述
date: 2018-09-12 13:43:57
tags: 
- java
- 并发
categories: 
- 面试
- java
- 并发
---

# 自己的理解- 全貌
这两天看了那本书《并发编程实战》以及一些相关的文章， 对并发编程有了一点理解。 

<!-- more --> 
## 问题的由来
- 单线程是没有并发问题的。 但是为了适应性能的要求以及多核cpu的兴起， 多线程应运而生。 
- 如果是多线程中的某个线程内存持有的变量， 换句话来说就是无状态的方法， 也是没有并发问题的。 
- 如果线程之间有了共享的数据， 变量或者别的什么， 那么就有可能有并发的问题。 比如说， 我们需要对类内部变量加1. 线程A拿了这个变量， 而B不知道A拿走了， 也拿走了加一， 问题在于拿到的是A 还没有加1的值。那么当大家把这个值放回去的时候， 发现你的修改已经丢失了。 
- [ ] 实例
- 整理一下带来的问题， 首先 
- 1. 是安全的问题， 也就是刚才实例中的两个线程同时修改， 结果并不是期待中的结果。 - 抽象一下， 主要是多个线程对于共享并且可变的状态的访问。 


- - 这里有一个线程安全的定义： 当多个线程访问某个类的时候， 不管运行时环境采取何种调度方式. 或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或者协同，这个类都可以表现出正确的行为， 那么这个类就是线程安全的。

- 2. 活跃性问题。 就是说某些情况甚至会造成死锁等情况发生， 这样的话系统的服务造成了中断。 
- 3. 性能问题。 是说在处理这些并发问题的时候， 还需要考虑性能的影响， 性能影响的原因是线程的切换会使得调度的消耗增多。 cpu 更多的在调度而不是在处理实际的任务。 

## 如何解决这个问题呢
1. 首先如果方法是无状态的， 自然是线性安全的。 
2. 原子性。 如果我们要加个count， 那么下面的代码就不是线性安全的。 

```
	private long count = 0; 
	
	public void service(ServletRequest req, ServletResponse res)
			throws ServletException, IOException {
		BigInteger i = extractFromRequest(req);
		BigInteger[] factors = factor(i);
		++count;
		encodeIntoResponse(res, factors);
	}
```
用AtomicLong 的话， 这个count 增加的时候是原子操作， 所以这样就是线程安全的。 

```
	private final AtomicLong count = new AtomicLong(0); 
	
	public void service(ServletRequest req, ServletResponse res)
			throws ServletException, IOException {
		BigInteger i = extractFromRequest(req);
		BigInteger[] factors = factor(i);
		count.incrementAndGet();
		encodeIntoResponse(res, factors);
	}
```
3. 如果service中不只是一个操作， 比如get， 然后set. 虽然可能每一步都是线程安全的， 但是两者中间可能是有别的线程插入， 这样的话就不是线程安全的。 

```
    if(vector1.contrains(123))
    {
        vetor1.add(e2)      
    }
```
解决办法是用synchronized. 

```
	public void synchronized service(ServletRequest req, ServletResponse res)
			throws ServletException, IOException {
		BigInteger i = extractFromRequest(req);
		BigInteger[] factors = factor(i);
		count.incrementAndGet();
		encodeIntoResponse(res, factors);
	}
```

重入 - 这个是顺便讲一下的。 就是说线程A 拿到了方法1的锁， 如果再次进去方法1， 是可以的。 否则会有问题。 比如如果子类继承了这个方法， super.parentMethod()  就无法拿到锁了。 所以syncrhonized 是可重入的。 

4. 带来了性能的问题， service 只能由一个线程来用， 这样性能受了太大的影响。 
解决： 可以根据业务逻辑， 对其中的一些语句用synchronized的来修饰。 

5. 关于可见性。刚才其实只是想解决共享状态原子性的问题。 实际上， 并发问题除了原子性， 还有可见性。 
原子性是否我对这个状态的处理不会受到任何其他线程的影响， 而可见性呢则是我修改了的状态的结果， 别的线程也是可以看到的。 一个常用的场景是，主线程把值置为true， 子线程根据这个值得true false 做操作。 如果没有任何的处理， 那么子线程是看不到这个结果的。 

自己写了程序跑， 其实没有见到错误的结论， 不确定是否在java8 有了什么优化。 
TODO


```
	private static class ReaderThread extends Thread{
		public void run(){
			while(!ready)
				System.out.println(ready);
				Thread.yield();
			System.out.println(number);
		}
	}
	
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new ReaderThread().start();
		number=42;
		ready=true;
	}
```

解决这个问题
- 可以加锁。 加了锁之后呢， 在第一个线程做了这个事情时候， 第二个线程开始做。 既解决了原子性， 也解决了可见性。 这里怎么弄呢， 可以把两个变量都隐藏， 只是暴露get set， 然后在这两个方法上面加锁即可。 
- 另外可以volatile 
- - volatile的要点在于。 两个线程被cpu 执行的时候， 每个cpu 有自己的寄存器的， 寄存器肯定是比内存快的。 可见性出现的元就是， 线程A cpu的改动放在自己的寄存器缓存， 而B 读的时候是看不到A的改动的。 
- - Volatile 做的事情是读的时候直接去内存读取。 而关于是否因为这个而直接写到内存， 不太清楚。 但是它保证的是一个线程的改动可以被别的线程立刻感知。 
- - volatile 是一种较弱的锁行为。 其实没有锁， 只是可见性。 

6. ThreadLocal 

这个关键词是什么意思， thread 自己存了一个备份。 而不是share的那个。 例子。 

```
public class ThreadLocalExample {

    public static class MyRunnable implements Runnable {

        private ThreadLocal threadLocal = new ThreadLocal();

        public void run() {
            threadLocal.set((int) (Math.random() * 100D));
//            try {
//            Thread.sleep(2000);
//            } catch (InterruptedException e) {
//
//            }
            System.out.println(threadLocal.get());
        }
    }

    public static void main(String[] args) {
         MyRunnable sharedRunnableInstance = new MyRunnable();
         Thread thread1 = new Thread(sharedRunnableInstance);
         Thread thread2 = new Thread(sharedRunnableInstance);
         thread1.start();
         thread2.start();
    }
}
```

## 工具模块以及使用

### 同步容器类
- Vector 和 Hashtable 这两个呢， 本身是线程安全的， 但是呢， 如果是复合操作， 还是有可能有问题的。 比如这个代码

```
	public static Object getLast(Vector list){
		int lastIndex = list.size()-1;
		return list.get(lastIndex);
	}
	
	public static void deleteLast(Vector list){
		int lastIndex = list.size()-1;
		list.remove(lastIndex);
	}
```
这样的话有时候会抛出exception， ArrayIndexOutOfBoundsException。 因为有的时候另外的线程把它删除了，这里就拿不到了。 
解决办法呢， 可以在两个组合上加一个锁。 如下。 

```
	public static Object getLast(Vector list){
		synchronized(list){
			int lastIndex = list.size()-1;
			return list.get(lastIndex);			
		}
	}
	
	public static void deleteLast(Vector list){
		synchronized(list){
			int lastIndex = list.size()-1;
			list.remove(lastIndex);			
		}
	}
```
第二种情况是隐形的， 比如说字串连接， 比如你打印一个东西， 用到tostring， 这样的话呢还是会抛exception。 

### 并发容器
- ConcurrentHashMap

ConcurrentHashMap使用了线程锁分段技术，每次访问只允许一个线程修改哈希表的映射关系，所以是线程安全的
另外还增加了一些别的操作， 比如若没有则添加， 这样呢就不需要为这两个操作再加一个锁。 

- ConpyOnWriteArrayList

每次修改都会创建并且发布一个新的容器副本， 从而实现可变性。 读呢则是原来的容器， 从而就避免了线程的冲突。 

### 同步工具类

下面几种常用的工具。 
- 什么意思， 使用场景。 
- 使用实例
- 注意事项如果有的话

#### CountDownLatch
- 场景是说一个线程想等其他的线程做完了再做。 这是一种闭锁。 
- 比如另外一个情况，希望主线程做了之后，其他线程再开始做。 
- - 不过可以把这个主线程的事情先做， 再创建好了。 这个其实不一定用countdown。
- 

```
work:
	public void run() {
		// TODO Auto-generated method stub
		try {
			startSignal.await();
			System.out.println(Thread.currentThread().getName() + "_start");
			doWork();
			System.out.println(Thread.currentThread().getName() + "_Done");
			endSignal.countDown();
		}catch (InterruptedException ex) {}
	}
	
	void doWork() {
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
main:
		CountDownLatch startSignal = new CountDownLatch(1);
		CountDownLatch endSignal = new CountDownLatch(N);
		
		for(int i=0; i< N; i++) {
			new Thread(new Worker(startSignal, endSignal)).start();
		}
		
		System.out.println("main start");
		startSignal.countDown();
		System.out.println("main ip");
		try {
			endSignal.await();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("main end");		
```


#### FutureTask
主线程等待所有子线程返回。 并且可以有回复的消息可以处理。 


```
		MyCallable call = new MyCallable(1000);
		FutureTask<String> futureTask= new FutureTask<String>(call);
		
		MyCallable call2 = new MyCallable(2000);
		FutureTask<String> futureTask2= new FutureTask<String>(call2);

		ExecutorService executor = Executors.newFixedThreadPool(2);
		executor.execute(futureTask);
		executor.execute(futureTask2);
		
		while(true) {
			try {
				if(futureTask.isDone() && futureTask2.isDone() ) {
					System.out.println("All are done!");
					executor.shutdown();
					return;
				}
				if(futureTask.isDone()) {
					System.out.println("featuretask 1 output= " + futureTask.get());
					System.out.println("wait for futuretask2");
				}
				if(futureTask2.isDone()) {
					System.out.println("featuretask 2 output= " + futureTask2.get());
					System.out.println("wait for futuretask1");
				}
			}catch(Exception e) {
				
			}
			
		}
```


```
worker：
class MyCallable implements Callable{

	private int waitTime;
	
	public MyCallable(int timeMillis) {
		this.waitTime=timeMillis;
	}
	public Object call() throws Exception {
		// TODO Auto-generated method stub
		System.out.println(Thread.currentThread().getName() + "call");
		return Thread.currentThread().getName();
	}
}
```

#### 信号量 Semaphore

- 基本逻辑就是用了一个东西控制资源同时使用量。 如果没有拿到， 就等。 
- 比如一个线程池， 如果用信号量来控制数目， semaphone 和 池子的大小一致， 这样的话如果没有资源的话， 就会阻塞在那里， 而不是直接拒绝。 
- 另外的， 其实更简单的办法用blockingQueue 也是一样的。 

- 原理就是信号量维护着一个许可证的池子， 如果有人acquire就减一， release 就加一。 如果没有了就等在这里， 知道中断或者有了资源。 
- - 公平锁
- - 非公平锁。  两者的区别是说 可以看到和非公平策略相比，就多了一个对阻塞队列的检查。如果非公平， 直接参与竞争。线程A一点时间都没等待就和线程B同等对待。 


```
      Semaphore sem = new Semaphore(1); 
      
      sem.acquire(); 
      sem.release(); 
```

#### 栅栏 Barrier CyclicBarrier
循环栅栏：CyclicBarrier

想法就是等别的几个线程都执行完成了， 然后大家一起执行下面的工作。 
和闭锁的区别呢： 就是等到这个点以后， 大家可以一起开始做什么事情。 而闭锁就是结束而已； 还有就是CyclicBarrier 是可以重用的， 而CountDownLatch 则是不能重用。 


```
		int N=4;
		CyclicBarrier barrier = new CyclicBarrier(N);
		
		for(int i=0; i<N; i++){
			new Writer(barrier).start();
		}
```


```
	static class Writer extends Thread{
		private CyclicBarrier cyclicBarrier;
		public Writer(CyclicBarrier cyclicBarrier){
			this.cyclicBarrier = cyclicBarrier;
		}
		
		public void run(){
			System.out.println(Thread.currentThread().getName() + " write data");
			
			try {
				Thread.sleep(2000);
				System.out.println(Thread.currentThread().getName() + "write down, wait for else");
				cyclicBarrier.await();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
```

## 关于显示锁

ReentrantLock 这个

--相对于synchronized 有些危险， 因为你如果没有release 会有问题。 

## 线程池
感觉没有什么. 

## java 内存模型

- 重排序： 就是说编译器根据自己的情况做一些优化。 底层实现看到了就收集一些吧。 
- happen before 规则： JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。

JVM定义的Happens-Before原则是一组偏序关系：对于两个操作A和B，这两个操作可以在不同的线程中执行。如果A Happens-Before B，那么可以保证，当A操作执行完后，A操作的执行结果对B操作是可见的。

但是说A 不一定在B之前执行， 怎么理解 ？？？


## 其他的ms 问题


- 什么叫fail-fast 。 就是如果进行操作， 直接抛出ConcurrentModificationException，

- 自旋锁， 可重入：
可重入是说当前线程获得锁之后呢， 还可以继续进入这个锁， 当然需要计数， release 的时候减少。 synchronize 和 reentonlock 的都是可以重入的。 	不可重入呢， 就是不能再进去了。 自旋锁是说我去拿一个锁， 没有拿到， 然后呢， while 不断循环去拿这个锁。 信号量也是， ——》 这两个都是获得锁的方式。 java code 看不到因为是底层处理的。 但是自旋锁呢是在执行的 running 队列里面的， cpu 有了时间片就可以执行这个。   而Spin lock则不然，它属于busy-waiting类型的锁，如果线程A是使用pthread_spin_lock操作去请求锁，那么线程A就会一直在 Core0上进行忙等待并不停的进行锁请求，直到得到这个锁为止。- 多核cpu
而信号量则是放在waiting 队列里的。 这个要等待。 


- 线程的几种状态：New（新建状态），Runnable（就绪状态），Running（运行状态），Blocked（阻塞状态），Dead（死亡状态）。
- wait/notify/notifyAll方法的作用是实现线程间的协作，那为什么这三个方法不是位于Thread类中，而是位于Object类中？位于Object中，也就相当于所有类都包含这三个方法（因为Java中所有的类都继承自Object类）。要回答这个问题，还是得回过来看wait方法的实现原理，大家需要明白的是，wait等待的到底是什么东西？如果对上面内容理解的比较好的话，我相信大家应该很容易知道wait等待其实是对象monitor，由于Java中的每一个对象都有一个内置的monitor对象，自然所有的类都理应有wait/notify方法。refer to http://www.cnblogs.com/paddix/p/5381958.html


## 小结以及下一步

对多线程的基本场景和基本方式进行了一些了解。对整体的理解有了一些深入。 

为什么多线程会有安全问题， synchronized， lock， volatile的一些用法。 常用工具类的用法：futuretask, countdownlock, cylicbarrier, setaphone, 
对线程池着墨不多。 

后期如有时间， 应该继续
- AQS 源码
- lock 还需要继续深入以及实例。 

Question
- 偏向锁： synchronized 好像1.7 加了一些优化。 
- wait sleep

注意
- 自己没有对cpu 指令层进行更多的学习， 觉得暂时必要性不大， 而且耗时较长。 以后有兴趣单独学习。 

参考
1. java 并发编程实战
2. 并发专题 https://www.jianshu.com/p/27860941a77b
3. 
2. 镇楼
![总图](C:/developer/blog/zyhnjust.github.io/source/_posts/java%E5%B9%B6%E5%8F%91%E6%A6%82%E8%BF%B0/2615789-0e32f116bae6062e.png)
![总图](2615789-0e32f116bae6062e.png)
3. code  https://www.geeksforgeeks.org/countdownlatch-in-java/
4. 
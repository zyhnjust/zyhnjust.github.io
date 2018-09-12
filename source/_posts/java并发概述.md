
---
title: java并发概述
date: 2018-09-12 13:43:57
tags: java, 并发
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


#### 栅栏 Barrier CyclicBarrier



注意
- 自己没有对cpu 指令层进行更多的学习， 觉得暂时必要性不大， 而且耗时较长。 以后有兴趣单独学习。 

参考
1. java 并发编程实战
2. 镇楼
![总图](C:/developer/blog/zyhnjust.github.io/source/_posts/java%E5%B9%B6%E5%8F%91%E6%A6%82%E8%BF%B0/2615789-0e32f116bae6062e.png)
![总图](2615789-0e32f116bae6062e.png)
3. code  https://www.geeksforgeeks.org/countdownlatch-in-java/
4. 
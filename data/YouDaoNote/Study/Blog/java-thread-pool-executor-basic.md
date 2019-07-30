---
title: Java 线程池
date: 2017-07-02 16:35:49
tags:
 - Java
 - ThreadPoolExecutor
categories: [Java]
---

# 引言

当我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：
　　如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。
　　那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？
　　在Java中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池。

# ThreadPoolEXecutor
合理利用线程池能够带来三个好处。

 - 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
 - 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
 - 提高线程的可管理性。
 
线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。但是要做到合理的利用线程池，必须对其原理了如指掌。

# 构造方法详解
构造方法有三个：
其一：
```
public ThreadPoolExecutor(
	int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue) 
{
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
}
```

其二：
```
public ThreadPoolExecutor(
	int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue,
	ThreadFactory threadFactory)
{
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
}
```

其三：
```
public ThreadPoolExecutor(
	int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue,
	RejectedExecutionHandler handler) 
{
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
}
```

源码对各个参数介绍如下
```
    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default thread factory.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code handler} is null
     */
```

看不懂？没关系，下面一一解释

 - corePoolSize：线程池的核心线程数
 当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
 - maximumPoolSize：线程池所能容纳的最大线程数
 如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
 - keepAliveTime：非核心线程闲置时的超时时长
 线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
 - unit：用于指定keepAliveTime参数的时间单位
 可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(TimeUnit.MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。常用的有分钟，秒，毫秒
 - workQueue：任务队列
 用于保存等待执行的任务的阻塞队列。有一下几种
	 - ArrayBlockingQueue：
	 是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
	 - LinkedBlockingQueue：
	 一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
	 - SynchronousQueue：
	 一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
	 - PriorityBlockingQueue：一个具有优先级的无限阻塞队列。
 - handler：饱和策略(不常用)
 当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。
	 - AbortPolicy：直接抛出异常。
	 - CallerRunsPolicy：只用调用者所在线程来运行任务。
	 - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
	 - DiscardPolicy：不处理，丢弃掉。
	 - 当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。


# 四种常用的线程池

## FixedThreadPool
是一种线程数量固定的线程池，当线程处于空闲状态时，它们并不会回收，除非线程池关闭了。当所有线程都处于活动状态时，新任务都会处于等待状态，知道有线程空闲出来。由于FixedThreadPool只有核心线程并且这些核心线程都不会被回收，意味着它能够更加快速的响应外界的请求。没有超时机制，任务队列也没有大小限制。  


创建实例
```
ExecutorService fixedThreadPoolExecutor = Executors.newFixedThreadPool(3);
```

newFixedThreadPool()方法定义如下：
```java
    /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

举个例子：
```
    private static void fixedThreadPoolTest(){
        ExecutorService fixedThreadPoolExecutor = Executors.newFixedThreadPool(3);
        for (int i=0; i<10; i++){
            final int index = i;
            System.out.println(System.currentTimeMillis() +
                    "in main thread index = " + index + " " +
                    "Thread id = " + Thread.currentThread().getId());
            fixedThreadPoolExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(System.currentTimeMillis() +
                            "in thread pool index = " + index + " " +
                            "Thread id = " + Thread.currentThread().getId());
                    try {
                        Thread.sleep(2000);
                    }catch (InterruptedException itre){
                        itre.printStackTrace();
                    }

                }
            });
        }
    }
```

运行结果
```java
1498982458294in main thread index = 0 Thread id = 1
1498982458296in main thread index = 1 Thread id = 1
1498982458296in thread pool index = 0 Thread id = 10
1498982458296in main thread index = 2 Thread id = 1
1498982458302in thread pool index = 1 Thread id = 11
1498982458302in main thread index = 3 Thread id = 1
1498982458303in main thread index = 4 Thread id = 1
1498982458303in main thread index = 5 Thread id = 1
1498982458303in main thread index = 6 Thread id = 1
1498982458303in main thread index = 7 Thread id = 1
1498982458303in main thread index = 8 Thread id = 1
1498982458303in main thread index = 9 Thread id = 1
1498982458304in thread pool index = 2 Thread id = 12
// 睡两秒
1498982460296in thread pool index = 3 Thread id = 10
1498982460303in thread pool index = 4 Thread id = 11
1498982460304in thread pool index = 5 Thread id = 12
// 睡两秒
1498982462297in thread pool index = 6 Thread id = 10
1498982462303in thread pool index = 7 Thread id = 11
1498982462304in thread pool index = 8 Thread id = 12
// 睡两秒
1498982464297in thread pool index = 9 Thread id = 10
```
可以看到主线程很快就执行完了，它只负责创建了10个任务。然后交给线程池执行。
线程池数量为3，因此每次只执行其中3个任务，即打印连续3条记录，执行线程为10,11,12，其余任务处于等待状态。然后等待两秒，此时线程池所有线程应该是空闲的，下一次依然在线程10,11,12中执行3个任务，直到所有任务完成。

主线程id为1，子线程id为10,11,12
因为线程池只设置了核心线程数为3，并且最大线程数也是3。



## CacheThreadPool
它是一种线程数量不固定的线程池，它只有非核心线程，并且虽大线程数为Integer.MAX_VALUE(一个很大的数)，相当于最大线程数可以任意大。当线程池中的线程都处于活动状态时，线程池会创建新的线程来处理新任务，否则就会利用空闲的线程来处理新任务。空闲线程有超时机制，60秒，超过60秒，回收空闲线程。
从CacheThreadPool的特性可以知道，这类线程池适合执行大量耗时少的任务。当整个线程池都处于闲置状态时候，线程池中的线程都会因为超时而被回收，几乎不占用任何系统资源。

创建实例
```
ExecutorService cacheThreadPoolExecutor = Executors.newCachedThreadPool();
```

newCachedThreadPool()方法如下：
```java
    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

举个例子
```
    private static void cacheThreadPoolTest(){
        ExecutorService cacheThreadPoolExecutor = Executors.newCachedThreadPool();

        for (int i=0; i<10; i++){
            final int index = i;
            /*
            try {
                Thread.sleep(1000);
            }catch (InterruptedException itre){
                itre.printStackTrace();
            }
            */
            
            cacheThreadPoolExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(System.currentTimeMillis() + " " +
                            "index = " +  index + " " +
                            "Thread id = " + Thread.currentThread().getId());
                }
            });
        }
    }
```

运行情况：
```
1498985856125 index = 0 Thread id = 10
1498985856126 index = 1 Thread id = 11
1498985856126 index = 2 Thread id = 12
1498985856127 index = 3 Thread id = 13
1498985856127 index = 7 Thread id = 10
1498985856127 index = 6 Thread id = 14
1498985856127 index = 9 Thread id = 14
1498985856127 index = 4 Thread id = 12
1498985856127 index = 5 Thread id = 11
1498985856131 index = 8 Thread id = 15
```
可以看出，当线程池的所有线程活动的时候，线程池不断创建线程了执行新的任务

如果去掉线程等待的注释，再运行一次
```
    private static void cacheThreadPoolTest(){
        ExecutorService cacheThreadPoolExecutor = Executors.newCachedThreadPool();

        for (int i=0; i<10; i++){
            final int index = i;
            
            try {
                Thread.sleep(1000);
            }catch (InterruptedException itre){
                itre.printStackTrace();
            }

            cacheThreadPoolExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(System.currentTimeMillis() + " " +
                            "index = " +  index + " " +
                            "Thread id = " + Thread.currentThread().getId());
                }
            });
        }
    }
```

运行结果：
```
1498985981855 index = 0 Thread id = 10
// 间隔一秒
1498985982854 index = 1 Thread id = 10
// 间隔一秒
1498985983854 index = 2 Thread id = 10
// 间隔一秒 下同省略了
1498985984854 index = 3 Thread id = 10
1498985985855 index = 4 Thread id = 10
1498985986855 index = 5 Thread id = 10
1498985987855 index = 6 Thread id = 10
1498985988855 index = 7 Thread id = 10
1498985989855 index = 8 Thread id = 10
1498985990855 index = 9 Thread id = 10
```
可以看到，当等待的时候，活动线程已经执行完毕，变成闲置线程，由于等待时间少于60秒，闲置线程未被回收，因此始终在10号线程中执行。


## ScheduledThreadPool
它的核心线程数量是固定的，而非核心线程数没有限制，并且非核心线程闲置时会立即被回收。这类线程池主要用于执行定时任务或具有固定周期的重复任务

创建实例
```
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(4);
```

newScheduledThreadPool()方法如下：
```java
    /**
     * Creates a thread pool that can schedule commands to run after a
     * given delay, or to execute periodically.
     * @param corePoolSize the number of threads to keep in the pool,
     * even if they are idle
     * @return a newly created scheduled thread pool
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

ScheduledThreadPoolExecutor()构造如下：
```
    /**
     * Creates a new {@code ScheduledThreadPoolExecutor} with the
     * given core pool size.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```

举个例子：
```
    private static void scheduledThreadPoolTest(){
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread id = " + Thread.currentThread().getId());
            }
        };

        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(4);
        //延时2s
        scheduledThreadPool.schedule(runnable,2, TimeUnit.SECONDS);
        //延迟1000ms后, 每隔500ms执行一次runnable
        scheduledThreadPool.scheduleAtFixedRate(runnable,1000, 500, TimeUnit.MILLISECONDS);
    }
```

## SingleThreadExecutor
这类线程池内部只有一个核心线程，它确保所有的任务都在同一线程中按顺序执行。SingleThreadExecutor的意义在于同一所有的外界任务到一个线程中，这使得在这些任务之间不需要处理线程同步问题。

创建实例
```
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
```

newSingleThreadExecutor()方法如下：
```java
    /**
     * Creates an Executor that uses a single worker thread operating
     * off an unbounded queue. (Note however that if this single
     * thread terminates due to a failure during execution prior to
     * shutdown, a new one will take its place if needed to execute
     * subsequent tasks.)  Tasks are guaranteed to execute
     * sequentially, and no more than one task will be active at any
     * given time. Unlike the otherwise equivalent
     * {@code newFixedThreadPool(1)} the returned executor is
     * guaranteed not to be reconfigurable to use additional threads.
     *
     * @return the newly created single-threaded Executor
     */
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

举个例子
```
    private static void singleThreadExecutorTest(){
        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        for (int i=0; i<10; i++){
            final int index = i;
            singleThreadExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println("index = " + index+ " Thread id = " + Thread.currentThread().getId());
                }
            });
        }
    }
```

运行结果：
```
index = 0 Thread id = 10
index = 1 Thread id = 10
index = 2 Thread id = 10
index = 3 Thread id = 10
index = 4 Thread id = 10
index = 5 Thread id = 10
index = 6 Thread id = 10
index = 7 Thread id = 10
index = 8 Thread id = 10
index = 9 Thread id = 10
```
可以看出，所有任务都在同一个线程中排队等待执行。



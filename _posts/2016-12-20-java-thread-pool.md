---
layout: post
keywords: Java 线程池
title: Java 线程池和队列
date: 2016-12-20
categories: Java
tags: 线程池
---

如果你还在继承Thread实现多线程？看看这个吧。

### 引子
传统上，我们都知道可以通过继承Thread类或者实现Runable接口来实现多线程的使用。 

从Java5开始，Java提供了自己的线程池。线程池就是一个线程的容器，每次只执行额定数量的线程。 java.util.concurrent.ThreadPoolExecutor就是这样的线程池。
<!-- more -->
### 线程池
1. 线程池的作用
根据系统的环境情况，可以自动或手动设置线程数量，达到运行的最佳效果；少了浪费了系统资源，多了造成系统拥挤效率不高。用线程池控制线程数量，其他线程排队等候。一个任务执行完毕，再从队列的中取最前面的任务开始执行。若队列中没有等待进程，线程池的这一资源处于等待。当一个新任务需要运行时，如果线程池中有等待的工作线程，就可以开始运行了；否则进入等待队列。

2. 为什么要用线程池:
 1) 减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
 2) 可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。
比较重要的几个类：

| ExecutorService      		| 真正的线程池接口 | 
| -------- 					| :-----:        |
| ScheduledExecutorService | 能和Timer/TimerTask类似，解决那些需要任务重复执行的问题|
| ThreadPoolExecutor       | ExecutorService的默认实现 | 
| ScheduledThreadPoolExecutor | 继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现|


### 常见的ThreadPoolExecutor使用流程
```java
// 实现runnale接口，在run方法中实现业务
class test implements Runnable {
    @Override
    public void run() {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("test");
    }
}
// 为某个业务单独创建一个线程池
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
        1,
        5,
        3,
        TimeUnit.MINUTES,
        new LinkedBlockingQueue<Runnable>()
);
// 在具体的业务逻辑处，调用execute方法，多线程执行业务
for (int i = 0; i < 10; i++) {
    threadPoolExecutor.execute(new test());
}
```

### ThreadPoolExecutor构造方法说明
```java
public ThreadPoolExecutor(
        int corePoolSize,                  //线程池维护线程的最少数量 
        int maximumPoolSize,               //线程池维护线程的最大数量 
        long keepAliveTime,	           //线程所允许的空闲时间(对超出corePoolSize的线程而言)
        TimeUnit unit,                     //线程所允许的空闲时间的单位 
        BlockingQueue<Runnable> workQueue, //线程池所使用的缓冲队列
        RejectedExecutionHandler handler   //线程池对拒绝任务的处理策略  
) {}
```
### 线程处理的优先级

1. 在线程池中线程数小于corePoolSize时，每次执行都新建一个线程
2. 在线程池中线程数等于corePoolSize，缓冲队列未满时，优先丢入队列
3. 在线程池中线程数等于或大于corePoolSize，缓冲队列已满时，新建线程执行任务，直到线程数达到maximumPoolSize
4. 在线程池中线程数等于maximumPoolSize，缓冲队列已满时，使用handler处理拒绝任务
5. 在线程池中线程数大于corePoolSize时，空闲线程的存活时间由keepAliveTime和unit决定

### BlockingQueue的种类

1. ArrayBlockingQueue
数组实现的阻塞队列，数组的大小就是队列的长度，如果队列为空且线程进行元素获取，或者队列已满且进行任务添加，都将导致阻塞等待。进出队列采用FIFO（先进先出）原则。

2. LinkedBlockingQueue
链表实现的阻塞队列，如果不在构造时指定大小，则其大小取决于Integer.MAX_VALUE，除了实现方式与ArrayBlockingQueue不同外，行为基本相同。

3. PriorityBlockingQueue
类似于LinkedBlockQueue，但其所含对象的排序不是FIFO，而是依据对象的自然排序顺序或者是构造函数的Comparator决定的顺序。

4. SynchronousQueue
该队列的操作必须是放和取交替完成的。在被元素被取走之前，该元素的插入操作不会结束，因此，名为同步队列，也即非异步、阻塞执行。该队列长度为0，元素插入就需要被取走。

5. DelayQueue
是一个无界的BlockingQueue，用于放置实现了Delayed接口的对象，其中的对象只能在其到期
时才能从队列中取走。这种队列是有序的，即队头对象的延迟到期时间最长。

6. LinkedTransferQueue
无界队列（Integer.MAX_VALUE），进出队列采用FIFO（先进先出）原则。生产者会一直阻塞直到所添加到队列的元素被某一个消费者所消费。主要用于线程间消息的传递，与SynchronousQueue很类似，但是比起SynchronousQueue更好用。LinkedTransferQueue既可以使用BlockingQueue的put方法进行常规的添加元素操作，也可以使用transfer方法进行阻塞添加，而且比SynchronousQueue灵活之处在于，队列长度非0，阻塞插入和非阻塞插入的元素可以共存。

7. 各种Deque（双端队列）
双端队列不仅可以实现FIFO（先进先出）队列，还可以实现FILO（先进后出）的栈，但是不常用，在此不多做介绍。

### RejectedExecutionHandler的种类

1. ThreadPoolExecutor.AbortPolicy()
抛出java.util.concurrent.RejectedExecutionException异常，注意，这是线程池的默认策略

2. ThreadPoolExecutor.CallerRunsPolicy()
重试添加当前的任务，他会自动重复调用execute()方法

3. ThreadPoolExecutor.DiscardOldestPolicy()
抛弃旧的任务（最先进入队列的任务）

4. ThreadPoolExecutor.DiscardPolicy()
抛弃当前的任务（即将进入队列的任务）

### JDK提供的默认实现
ThreadPoolExecutor的构造方法不可谓不复杂，因此JDK也不推荐直接使用。java.util.concurrent.Executors提供了默认的实现，以此应对不同场景。ExcuteService实现了Executors并且是ThreadPoolExecutor的父接口。
通过以上介绍，查看源码即可很清晰的明白这四种实现的区别。

### FixedThreadPool

线程数量固定的线程池，空闲线程销毁时间为0，无界队列，拒绝任务的策略为抛出异常
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(
    	nThreads,
    	nThreads,
        0L,
        TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>()
    );
}
```

### SingleThreadExecutor

最大线程数为1的线程池，空闲线程销毁时间为0，无界队列，拒绝任务的策略为抛出异常
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(
            1,
            1,
            0L,
            TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>()
        ));
}
```

### CachedThreadPool

无界线程池，可以进行自动线程回收。之所以说可以自动回收是因为corePoolSize被设置为零，此外，这个线程池比较特殊的特点是采用了SynchronousQueue。结合上文，就会明白，所有元素都会单起一个线程阻塞的等待获取。
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(
    	0,
    	Integer.MAX_VALUE,
        60L,
        TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>()
    );
}
```

### WorkStealingPool

在Executors中还会看到这种池，这个不是平常使用的线程池。而是一个ForkJoinPool。
ForkJoinPool是一个可以执行ForkJoinTask的ExcuteService，与ExcuteService不同的是它采用了work-stealing模式：所有在池中的线程尝试去执行其他线程创建的子任务（多线程并行执行一个任务），这样就很少有线程处于空闲状态，非常高效。
ForkJoinPool解决的不是并发问题，而是高效并行问题，在这里不做具体介绍。

### SingleThreadScheduledExecutor

ScheduledThreadPoolExecutor是ThreadPoolExecutor的子类，用于多线程的执行定时任务，其内部使用的是与上文提到的DelayQueue相类似的队列，DelayedWorkQueue。此类线程池也不多做介绍。

### 总结
看到这里，想必对ThreadPoolExecutor的使用已经有了概念，接下来，只要根据需要使用响应的Executors即可。或者，现在就去写个demo吧 ：）。



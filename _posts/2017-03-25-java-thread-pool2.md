---
layout: post
keywords: Java 并发
title: 再谈Java并发
date: 2017-03-25
categories: Java
tags: Concurrency
---

关于java线程池中的 allowCoreThreadTimeOut 和 Guava Futures的使用。

之前写过一篇关于java线程池的文章：[Java线程池和队列](/2016/12/20/java-thread-pool)

### 要做异步任务执行队列，有个如下需求：

1) 有个线程池用于执行任务

2) 有个无界队列，用于缓存未执行的任务

3) 没有任务执行时，我希望线程池中的线程停掉

这看似是个很正常的需求，但是用JDK1.5(我的工作地方JDK还是1.5的)实现，真得很困难的。

ThreadPoolExecutor中线程池有 corePoolSize 和 maximumPoolSize 两个参数。JDK1.5 中线程池至少保持 corePoolSize 的线程，所以为了满足上面的需求，corePoolSize必须被设置为0。这时，如果JDK1.5中队列不满的话，是不会创建大于corePoolSize大小的线程数的。也就是，corePoolSize为0时，队列满了，才会创建新的线程，这显然不满足我的需求。

在JDK1.6，发现ThreadPoolExecutor多了一个allowCoreThreadTimeOut方法。这个方法是允许线程数低于corePoolSize时，线程也因为空闲而终止。有了这个方法，实现上面的需求就非常简单了。将 corePoolSize 和 maximumPoolSize 设置为相同的大小，allowCoreThreadTimeOut设置为true，加上一个无界队列，就OK了。

<!-- more -->
### allowCoreThreadTimeOut源码

```java
	/**
     * Performs blocking or timed wait for a task, depending on
     * current configuration settings, or returns null if this worker
     * must exit because of any of:
     * 1. There are more than maximumPoolSize workers (due to
     *    a call to setMaximumPoolSize).
     * 2. The pool is stopped.
     * 3. The pool is shutdown and the queue is empty.
     * 4. This worker timed out waiting for a task, and timed-out
     *    workers are subject to termination (that is,
     *    {@code allowCoreThreadTimeOut || workerCount > corePoolSize})
     *    both before and after the timed wait, and if the queue is
     *    non-empty, this worker is not the last thread in the pool.
     *
     * @return task, or null if the worker must exit, in which case
     *         workerCount is decremented
     */
    private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
            //allowCoreThreadTimeOut为ture，则timed始终为true
            //此时只要timeout时间一过，则会销毁线程，则线程数可能会销毁至0
            //效果跟cachedThreadPool一致
			  
            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }

```

#### 参数说明
allowCoreThreadTimeOut为true时，则线程池数量最后可以销毁到0个。
allowCoreThreadTimeOut为false时：超过核心线程数时，而且（超过最大值或者超过timeout），就会销毁。

没有timeout值的时候始终保持在maxPoolSize水平；

如果有timeout情况，那么保持在corePoolSize水平。默认是cachedThreadPool才会设置timeout为60秒，其他Executors造出来的timeout为0，即没有timeout。而cachedThreadPool的corePoolSize为0，即cachedThreadPool，最后线程数量为0.  
（1）当没有超过核心线程时，不会销毁线程   
（2）当超过核心线程数：再判断，如果超过最大值，则销毁；如果超过timeout，则销毁

### 线程池执行服务对象

```java
import com.google.common.base.Joiner;
import com.google.common.util.concurrent.MoreExecutors;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;

import java.util.concurrent.*;

public final class ExecutorServiceObject {

    private final ThreadPoolExecutor threadPoolExecutor;

    private final BlockingQueue<Runnable> workQueue;

    public ExecutorServiceObject(final String namingPattern, final int threadSize) {
        workQueue = new LinkedBlockingQueue<>();
        threadPoolExecutor = new ThreadPoolExecutor(threadSize, threadSize, 60L, TimeUnit.SECONDS, workQueue,
                new BasicThreadFactory.Builder().namingPattern(Joiner.on("-").join(namingPattern, "%s")).build());
        threadPoolExecutor.allowCoreThreadTimeOut(true);
    }

    /**
     * 创建线程池服务对象.
     *
     * @return 线程池服务对象
     */
    public ExecutorService createExecutorService() {
        return MoreExecutors.listeningDecorator(MoreExecutors.getExitingExecutorService(threadPoolExecutor));
    }

    public boolean isShutdown() {
        return threadPoolExecutor.isShutdown();
    }

    /**
     * 获取当前活跃的线程数.
     *
     * @return 当前活跃的线程数
     */
    public int getActiveThreadCount() {
        return threadPoolExecutor.getActiveCount();
    }

    /**
     * 获取待执行任务数量.
     *
     * @return 待执行任务数量
     */
    public int getWorkQueueSize() {
        return workQueue.size();
    }
}
```

这里我们使用到了MoreExecutors类，其中包含了大量的静态方法用于处理Executor, ExecutorService和ThreadPool实例，翻开源码，整理其中的公共方法，如下：

getExitingExecutorService(ThreadPoolExecutor executor, long terminationTimeout, TimeUnit timeUnit)：将给定的ThreadPoolExecutor转换成ExecutorService实例，在程序完成时退出， 它是通过使用守护线程和添加一个关闭钩子来等待他们完成。

getExitingScheduledExecutorService(ScheduledThreadPoolExecutor executor, long terminationTimeout, TimeUnit timeUnit)：将给定的ScheduledThreadPoolExecutor转换成ScheduledExecutorService实例，在程序完成时退出， 它是通过使用守护线程和添加一个关闭钩子来等待他们完成。

addDelayedShutdownHook(ExecutorService service, long terminationTimeout, TimeUnit timeUnit)：添加一个关闭的钩子来等待给定的ExecutorService中的线程完成。

getExitingExecutorService(ThreadPoolExecutor executor)：将给定的ThreadPoolExecutor转换成ExecutorService实例，在程序完成时退出， 它是通过使用守护线程和添加一个关闭钩子来等待他们完成。

getExitingScheduledExecutorService(ScheduledThreadPoolExecutor executor)：将给定的ThreadPoolExecutor转换成ScheduledExecutorService实例，在程序完成时退出， 它是通过使用守护线程和添加一个关闭钩子来等待他们完成。

sameThreadExecutor()：创建一个ExecutorService实例，运行线程中的每一个任务。

listeningDecorator(ExecutorService delegate)：创建一个ExecutorService实例，通过线程提交或者唤醒其他线程提交ListenableFutureTask到给定的ExecutorService实例。

listeningDecorator(ScheduledExecutorService delegate)：创建一个ScheduledExecutorService实例，通过线程提交或者唤醒其他线程提交ListenableFutureTask到给定的ExecutorService实例。

platformThreadFactory()：返回一个默认的线程工厂用于创建新的线程。

shutdownAndAwaitTermination(ExecutorService service, long timeout, TimeUnit unit)：逐渐关闭指定的ExecutorService，首先会禁用新的提交， 然后会取消现有的任务。 

### 测试用例

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;

public class ThreadTest {

//    private static AtomicInteger threadcounter = new AtomicInteger(1);
//    private static ExecutorService executor = Executors.newFixedThreadPool(100, r -> {
//        Thread thread = new Thread(r);
//        thread.setName("Thread " + threadcounter.getAndIncrement());
//        System.out.println(thread.getName());
//        return thread;
//    });

    private static ExecutorService executor = new ExecutorServiceObject("thread-test", 100).createExecutorService();

    private static AtomicBoolean isConnected = new AtomicBoolean(false);

    public static void main(String[] args) throws Exception {

        for (int i = 0; i < 1000; i++) {
            process(i);
        }

        // 模拟服务器断连
        sleepMs(10 * 1000);

        isConnected.set(true);

        // 模拟线程池销毁线程后
        sleepMs(80 * 1000);
        System.out.println("============================分割线==========================");
        for (int i = 0; i < 1000; i++) {
            process(i);
        }
    }

    public synchronized static void reconnect() {
        if (!isConnected.get()) {
            sleepMs(3000);
        }
    }

    public static void process(int index) {
        executor.execute(() -> {
            System.out.println("thread_name: " + Thread.currentThread().getName() + ", index: " + index);

            if (isConnected.get()) {
                sleepMs(10);
            } else {
                reconnect();

                process(index);
            }
        });
    }

    public static void sleepMs(long milliSeconds) {
        try {
            TimeUnit.MILLISECONDS.sleep(milliSeconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
执行的结果太多这里就不贴了。

### Guava Futures

Java 5中引入了concurrent包，其中提供了许多重要的并发设计，其中一个便是Future对象，Future用于表示一个异步计算任务，我们通常需要启动一个Executor实例，之后调用submit方法获取到Future对象，并通过future.get()方法获取线程执行完成后的结果，ListenableFuture接口继承了Future接口进行了扩展，允许我们注册一个Callback函数，并在任务完成后执行。

#### 一个简单的Future栗子

```java

ExecutorService executor = Executors.newCachedThreadPool();

Future<Integer> future = executor.submit(new Callable<Integer>() {

    @Override

    public Integer call() throws Exception {

        //这里调用一些处理逻辑

        return 1 + 1;
    }
});
```
#### 获得ListenableFuture接口

```java
ListeningExecutorService executorService = MoreExecutors.listeningDecorator(executor);
```

#### 一个ListenableFuture栗子
```java
public class Test1 {

    public static void main(String[] args) {
        int NUM_THREADS = 10;//10个线程

        ListeningExecutorService executorService = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(NUM_THREADS));

        ListenableFuture<String> listenableFuture =

                executorService.submit(new Callable<String>() {
                    @Override
                    public String call() throws Exception {
                        return null;
                    }

                });

        listenableFuture.addListener(new Runnable() {

            @Override
            public void run() {
                //在Future任务完成之后运行的一些方法
                System.out.println("methodToRunOnFutureTaskCompletion");
            }

        }, executorService);
    }
}
```

不过，ListenableFuture.addListener有一个小的缺陷，我们没有办法接收返回的对象，这就导致在任务执行失败或成功的时候，我们不能执行其他的操作，不过Guava 提供了FutureCallback接口来弥补这个缺陷。

### Guava Futures

```java
public class FuturesTest {

    @Test
    public void v1() throws Exception {
        ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));

        ListenableFuture future1 = service.submit(new Callable<Integer>() {
            public Integer call() throws InterruptedException {
                Thread.sleep(1000);
                System.out.println("call future 1 ...");
                return 1;
            }
        });

        ListenableFuture future2 = service.submit(new Callable<Integer>() {
            public Integer call() throws InterruptedException {
                Thread.sleep(1000);

//                throw new RuntimeException("call future 2 ...");

                System.out.println("call future 2 ...");
                return 2;
            }
        });

        final ListenableFuture allFutures = Futures.allAsList(future1, future2);
//        final ListenableFuture allFutures = Futures.successfulAsList(future1, future2);

        final ListenableFuture transform = Futures.transform(allFutures, new AsyncFunction<List<Integer>, Boolean>() {
            @Override
            public ListenableFuture apply(List<Integer> results) throws Exception {
                System.out.println("results: " + results);
                return Futures.immediateFuture(String.format("success future: %d", results.size()));
            }
        });

        Futures.addCallback(transform, new FutureCallback<Object>() {

            public void onSuccess(Object result) {
                System.out.println();
                System.out.printf("onSuccess: %s%n", result);
            }

            public void onFailure(Throwable thrown) {
                System.out.printf("onFailure: %s%n", thrown.getMessage());
            }
        });

        System.out.println(transform.get());
    }
}
```

在Guava中Futures对于Future扩展还有：

transform：对于ListenableFuture的返回值进行转换。  
allAsList：对多个ListenableFuture的合并，返回一个当所有Future成功时返回多个Future返回值组成的List对象。注：当其中一个Future失败或者取消的时候，将会进入失败或者取消。  
successfulAsList：和allAsList相似，唯一差别是对于失败或取消的Future返回值用null代替。不会进入失败或者取消流程。  
immediateFuture/immediateCancelledFuture：立即返回一个待返回值的ListenableFuture。  
makeChecked: 将ListenableFuture 转换成CheckedFuture。CheckedFuture 是一个ListenableFuture ，其中包含了多个版本的get 方法，方法声明抛出检查异常.这样使得创建一个在执行逻辑中可以抛出异常的Future更加容易  
JdkFutureAdapters.listenInPoolThread(future): guava同时提供了将JDK Future转换为ListenableFuture的接口函数。



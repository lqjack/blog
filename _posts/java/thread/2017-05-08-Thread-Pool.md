---
layout: post
title: 线程池
date: 2016-02-24 22:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- Thread Pool
star: true
category: blog
author: Jack
description: 线程池
---

1 Exectors.newCachedThreadPool(无界线程池，可以进行自动回收)
<small>无界队列，阻塞队列使用SynchronousQueue（每个插入操作对应一个移除操作)</small>

``` java
	public static ExecutorService newCachedThreadPool() {  
		return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, 
		TimeUnit.SECONDS,new SynchronousQueue<Runnable>());  
	} 
```

2 Exectors.newFixedThreadPool(固定线程池)
<small>corePoolSize和maximumPoolSize的大小是一样的，不想实现Keep alive，队列无界。</small>

``` java
	public static ExecutorService newFixedThreadPool(int nThreads) {  
		return new ThreadPoolExecutor(nThreads, nThreads,0L, 
		TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());  
	}  
```


3 Exectors.newSingleThreadPool(单个线程池)

``` java
	public static ExecutorService newSingleThreadExecutor() {  
		return new FinalizableDelegatedExecutorService(
		new ThreadPoolExecutor(1,1,0L, 
		TimeUnit.MILLISECONDS,
		new LinkedBlockingQueue<Runnable>()));  
	}
```

**BlockingQueue<Runnable>**
* 如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。（如果当前运行的线程小于corePoolSize，则任务根本不会存放，添加到queue中。而是开始运行）
* 如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。
* 如果无法将请求加入队列，则创建新的线程，除非创建此线程超出maximumPoolSize，在这种情况下，任务将被拒绝。

排队有三种通用策略：
+ 直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交时，通常要求无界，maximumPoolSizes以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
+ 无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
+ 有界队列。当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。  

<p>
例子一：使用直接提交策略，也即SynchronousQueue。
 
首先SynchronousQueue是无界的，也就是说他存数任务的能力是没有限制的，但是由于该Queue本身的特性，在某次添加元素后必须等待其他线程取走后才能继续添加。在这里不是核心线程便是新创建的线程，但是我们试想一样下，下面的场景。
</p>
我们使用一下参数构造ThreadPoolExecutor：
 
```Java
new ThreadPoolExecutor(  
				2, 3, 30, TimeUnit.SECONDS,   
				new SynchronousQueue<Runnable>(),   
				new RecorderThreadFactory("CookieRecorderPool"),   
				new ThreadPoolExecutor.CallerRunsPolicy());
````

当核心线程已经有2个正在运行.此时继续来了一个任务（A），根据前面介绍的“如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。”,所以A被添加到queue中。
又来了一个任务（B），且核心2个线程还没有忙完，OK，接下来首先尝试1中描述，但是由于使用的SynchronousQueue，所以一定无法加入进去。
此时便满足了上面提到的“如果无法将请求加入队列，则创建新的线程，除非创建此线程超出maximumPoolSize，在这种情况下，任务将被拒绝。”，所以必然会新建一个线程来运行这个任务。
暂时还可以，但是如果这三个任务都还没完成，连续来了两个任务，第一个添加入queue中，后一个呢？queue中无法插入，而线程数达到了maximumPoolSize，所以只好执行异常策略了。
所以在使用SynchronousQueue通常要求maximumPoolSize是无界的，这样就可以避免上述情况发生（如果希望限制就直接使用有界队列）。对于使用SynchronousQueue的作用jdk中写的很清楚：此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。
(如果你的任务A1，A2有内部关联，A1需要先运行，那么先提交A1，再提交A2，当使用SynchronousQueue我们可以保证，A1必定先被执行，在A1么有被执行前，A2不可能添加入queue中)

例子二：使用无界队列策略，即LinkedBlockingQueue

这个就拿newFixedThreadPool来说，根据前文提到的规则：
 写道
如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。
 那么当任务继续增加，会发生什么呢？
 写道
 
如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。
 OK，此时任务变加入队列之中了，那什么时候才会添加新线程呢？
 
 写道
如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。
这里就很有意思了，可能会出现无法加入队列吗？不像SynchronousQueue那样有其自身的特点，对于无界队列来说，总是可以加入的（资源耗尽，当然另当别论）。换句说，永远也不会触发产生新的线程！corePoolSize大小的线程数会一直运行，忙完当前的，就从队列中拿任务开始运行。所以要防止任务疯长，比如任务运行的实行比较长，而添加任务的速度远远超过处理任务的时间，而且还不断增加，如果任务内存大一些，不一会儿就爆了，呵呵。
 
可以仔细想想哈。
 
例子三：有界队列，使用ArrayBlockingQueue。
 
这个是最为复杂的使用，所以JDK不推荐使用也有些道理。与上面的相比，最大的特点便是可以防止资源耗尽的情况发生。
 
举例来说，请看如下构造方法：
```Java
new ThreadPoolExecutor(  
            2, 4, 30, TimeUnit.SECONDS,   
            new ArrayBlockingQueue<Runnable>(2),   
            new RecorderThreadFactory("CookieRecorderPool"),   
            new ThreadPoolExecutor.CallerRunsPolicy());  
```
假设，所有的任务都永远无法执行完。
 
对于首先来的A,B来说直接运行，接下来，如果来了C,D，他们会被放到queu中，如果接下来再来E,F，则增加线程运行E，F。但是如果再来任务，队列无法再接受了，线程数也到达最大的限制了，所以就会使用拒绝策略来处理。
 
小结：
* ThreadPoolExecutor的使用还是很有技巧的。
* 使用无界queue可能会耗尽系统资源。
* 使用有界queue可能不能很好的满足性能，需要调节线程数和queue大小
* 线程数自然也有开销，所以需要根据不同应用进行调节。
* 通常来说对于静态任务可以归为：
* 数量大，但是执行时间很短
* 数量小，但是执行时间较长
* 数量又大执行时间又长
* 除了以上特点外，任务间还有些内在关系
* 看完这篇问文章后，希望能够可以选择合适的类型了


**估算线程池大小：**

1. 最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目
或者 最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）* CPU数目

2. 依据：线程等待时间所占比例越高，需要越多线程。线程CPU时间所占比例越高，需要越少线程。
一个系统最快的部分是CPU，所以决定一个系统吞吐量上限的是CPU。增强CPU处理能力，可以提高系统吞吐量上限。但根据短板效应，真实的系统吞吐量并不能单纯根据CPU来计算。那要提高系统吞吐量，就需要从“系统短板”（比如网络延迟、IO）着手：
尽量提高短板操作的并行化比率，比如多线程下载技术
增强短板能力，比如用NIO替代IO






---
typora-copy-images-to: ..\..\..\img
---

# Java中的线程池

## 1 .为什么要使用线程池？

&emsp;&emsp;Java中的线程池是运用场景最多的并发框架，几乎所有需要异步或并发执行任务的程序都可以使用线程池。在开发过程中，合理地使用线程池能够带来3个好处。

1. **降低资源消耗**。通过利用已创建的线程降低线程创建和销毁造成的开支
2. **提高响应速度**。任务到达时，任务不需要等待创建线程就能被直接执行。
3. **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，而且会影响系统的稳定性。使用线程池可以进行统一分配、调度和监控。

&emsp;&emsp;要合理运用线程池，必须掌握其实现原理。

## 2 .任务处理流程

&emsp;&emsp;当一个任务提交到线程池后，其流程如下：

1. 判断线程池中的的核心线程是否都在执行任务。如果不是，则创建一个新的核心线程执行任务(也叫线程池预热阶段)，否则，进行下一个步骤。

2. 判断工作队列是否已满了。如果工作队列没有满，则线程放置到工作队列中。如果满了，则进入下一步骤。
3. 判断线程池中的线程总数量是否到达了阈值，如果没到，则创建一个新的线程来执行任务。否则，交给饱和策略来处理这个任务。

流程图如下所示：

![1559546177713](..\..\..\img\1559546177713.png)

综上，我们可以得知，ThreadPoolExecutor在执行任务时会分为以下4种情况。

1. 线程数量<核心线程数量,则创建新的线程来处理任务(注意：执行这一步需要获取全局锁)
2. 线程数量>=核心线程数量，则将任务添加到工作队列(阻塞队列)。
3. 工作队列已满，则创建新的线程来执行任务(需要获取全局锁)
4. 线程池已满，则任务被拒绝，调用$RejectedExecutionHandler.rejectedExecution()$方法。

&emsp;&emsp;ThreadPoolExecutor采取上述步骤的总体设计思路，是为了在执行execute()方法时，尽可能地避免获取全局锁（那将会是一个严重的可伸缩瓶颈）。在ThreadPoolExecutor完成预热之后（当前运行的线程数大于等于corePoolSize），几乎所有的execute()方法调用都是执行步骤2，而步骤2不需要获取全局锁。

**执行示意图：**

![1559549847962](..\..\..\img\1559549847962.png)

**源码分析（JDK1.8）**

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
    	//1 线程数量<核心线程数，则创建新线程
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
   		 //2 判断线程池在工作状态，将任务加入队列
        if (isRunning(c) && workQueue.offer(command)) {
            //再次检测线程池状态 
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
    //拒绝任务
        else if (!addWorker(command, false))
            reject(command);
    }
```

分析:线程池状态有如下几种

```java

    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

&emsp;&emsp;在第二步中，通过源码可以看到检测线程池在运行中状态且成功添加到线程池中后，又检测了一遍状态，原因是在第一次状态检测后可能存在线程销毁，或者在将任务添加到工作队列中后线程池被shutdown掉。如果线程池不在运行状态，则移除队列并拒绝任务。否则，如果线程池没线程，采用的措施是不将任务移出队列，而是创建一个新的工作线程。(工作线程会循环去队列取任务)

## 3线程池的创建

### 3.1通过构造函数创建

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

**参数**：

- **corePoolSize(核心线程数的数量)**：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。

- maximumPoolSize：线程池线程数量上限。

- keepAliveTime：空闲线程存活时间

- TimeUnit unit：时间单位

- BlockingQueue<Runnable> workQueue：工作队列，用来保存等待执行任务的阻塞队列

- threadFactory：创建线程的工厂，，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架guava提供的ThreadFactoryBuilder可以快速给线程池里的线程设置有意义的名字，代码如下：

  ```java
  new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
  ```

- RejectedExecutionHandler handler:饱和策略,当线程池和工作队列都满了无法执行新任务时，需要一种策略处理提交的任务。默认使用AbortPolicy策略，直接抛出异常。支持的几种策略：

  - AbortPolicy：直接抛出异常。
  - CallerRunsPolicy：只用调用者所在线程来运行任务。
  - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
  - DiscardPolicy：不处理，丢弃掉。

### 3.2 使用工厂方法创建

## 4 提交任务

&emsp;&emsp;支持两种方法向任务池提交任务，分别为execute()和submit()方法

#### 4.1 execute()

&emsp;&emsp;execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功。通过以下代码可知execute()方法输入的任务是一个Runnable类的实例。

```java
threadsPool.execute(new Runnable() {
	@Override
	public void run() {
	}
});
```



#### 4.2 submit()

&emsp;&emsp;submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，**并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成**，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

```java
Future<Object> future = executor.submit(harReturnValuetask);
try {
	Object s = future.get();
} catch (InterruptedException e) {
// 处理中断异常
} catch (ExecutionException e) {
// 处理无法执行任务异常
} finally {
// 关闭线程池
	executor.shutdown();
}
```

## 5 关闭线程池

&emsp;&emsp;可以通过调用线程池的$shutdown$或$shutdownNow$方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。

&emsp;&emsp;但是它们存在一定的区别，shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。只要调用了这两个关闭方法中的任意一个，isShutdown方法就会返回true。当所有的任务都已关闭后，才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown方法来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow方法。

## 6 线程池的配置

要想合理地配置线程池，就必须首先分析任务特性，可以从以下几个角度来分析。

- 任务的性质：CPU密集型任务、IO密集型任务和混合型任务。
- 任务的优先级：高、中和低。
- 任务的执行时间：长、中和短。
- 任务的依赖性：是否依赖其他系统资源，如数据库连接。
  性质不同的任务可以用不同规模的线程池分开处理。
  - <font color='red'>**CPU密集型任务应配置尽可能少的线程，如配置Ncpu+1个线程的线程池。从而减少线程切换的开支，保证线程占用CPU的时间**</font>
  - <font color='red'>**由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如2*Ncpu，避免在IO期间浪费CPU的资源**。</font>
  - <font color='red'>**混合型的任务，如果可以拆分，将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果这两个任务执行时间相差太大，则没必要进行分解。**</font>

&emsp;&emsp;可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。
&emsp;&emsp;优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先执行。

> **注意:如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。**

&emsp;&emsp;执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。
&emsp;&emsp;依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时间越长，则CPU空闲时间就越长，**那么线程数应该设置得越大**，这样才能更好地利用CPU。
&emsp;&emsp;**建议使用有界队列**。有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点儿，比如几千。有一次，我们系统里后台任务线程池的队列和线程池全满了，不断抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞，任务积压在线程池里。如果当时我们设置成无界队列，那么线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。

当然，我们的系统所有的任务是用单独的服务器部署的，我们使用不同规模的线程池完成不同类型的任务，但是出现这样问题时也会影响到其他任务。

## 7 线程池的监控

&emsp;&emsp;如果在系统中大量使用线程池，则有必要对线程池进行监控，方便在出现问题时，可以根据线程池的使用状况快速定位问题。可以通过线程池提供的参数进行监控，在监控线程池的时候可以使用以下属性。

- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
- largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过
- getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
- getActiveCount：获取活动的线程数。

&emsp;&emsp;可以通过继承线程池来自定义线程池，重写线程池的beforeExecute、afterExecute和terminated方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。例如，监控任务的平均执行时间、最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。

```java
protected void beforeExecute(Thread t, Runnable r) { }
```


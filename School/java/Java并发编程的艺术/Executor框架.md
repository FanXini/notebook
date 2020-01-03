---
typora-copy-images-to: ..\..\..\img
---

# Executor框架

## 1. Excutor框架的两级调度模型

&emsp;&emsp;在HotSpot VM的线程模型中，java线程和本地操作系统线程是一对一的关系。即创建一个java线程就会会创建一个本地操作系统线程。java线程终止时，操作系统线程也会被回收。

&emsp;&emsp;在上层中，java多线程程序会把一个应用分解成若干任务，然后使用用户级的调度器(Excutor框架)将这些任务映射成固定数量的线程；在底层，操作系统内核将线程映射到硬件处理器上。这种两级调度模型的示意图如下图所示。

![1559557337867](..\..\..\img\1559557337867.png)

&emsp;&emsp;从图中可以看出，应用程序通过Executor框架控制上层的调度；而下层的调度由操作系统内核控制，下层的调度不受应用程序的控制。

## 2 Executor 框架的结构和成员

### 2.1 Executor的结构

- 任务：任务需要实现的接口：Runable接口和Callable接口
- 任务的执行：包括任务执行机制的核心接口Executor，以及继承Executor接口的ExecutorService接口。Executor框架有两个关键类实现了ExecutorService接口（ThreadPoolExecutor和ScheduledThreadPoolExecutor）。
- 异步计算的结果：包括Future接口和实现Future接口的FutureTask类。

### 2.2 类和接口的介绍

- Executor是一个接口，它是Executor框架的基础，将任务的提交和执行分离开来
- ThreadPoolExcutor是线程池的核心实现类，用来执行提交的任务
- ScheduledThreadPoolExecutor是一个实现类，可以在给定的延迟后运行命令，或者定期执行命令。ScheduledThreadPoolExecutor比Timer更灵活，功能更强大。
- Future接口和实现Future接口的FutureTask类，代表异步计算的结果。
- Runnabble接口的实现类，都可以被ThreadPoolExecutor或Scheduled-
  ThreadPoolExecutor执行。

**框架使用示意图：**

![1559561281877](..\..\..\img\1559561281877.png)

&emsp;&emsp;主线程首先要创建实现Runnable或者Callable接口的任务对象。工具类Executors可以把一个Runnable对象封装为一个Callable对象（Executors.callable（Runnable task）或Executors.callable（Runnable task，Object resule））。

&emsp;&emsp;然后可以把Runnable对象直接交给ExecutorService执（ExecutorService.execute（Runnable
command））；或者也可以把Runnable对象或Callable对象提交给ExecutorService执（Executor-
Service.submit（Runnable task）或ExecutorService.submit（Callable<T>task））。

&emsp;&emsp;如果执行ExecutorService.submit（…），ExecutorService将返回一个实现Future接口的对象（到目前为止的JDK中，返回的是FutureTask对象）。由于FutureTask实现了Runnable，程序员也可以创建FutureTask，然后直接交给ExecutorService执行。最后，主线程可以执行FutureTask.get()方法来等待任务执行完成。主线程也可以执行FutureTask.cancel（boolean mayInterruptIfRunning）来取消此任务的执行。

### 2.3 Future接口

&emsp;&emsp;Future接口和实现Future接口的FutureTask类用来表示异步计算的结果。当我们把Runable接口或者Callable接口的实现类提交(submit)给ThreadPoolExcutor或者ScheduledThreadPoolExecutor时，ThreadPoolExecutor或者ScheduledThreadPoolExecutor会返回一个FutureTask对象。

对应的API

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
 Future<?> submit(Runnable task);
```

尽管submit方法能提供线程执行的返回值，但只有**实现了Callable**才会有返回值，而实现Runnable的线程则是没有返回值的，也就是说在上面的3个方法中，submit(Callable<T> task)能获取到它的返回值，submit(Runnable task, T result)能通过传入的载体result间接获得线程的返回值或者准确来说交给线程处理一下，而最后一个方法submit(Runnable task)则是没有返回值的，就算获取它的返回值也是null。

**submit(Runable task,T result)使用示例：**

```java
package com.threadpoolexecutor;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * ThreadPoolExecutor#submit(Runnable task, T result)
 * Created by yulinfeng on 6/17/17.
 */
public class Submit2 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService executor = Executors.newSingleThreadExecutor();
        Data data = new Data();
        Future<Data> future = executor.submit(new Task(data), data);
        System.out.println(future.get().getName());
    }
}

class Data {
    String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

class Task implements Runnable {
    Data data;

    public Task(Data data) {
        this.data = data;
    }
    public void run() {
        System.out.println("This is ThreadPoolExetor#submit(Runnable task, T result) method.");
        data.setName("kevin");
    }
}
```

实际上就是借助一个载体实现值的修改并对开发者所见。

#### FutureTask的使用

&emsp;&emsp;可以把FutureTask交给Executor执行；也可以通过ExecutorService.submit（…）方法返回一个
FutureTask，然后执行FutureTask.get()方法或FutureTask.cancel（…）方法。除此以外，还可以单独使用FutureTask。当一个线程需要等待另一个线程把某个任务执行完后它才能继续执行，此时可以使用FutureTask。假设有多个线程执行若干任务，每个任务最多只能被执行一次。当多个线程试图
同时执行同一个任务时，只允许一个线程执行任务，其他线程需要等待这个任务执行完后才能继续执行。下面是对应的示例代码。

```java
package executor;

import java.util.concurrent.*;

public class FutureTest {
    private final ConcurrentMap<Object, Future<String>> taskCache =
            new ConcurrentHashMap<Object, Future<String>>();

    private String executionTask(final String taskName)
            throws ExecutionException, InterruptedException {
        while (true) {
            Future<String> future = taskCache.get(taskName); // 1.1,2.1
            if (future == null) {
                Callable<String> task = new Callable<String>() {
                    public String call() throws InterruptedException {
                        return taskName;
                    }
                };
                FutureTask<String> futureTask = new FutureTask<String>(task);
                future = taskCache.putIfAbsent(taskName, futureTask);// 1.3
                if (future == null) {
                    future = futureTask;
                    futureTask.run(); // 1.4执行任务
                }
            }
            try {
                return future.get(); // 1.5,2.2} catch (CancellationException e) {

            } catch (Exception e) {
                taskCache.remove(taskName, future);
                e.printStackTrace();
            }
        }
    }

    public static void main(String agrs[])throws  Exception{
        FutureTest test=new FutureTest();
        for(int i=0;i<10;i++){
           System.out.println( test.executionTask("yes"));
        }
    }
}

```

**执行示意图：**

![1559565177674](..\..\..\img\1559565177674.png)

&emsp;&emsp;当两个线程试图同时执行同一个任务时，如果Thread 1执行1.3后Thread 2执行2.1，那么接下来Thread 2将在2.2等待，直到Thread 1执行完1.4后Thread 2才能从2.2（FutureTask.get()）返回。

## 2.4 Runnable接口和Callable接口

共同点：实现这两个的接口的任务，都可以提交给线程池ThreadPoolExecutor执行

不同点：Runnable不可以返回结果，Callable可以放回结果

可以通过Executors提供的API,将一个Runable包装成一个Callable。

```java
 public static Callable<Object> callable(Runnable task) {
        if (task == null)
            throw new NullPointerException();
        return new RunnableAdapter<Object>(task, null);  //callable1
    }

```

假设调用该接口后返回的对象为callable1;

或者将一个Runable和 一个待返回的结果包装成一个Callable

```java
public static <T> Callable<T> callable(Runnable task, T result) {
        if (task == null)
            throw new NullPointerException();
        return new RunnableAdapter<T>(task, result); //callable2
    }
```

假设调用该接口后返回的对象为callable2;

前面讲过，当我们把一个Callable对象（比如上面的Callable1或Callable2）提交给
ThreadPoolExecutor或ScheduledThreadPoolExecutor执行时，submit（…）会向我们返回一个
FutureTask对象。我们可以执行FutureTask.get()方法来等待任务执行完成。当任务成功完成后
FutureTask.get()将返回该任务的结果。例如，如果提交的是对象Callable1，FutureTask.get()方法
将返回null；如果提交的是对象Callable2，FutureTask.get()方法将返回result对象。
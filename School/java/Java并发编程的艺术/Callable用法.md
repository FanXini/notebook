---
typora-copy-images-to: ..\..\..\img
---

# 线程池中使用Callable

jvm在线程池中得到可用线程，接着就可以执行线程的方法，线程默认可以使用实现了Runnable接口的线程。如下代码：

```java
package com.xxx.future;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class NoFuture {
	public static class Tasker implements Runnable{		
		@Override
		public void run(){
			System.out.println("ah...");
		}
	}
	public static void main(String[] args) {
		ExecutorService service = Executors.newFixedThreadPool(3);
		for(int i=0;i<5;i++){
			service.submit(new Tasker());
		}
		service.shutdown();
	}
}
```


执行结果很直观，打印5行ha...

这种方式可以实现多线程，但是无法得到线程执行之后的结果（如果需要的话）。

jdk从1.5开始提供了Callable和Future来实现获取线程执行的结果,先看一段代码:

```java
package com.xxx.future;
 
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class CallableAndFuture {
	
	public static class Tasker implements Callable<String>{
 
		@Override
		public String call() throws Exception {
			return "hello";
		}
		
	}
 
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ExecutorService threadPool = Executors.newFixedThreadPool(3);
		List<Future<String>> futures = new ArrayList<Future<String>>();
		Future<String> res = null;
		for(int i=0;i<5;i++){
			res = threadPool.submit(new Tasker());
			futures.add(res);
		}
		threadPool.shutdown();
		for(Future<String> future:futures){
			System.out.println(future.get());
		}
	}
 
}

```


运行结果如下：



与Runnable不同的是，实现Callable的线程类需要实现call接口，并且给出泛型类,用来匹配线程执行完成的结果。Future则用来接收线程返回的结果，他提供的两个主要方法get(),isDone()分别用来获取结果和判断线程是否执行完成。

   我们知道，线程池虽然可以解决多线程的问题，但是有一个问题，如果线程数量过多,而池中的可用线程不足，线程池会使用一个队列做缓冲区，这样后续的线程可能还没有执行，而我们的请求已经发送了，在线程执行结束这段时间内，我们是得不到这个结果的，如果我们使用Future提供的get()来获取结果，只会让主线程进入阻塞，我们来看一个例子。

```java
package com.xxx.future;
 
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class BlockingTest {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		final ExecutorService service = Executors.newFixedThreadPool(5);
		Future<String> result = service.submit(new Callable<String>() {
			@Override
			public String call() throws Exception {
				//模拟现实中的异步过程，等待五秒返回结果
				Thread.sleep(1000*5);			
				return "ok";
			}		
		});
		System.out.println("task begin...");
		//如果线程没有执行完成，那么继续等待，通过打印信息来直观感受线程执行过程
		while(!result.isDone()){
			System.out.println("wait for 1 second");
			Thread.sleep(1000);
		}
		String ok = result.get();
		service.shutdown();
		System.out.println("task end "+ok);
	}
 
}

```



执行结果：

![1547558579562](..\..\..\img\1547558579562.png)

这个例子很好的说明了，如果线程执行需要时间，或者并发量大的情况下，用户请求可能不能立马得到结果，这种情况下我们会先给用户一个中性的提示，等到线程执行完成，再更改相应的状态。

作者：luffy5459 
来源：CSDN 
原文：https://blog.csdn.net/feinifi/article/details/78194373 
版权声明：本文为博主原创文章，转载请附上博文链接！
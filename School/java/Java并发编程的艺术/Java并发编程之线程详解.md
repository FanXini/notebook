[TOC]



# Java并发编程之线程详解

## 1.什么是线程？

&emsp;&emsp;现代操作系统在运行一个程序时，会为其创建一个进程。比如打开QQ音乐，视频软件等待。而现代操作系统调度(执行)的最小单位是线程，也叫轻量级进程。一个进程里由多个线程分工合作来完成进程的功能，这些线程都拥有各自的程序计数器、堆栈和局部变量表等属性，并且能访问共享的内存变量。有关线程和进程的关系，[可以参看这篇文章](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)**(强烈推荐)**

## 2.为什么要使用多线程

1. 充分利用多处理器核心

   如今处理器的发展从升频转到了多核上来，现在稍好的笔记本都是4核起步。而一个线程只能在一个核上运行，如果程序是一个单线程程序，那么即使你的处理器核数再多，性能再好，处理速度也得不到加快。因此，使用多线程，可以充分利用处理器的性能，多核同时执行多个线程，能提高程序运行的速度。

2. 更快的响应时间

   多任务同时执行，执行效率优于单任务执行。

3. 更好编程模型

   Java提供了非常容易上手的编程模型，程序员可以把精力集中到问题求解的建模上来，模型建好了，很容易使用Java提供的编程模型来解决问题。

## 3.线程的状态

java线程有6个状态:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190422150523754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
线程状态的切换如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190422150543185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
解析：线程在调用start()的方法后，由NEW状态进入到RUNNING(运行中)状态,当线程调用object.wait()方法后，线程会进入等待状态，进入等待状态的线程需要其它线程的通知才能恢复到运行状态。相似的还有wait(time)方法，区别的是线程等待时间超过time后会自动返回到运行状态，线程在需要运行临界区代码时需要获取对象的锁，获取不到会进入阻塞状态(也可能不进入阻塞状态，通过自旋获取锁)，获取到锁后返回运行状态，线程运行完后进入终止状态。

## 4.线程的生命周期

### 4.1线程的创建

java支持两种线程的创建方式：

1. 继承Thread类
2. 实现Runable接口

JDK1.8  java.lang.Thread中线程初始化init()方法：

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;
	//当前线程就是该线程的父线程
    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
           what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
           use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
       explicitly passed in. */
    g.checkAccess();

    /*
     * Do we have the required permissions?
     */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g;
    //继承父线程的daemon、priority属性
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
            acc != null ? acc : AccessController.getContext();
    this.target = target;
    setPriority(priority);
    // 将父线程的InheritableThreadLocal复制过来
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    //分配一个线程id
    tid = nextThreadID();
}
```

&emsp;&emsp;在上述过程中，一个新构造的线程对象是由其parent线程来进行空间分配的，而child线程继承了parent是否为Daemon、优先级和加载资源的contextClassLoader以及可继承的ThreadLocal，同时还会分配一个唯一的ID来标识这个child线程。至此，一个能够运行的线程对象就初始化好了，在堆内存中等待着运行。

### 4.2 线程的启动

&emsp;&emsp;线程对象在初始化完成之后，调用start()方法就可以启动这个线程。线程start()方法的含义是：当前线程（即parent线程）同步告知Java虚拟机，只要线程规划器空闲，应立即启动调用start()方法的线程。
**注意**：

&emsp;&emsp;启动一个线程前，最好为这个线程设置线程名称，因为这样在使用jstack分析程序或者进行问题排查时，就会给开发人员提供一些提示，自定义的线程最好能够起个名字。

### 4.3 线程的中断

&emsp;&emsp;调用线程的interruput()方法可以对线程进行中断。线程通过检测自身的中断标识符来判断是否响应中断，通过调用

```java
public boolean isInterrupted() {
    return isInterrupted(false); //参数：CleanInterruped
}
```

来判断当前是否被中断，return false表示没有，return ture表示中断。

可以调用Thread的静态方法Thread.interrupted()来对中断标识位进行复位。

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true); //参数CleanInterruped
}
```

&emsp;&emsp;从Java的API中可以看到，许多声明抛出InterruptedException的方法（例如Thread.sleep(longmillis)方法）这些方法在抛出InterruptedException之前，Java虚拟机会先将该线程的中断标识位清除，然后抛出InterruptedException，此时调用isInterrupted()方法将会返回false。

&emsp;&emsp;使用interrupt()方法只能中断处于阻塞状态的线程，但是不能中断处于运行中状态的线程。要想中断运行中的线程，需要配合isInterrupted()方法。在4.4会介绍到。

### 4.4 线程的暂停、恢复和停止

| Method    | Description |
| --------- | ----------- |
| suspend() | 线程的暂停  |
| resume()  | 线程的恢复  |
| stop()    | 线程的停止  |

这三个API已经过期了，不建议使用。原因：

&emsp;&emsp;以suspend()方法为例，在调用后，线程不会释放已经占有的资源（比如锁），而是占有着资源进入睡眠状态，这样容易引发死锁问题。同样，stop()方法在终结一个线程时不会保证线程的资源正常释放，通常是没有给予线程完成资源释放工作的机会，因此会导致程序可能工作在不确定状态下。

**暂停和恢复操作可以用后面提到的<font color='red'>等待/通知机制</font>来替代。**

而如何安全的终止线程呢？

一个是使用4.3提到的中断方式，一个是使用状态标识符，示例代码如下：

```java
public class Shutdown {
	public static void main(String[] args) throws Exception {
		Runner one = new Runner();
		Thread countThread = new Thread(one, "CountThread");
		countThread.start();
		// 睡眠1秒，main线程对CountThread进行中断，使CountThread能够感知中断而结束
		TimeUnit.SECONDS.sleep(1);
		countThread.interrupt();
		Runner two = new Runner();
		countThread = new Thread(two, "CountThread");
		countThread.start();
		// 睡眠1秒，main线程对Runner two进行取消，使CountThread能够感知on为false		而结束
		TimeUnit.SECONDS.sleep(1);
		two.cancel();
	}
    
	private static class Runner implements Runnable {
		private long i;
		private volatile boolean on = true;
		@Override
		public void run() {
		while (on && !Thread.currentThread().isInterrupted()){
			i++;
		}
		System.out.println("Count i = " + i);
	}
        
	public void cancel() {
		on = false;
	}
}


```

&emsp;&emsp;示例在执行过程中，main线程通过中断操作和cancel()方法均可使CountThread得以终止。这种通过标识位或者中断操作的方式能够使线程在终止时有机会去清理资源，而不是武断地将线程停止，因此这种终止线程的做法显得更加安全和优雅。

## 5.线程间的通信

&emsp;&emsp;线程之间的通信是通过隐士的共享变量的方式。每个线程都可以使用对堆内存中的共享变量，但同时线程栈内存中也会有共享变量的拷贝，原因是加速程序的执行，现代处理器会使用缓存区的技术避免线程长时间等待内存的读写。

### 5.1 利用volitale和synchonized关键字

&emsp;&emsp;通过volitale和锁的使用可以保证共享变量的可见性(A线程对共享变量的修改能马上后续使用到该变量的线程可见)。

&emsp;&emsp;例如被volitale修饰的变量，可以保证当线程对该变量进行写操作时会把线程缓存区的所有数据全部刷新到共享内存中去，当线程对该变量进行读操作时会把线程缓存区的数据置为无效，重新区共享内存中读取新的数据，这样就能避免线程使用的数据是过时的数据。

&emsp;&emsp;同理:被synchonized修饰的方法或代码块或使用Concurent包Lock类进行显式加锁解锁的代码块能保证在任意时刻都只能有一个线程进入到方法或者代码块中，从而保证共享变量的可见性。

因此对于这段伪代码：

```java
volitale int a=1;

public void increment(){  //A线程
     a++;
}
public void output(){ //B线程
 	if(a==2){ 
    	... 
    }else{
        
    }   
}
```

A线程调用increment( )进行了a+1的操作，并写回了共享内存，而后B线程运行了output，读取到的a是A线程+1后的值，于是进入第一个分支。因此，这就相当于A线程向B线程发出了一个通信。

### 5.2 等待通知机制

&emsp;&emsp;前面提到过，线程的暂停(suspend)和恢复(resume)API不推荐使用，因为不能保证资源的安全释放。因此，这个介绍一个等待通知机制来代替这两个功能。而等待通知机制不仅仅是完成线程等待恢复的功能，它的作用可以被程序员的水平无限扩大。

&emsp;&emsp;假设线程A修改了一个共享变量，B线程感应到这个变化后执行相应的操作，然后进行相应的操作，整个过程开始于一个线程，而最终执行又是另一个线程。前者是生产者，后者就是消费者，这种模式隔离了“做什么”（what）和“怎么做”（How），在功能层面上实现了解耦，体系结构上具备了良
好的伸缩性，但是在Java语言中如何实现类似的功能呢？

&emsp;&emsp;简单的办法是让消费者线程不断地循环检查变量是否符合预期，如下面代码所示，在while循环中设置不满足的条件，如果条件满足则退出while循环，从而完成消费者的工作。

```java
while (value != desire) {
	Thread.sleep(1000);
}
doSomething();
```

缺点：

1. 及时性得不到保证
2. 开销大，浪费计算资源

&emsp;&emsp;以上两个问题，看似矛盾难以调和，但是Java通过内置的等待/通知机制能够很好地解决这个矛盾并实现所需的功能。等待/通知的相关方法是**任意Java对象**都具备的，因为这些方法被定义在所有对象的超类java.lang.Object上，方法和描述如下表所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190422150615588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;等待/通知机制，是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而执行后续操作。上述两个线程通过对象O来完成交互，而对象上的wait()和notify/notifyAll()的关系就如同开关信号一样，用来完成等待方和通知方之间的交互工作。

使用示例：

&emsp;&emsp;所示的例子中，创建了两个线程——WaitThread和NotifyThread，前者检查flag值是否为false，如果符合要求，进行后续操作，否则在lock上等待，后者在睡眠了一段时间后对lock进行通知，示例如下所示

```java
package 并发学习;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.TimeUnit;


public class WaitNotify {
	static boolean flag=true;
	static Object lock=new Object();
	public static void main(String args[]){
		WaitNotify waitNotify=new WaitNotify();
		Thread waitThread=new Thread(waitNotify.new Wait(),"waitThread");
		waitThread.start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		Thread notifyThread=new Thread(waitNotify.new Notify(),"notifyThread");
		notifyThread.start();
		
	}
	class Wait implements Runnable{
		@Override
		public void run(){
			//加锁，拥有lock的Monitor
			synchronized (lock) {
				while(flag){
					System.out.println(Thread.currentThread()+"flag is true .wait @"
				    +new SimpleDateFormat("HH:mm:ss").format(new Date()));
					try {
						//wait()将会释放lock对象的锁
						lock.wait();       
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				System.out.println(Thread.currentThread()+"flag is false .running @"+
				new SimpleDateFormat("HH:mm:ss").format(new Date()));
			}
		}
	}
	
	class Notify implements Runnable{
		@Override
		public void run(){
			//得到lock对象的锁
			synchronized (lock) {
				System.out.println(Thread.currentThread()+"hold lock .notify @"+new SimpleDateFormat("HH:mm:ss").format(new Date()));
				lock.notifyAll();
				flag=false;
				try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			//重新获得锁
			synchronized (lock) {
				System.out.println(Thread.currentThread()+"hold lock again .sleep"+new SimpleDateFormat("HH:mm:ss").format(new Date()));
				try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}	
}
```

输入如下：

```
Thread[WaitThread,5,main] flag is true. wait @ 22:23:03
Thread[NotifyThread,5,main] hold lock. notify @ 22:23:04
Thread[NotifyThread,5,main] hold lock again. sleep @ 22:23:09
Thread[WaitThread,5,main] flag is false. running @ 22:23:14
```

上述第3行和第4行输出的顺序可能会互换，而上述例子主要说明了调用wait()、notify()以及notifyAll()时需要注意的细节，如下。

1. **使用wait()、notify()和notifyAll()时需要先对调用对象加锁。**
2. 调用wait()方法后，线程状态由RUNNING变为WAITING，**并将当前线程放置到对象的**
   **等待队列。**
3. notify()或notifyAll()方法调用后，等待线程依旧不会从wait()返回，需要调用notify()或
   notifAll()的线程释放锁之后，等待线程才<font color='red'>**有机会**</font>从wait()返回。
4. notify()方法将等待队列中的一个等待线程从等待队列中移到同步队列中，而notifyAll()
   方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态由WAITING变为
   BLOCKED。
5. 从wait()方法返回的**前提是获得了调用对象的锁。**

[阿里巴巴面试题： 为什么wait()和notify()需要搭配synchonized关键字使用](https://blog.csdn.net/lengxiao1993/article/details/52296220)

上述代码的运行流程图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190422150639543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;WaitThread首先获取了对象的锁，然后调用对象的wait()方法，从而放弃了锁并进入了对象的等待队列WaitQueue中，进入等待状态。由于WaitThread释放了对象的锁，NotifyThread随后获取了对象的锁，并调用对象的notify()方法，将WaitThread从WaitQueue移到SynchronizedQueue中，此时WaitThread的状态变为阻塞状态。NotifyThread释放了锁之后，WaitThread再次获取到锁并从wait()方法返回继续执行。

### 5.3 等待/通知的经典范式

从5.2节中的WaitNotify示例中可以提炼出等待/通知的经典范式，该范式分为两部分，分别针对等待方（消费者）和通知方（生产者）。
等待方遵循如下原则。
1）获取对象的锁。
2）如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件。
3）条件满足则执行对应的逻辑。
对应的伪代码如下：

```java
synchronized(对象) {
	while(条件不满足) {
		对象.wait();
	}
	对应的处理逻辑
}
```

通知方遵循如下原则。
1）获得对象的锁。
2）改变条件。
3）通知所有等待在对象上的线程。
对应的伪代码如下:

```java
synchronized(对象) {
	改变条件
	对象.notifyAll();
}
```

### 5.4 管道输入输出流

&emsp;&emsp;管道输入/输出流和普通的文件输入/输出流或者网络输入/输出流不同之处在于，它主要用于线程之间的数据传输，而传输的媒介为内存。
&emsp;&emsp;管道输入/输出流主要包括了如下4种具体实现：PipedOutputStream、PipedInputStream、PipedReader和PipedWriter，前两种面向字节，而后两种面向字符。

使用示例：

```java
import java.io.IOException;
import java.io.PipedReader;
import java.io.PipedWriter;
import java.util.Scanner;

public class PipTest {

	private static Scanner scanner=new Scanner(System.in);
	public static void main(String agrs[]) throws IOException{
		PipedReader reader=new PipedReader();
		PipedWriter writer=new PipedWriter();
		new Thread(new Print(reader),"reader").start();
		//通过connect将pip通道连接
		reader.connect(writer);
		//读取键盘输入
		while(true){
			String content=scanner.next();
			writer.write(content);
		}
	}
	static class Print implements Runnable{
		private PipedReader reader;
		public Print(PipedReader reader) {
			this.reader=reader;
		}
		@Override
		public void run() {
			int receive=0;
			try {
				while((receive=reader.read())!=-1){
					System.out.print((char) receive);
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
			
		}
		
	}

}

```

运行该示例，输入一组字符串，可以看到被printThread进行了原样输出。

```
HELLO WORLD
HELLO WORLD
```

&emsp;&emsp;对于Piped类型的流，必须先要进行绑定，也就是调用connect()方法，如果没有将输入/输出流绑定起来，对于该流的访问将会抛出异常。
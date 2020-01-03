# Java内存模型

在并发编程中，需要处理两个关键的问题：

> 1. 线程之间是如何通信的。
> 2. 线程之间是如何同步的。

线程之间的通信是指线程之间通过何种机制来进行信息的传递，目前有两种方式通过进行线程间的通信

- 共享内存的方式：通过写-读共享内存中的公共状态来进行隐式的通信。
- 消息传递的方式：在基于消息传递的并发模型里，线程之间没有共享内存，线程之间必须通过传递消息来进行显示的通信。

2.线程之间的同步指的是控制不同的线程间的操作发生相对顺序的机制。

- 在基于共享内存模型里面，因为线程之间的通信时隐性的，所以程序员必须显示的指定某段代码或方法中线程的相对执行顺序。
- 在基于消息传递的模型里面，由于消息的发送一定是在消息的接受之前，所以，线程之间的同步是隐式的。

而java的内存模型是基于**共享内存**的方式，因此线**程之间的通信是隐式**进行的**，同步是显示进行的**。

**JMM的作用：**

JMM对正确同步的多线程程序的内存一致性做了如下保证:

1. 对于单线程程序：单线程程序不会出现数据可见性问题，JMM保证程序的执行结果和顺序一致性模型的执行结果一致。
2. 对于正确同步的多线程程序：JMM保证正确同步的多线程程序的执行结果和顺序一致性模型的执行结果一致。JMM通过添加内存屏障限制编译重排序和处理器重排序来保证这一点。
3. 对于未正确同步的多线程程序：JMM提供最小的保证：线程执行时读取到的值要么时之前某个线程写入的，要么是默认值(0,null,false)

## 理想模型:顺序一致性内存模型

&emsp;&emsp;顺序一致性内存模型是一个被计算机科学家理想化了的理论参考模型，它为程序员提供了极强的内存可见性保证。顺序一致性内存模型有两大特性。
1）一个线程中的所有操作必须按照程序的顺序来执行。
2）（不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。

线程使用共享变量的视图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221724715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
任意时刻只有一个线程能使用到共享内存，这可以保证任意一个线程的操作都立刻被后续使用共享内存的线程可见。

而java的内存模型并非如此。现代处理器为了提高处理性能，往往在线程和内存之间增加一个高速缓存区，避免处理器长期处理等待内存读写的状态。

## 读/写缓存区对内存可见性的影响

现代的计算机处理器都支持使用缓存区临时保存向内存写入的数据

```
  优点：因为CPU的处理速度远远大于内存的处理速度，因此在CPU与内存之间增加一个读/写缓存区来进行过渡，避免处理器停顿下来等待向内存写入数据而产生的延迟。同时通过批处理的方式刷新写缓冲区，不仅可以减少对系统总线的占用，同时可以合并对写缓冲区同一数据的多次更新。

缺点：写缓存区虽然有上述优点，但是，每个线程的缓存区只对自己可见，不对其他线程可见。这个特性会产生一个问题，处理器对内存的读写操作的执行顺序，不一定与内存实际发生的读/写操作顺序一致！这就会导致内存的可见性问题。

例如:初始状态（a=b=0）

A线程：a=1;x=b;

B线程:   b=2;y=a;

会出现这样的结果:x=y=0;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221744288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp; 从内存操作实际发生的顺序来看，直到处理器A执行A3来刷新自己的写缓存区，写操作A1才算真正执行了。虽然处理器A执行内存操作的顺序为：A1→A2，但内存操作实际发生的顺序却A2→A1。此时，处理器A的内存操作顺序被重排序了（处理器B的情况和处理器A一样，这里就不赘述了）。

&emsp;&emsp;这里的关键是，由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。由于现代的处理器都会使用写缓冲区，因此现代的处理器都会允许对写-读操作进行重排序。

下图是常见处理器允许的重排序类型的列表。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221758714.png)
注意，N”表示处理器不允许两个操作重排序，“Y”表示允许重排序。
从上图我们可以看出：常见的处理器都允许Store-Load重排序；常见的处理器都不允许对存在数据依赖的操作做重排序。sparc-TSO和X86拥有相对较强的处理器内存模型，它们仅允许对写-读操作做重排序（因为它们都使用了写缓冲区）。

## java内存模型(JMM)的抽象结构

&emsp;&emsp;因此，JMM也采取了增加缓冲区的做法，在java中，所有实例域、静态域、和数组元素都存储在堆内存中，堆内存在线程之间共享，也就是共享内存。而局部变量、方法定义参数和异常处理器参数不会在线程之间共享，他们不会有内存的可见性问题，也不受内存模型的影响。

&emsp;&emsp;每一个线程都运行在栈内存中，每个线程都有自己的工作内存（Working Memory）,比如寄存器Register、高速缓冲存储器Cache等，线程的计算一般都是通过工作内存进行交互的，而不是直接和共享内存进行交互，如图所示

![img](https://img-blog.csdn.net/20180403162719916?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这就导致线程对共享变量的操作仅保存在本地工作内存，而没有刷新到共享内存中去，那么对于其他线程而言，这些操作并不可见，因此会导致数据一致性问题。

## 从源代码到指令序列的重排序

&emsp;&emsp;在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序，重排序分为三种：

- 编译器优化的重排序。编译器在不更改语义的情况下，可以更改语句的执行顺序。
- 指令级的重排序。现代处理器采用了指令级并行技术(Instruction-Level Parallelism,ILP)来将多条指令重叠执行。如果不存在数据的依赖性，处理器可以更改指令的执行顺序
- 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和内存操作看上去是乱序排序。

从Java源代码到最终实际执行的指令序列，会依次经历下面三种重排序：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221828666.png)
&emsp;&emsp;上述的1属于编译器重排序，2和3属于处理器重排序。**这些重排序可能会导致多线程程序出现内存可见性问题**。请看下面的示例代码

```java
class ReorderExample {
	int a = 0;
	boolean flag = false;
	public void writer() {
		a = 1; // 1
		flag = true; // 2
	}
	Public void reader() {
		if (flag) { // 3
		int i = a * a; // 4
		……
		}
	}
}
```

解析：

> flag 变量是一个标记，用来标识a是否被写入。假设有两个线程A、B，A首先执行writer()方法。随后B线程接着执行reader()方法。线程B在执行操作4时，能否看到线程A在操作1时对共享变量a的写入？
>
> 答案：**不一定**

原因：由于操作1和操作2没有数据依赖关系，编译器和处理器可以对这两个操作重排序；同样，
操作3和操作4没有数据依赖关系，编译器和处理器也可以对这两个操作重排序。让我们先来
看看，当操作1和操作2重排序时，可能会产生什么效果？请看下面的程序执行时序图，如图下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221849999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;如上图所示，操作1和操作2做了重排序。程序执行时，线程A首先写标记变量flag，随后线
程B读这个变量。由于条件判断为真，线程B将读取变量a。此时，变量a还没有被线程A写入，在
这里多线程程序的语义被重排序破坏了！

&emsp;&emsp;当操作3和4发生重排序时，又会发生什么情况？程序执行时序图如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221905460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;在程序中，操作3和操作4存在控制依赖关系。当代码中存在<font color='red'>**控制依赖性**</font>时，会影响指令序
列执行的并行度。为此，编译器和处理器会采用**猜测（Speculation）执行**来克服控制相关性对并
行度的影响。以处理器的猜测执行为例，执行线程B的处理器可以提前读取并计算a*a，然后把
计算结果临时保存到一个名为重排序缓冲（Reorder Buffer，ROB）的硬件缓存中。当操作3的条
件判断为真时，就把该计算结果写入变量i中。

&emsp;&emsp;从上图中我们可以看出，猜测执行实质上对操作3和4做了重排序。重排序在这里破坏了多线程程序的语义！

&emsp;&emsp;所以，JMM的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令序列时，插入特定类型的**内存屏障**（Memory Barriers，Intel称之为Memory Fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序。

&emsp;&emsp;JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。



## JMM中的内存屏障

&emsp;&emsp;为了保证内存可见性，Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM把内存屏障指令分为4类，如下图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221922178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;StoreLoad Barriers是一个“全能型”的屏障，它同时具有其他3个屏障的效果。现代的多处理器大多支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（Buffer Fully Flush）。

## happens -before 简介

&emsp;&emsp;如果一个操作的执行结果要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。这里提到的两个操作可以在是一个线程之内的，也可以是在两个不同线程之间的。而happens-before也是基于JMM对某些特性重排序的禁止来实现的。

&emsp;&emsp;与程序员密切相关的happens-before规则如下：

- 程序顺序规则：一个线程中的每个操作， happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- 传递性：如果A happens-before B,且B happens-before C，那么A happens-before C。

**注意**：两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个
操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一
个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。
happens-before的定义很微妙，后文会具体说明happens-before为什么要这么定义。

**happens-before与JMM的关系如下图所示：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417221942999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;一个happens-before规则对应于一个或多个编译器和处理器重排序规则。对于Java程序员来说，happens-before规则简单易懂，它避免Java程序员为了理解JMM提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现方法。

## as-if-serial语义

> as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）
> 程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。

&emsp;&emsp;为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因
为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被
编译器和处理器重排序。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417222001744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
上面3种情况，只要重排序两个操作的执行顺序，程序的执行结果就会被改变。因此，as-if-serial需要禁止这种存在数据依赖关系的重排序。

## volatile的内存语义

&emsp;&emsp;当声明共享变量为volatile后，对这个变量的读/写将会很特别。为了揭开volatile的神秘面
纱，下面将介绍volatile的内存语义及volatile内存语义的实现。

```
volatile自身具有以下特性：
```

- 可见性。对一个volatile变量的读，总是能看到任意线程对这个volatile变量最后的写入。
- 原子性。对任意一个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

但volitile对线程内存可见性的影响比volatile自身的特性更为重要，也更需我们去关注。

- **volatile写的内存语义如下：**

  当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值全部刷新到主内存。

- **volatile读的内存语义如下：**

  当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中去读取共享变量。

**总结：**

&emsp;&emsp;也就是说A线程写一个volitile变量后，B线程读取同一个volatile变量，在A线程写volitle操作之前的任何共享变量，在B线程读取同一个volatile变量之后，将立刻对B线程可见。

举一个例子：

```java
public class VolatileExample {

	private volatile boolean flag=false;
	private int a=0;
	
	public void write(){ 
		a=1;             //1
		flag=true;       //2
	}
	
	public void read(){
		if(flag){
			int i=a;  //3
			System.out.println(i);   //4     输出1
		} 
	}
}   
```

[外链图片转存失败(img-SYpBGn1C-1564967327724)(data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)]

在这段代码里面，根据happen-before规则，这个过程建立的happen-before规格如下：

1.根据程序次序规则：1 happen-before 2,3 happen-before 4;

2.根据volatile读写规格：2 happen-before 3

3.根据传递性规则：1 happen-before 4;

&emsp;&emsp;因此A线程中对a的写，永远对B线程可见。即使a不是volatile变量。这就是flag这个volatile变量对内存可见性的作用，当对一个线程对volatile变量进行写操作时，会把该线程本地内存中的共享变量全部刷新到主内存中去，当一个线程读一个volatile变量时，JMM会将该线程的本地内存置为无效，因此线程会去共享内存中读取变量。所以在上面例子中，B线程读取a的值，A已经刷新到主内存中的新值1。

**对volatile的内存语义进行总结：**

- **线程A写一个volatile变量，实际上是线程A向接下来要读这个volatile变量的某个线程发出了(其对共享变量所做修改的)消息。**
- **线程B读一个volatile变量，实际上是接收了之前某个线程所发出的（在写这个volatile变量之前对共享变量所做修改的消息）**
- **线程A写一个volatile变量，随后线程B读一个volatile变量，这个过程实际上就是线程A通过主内存向线程B发送消息。**

## volatile内存语义的实现

之前在我的《java内存模型》中已经提到过，重排序分为两种，编译重排序和处理器重排序。

而volatile内存语义的实现，是JMM通过限制这两种类型的某种重排序来实现的.

volatile的重排序规则表:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417222044689.png)

1. 如果第二个操作是volatile写，那么不管第一个操作是什么，都不能进行重排序。理解：因为JMM要赋予给volatile写的内存语义是**将该线程的本地内存个刷新到主内存去，**所以要保证在volatile写之前所修改的共享变量值全部能够刷新到共享内存中去，因此不能和之前的操作进行重排序。比如 有两个变量。int a =1;volatile boolean flag=false;进行两个操作：a=0;flag=true;如果允许进行重排序的话，执行顺序可能为flag=true;a=0；此时共享变量a的值将无法刷新到主内存中去，就无法满足volatile写的语义了。
2. 如果第一个操作是volatile读，则不允许和后面任何操作进行重排序。理解：因为JMM要赋予个volatile读的内存语义是能够接收上一个线程（写过同一个volatile变量）所发出的修改过共享变量的消息。它要保证volatile读之后的操作所使用的共享变量都是最新的。如果允许volatile变量读和之后的操作进行重排序的话，将不能保证之后的操作所使用到的共享变量是从主内存中所获取到的更新后的变量。比如有一段代码：

```java
public class VolatileExample {
	private volatile boolean flag=false;
	private int a=0;

	public void write(){
		a=1;  //1
		flag=true;  //2
	}

	public void read(){
		if(flag){  //3
			int y=a;//4
			System.out.println(y);
		}
}}
//解析：
//在这里,A线程先调用write()方法，随后B进程调用read()方法，按照volatile读和写的内存语义，4操作y取得的值应该是更新后a的值1，但是如果允许volatile读和之后的操作进行重排序，即允许3和4进行重排序，那么y得到的值就不一定是A线程对a更新后的新值，而是本地内存中的a的值0，因为此时还没有执行volatile读操作，本地内存的值还有效。
```

1. 如果第一个操作是volatile写，第二个操作是volatile读是，不能进行重排序。理解：第一个操作是volatile写，要保证之前修改的所有共享变量刷新到主内存中去，第二个操作是volatile读，该线程的本地内存置为无效状态，因此要去主内存中去读取共享变量，此时读取到的共享变量值为volatile写之前对共享变量修改后的新值。因此，这两个顺序肯定不能换，更换的话volatile写更新后的值并没有被volatile读后面的操作获取到。

而禁止这些重排序，是通过编译器在生成字节码时，在指令序列中插入**内存屏障**来实现的。
JMM保守的内存屏障插入策略：

```
volatile写之前插入StoreStore屏障：防止volatile写之前对共享变量的写先于volatile写。

volatile写之后插入StoreLoad屏障：防止volatile写与下面可能有的volatile读/写重排序。

volatile读之后插入一个LoadLoad屏障：防止volatile读与下面可能有的普通读操作重排序。

volatile读之后插入一个LoadStore屏障:防止volatile读与下面可能有的普通写操作重排序。
```

而不同的处理器有不同的“松紧度”内存模型，在x86处理器中，仅会对写-读进行重排序，而不会对读-读、写-写、读-写进行重排序，因此，JMM只需要在volatile写后面增加Store-Load内存屏障即实现volatile的内存语义。

## 锁的内存语义

 众所周知，锁可以实现临界区的互斥执行。

```
 java中锁的内存语义和volatile的内存语义类似，其加锁的内存语义和volatile读内存语义相同，其解锁的内存语义和volatile写的内存语义相同，而java锁内存语义的实现又和volatile有着千丝万缕的联系，我们接下来就一起解开java锁的神秘面纱。
```

**锁的释放：**

```
    **当线程释放锁时，JMM会把该线程的本地内存中的共享变量刷新到主内存中去。**
```

**锁的获取**

```
    **当线程获取锁时，JMM会把该线程的本地内存置为无效，线程需要访问主内存去获取共享变量。**
```

**总结：**

- **线程A释放一个锁，实际上是该线程向接下来要获取这个锁的线程发送（线程A对共享变量进行过修改的）消息。**
- **线程B获取一个锁，实际上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。**
- **线程A释放锁，线程B获取找哥哥锁，这个过程实质上是线程A通过主内存向线程发送消息。**

**是不是和上篇博客《volatile内存语义》中介绍的volatile内存语义很像呢？**

## 锁内存语义的实现

解析重入锁ReentranLock的源代码为例：

```java
class ReentrantLockExample {
	int a = 0;
	ReentrantLock lock = new ReentrantLock();
	public void writer() {
		lock.lock();　　　　 // 获取锁
		try {
			a++;
		} finally {
			lock.unlock();　　// 释放锁
		}
	}
	public void reader () {
		lock.lock();　　　　 // 获取锁
		try {
		int i = a;
         ...
		} finally {
		lock.unlock();　 // 释放锁
		}
	}
}
```

&emsp;&emsp;ReentrantLock的实现依赖于Java同步器框架AbstractQueuedSynchronizer（本文简称之为
AQS）。AQS使用一个整型的volatile变量（命名为state）来维护**同步状态**，马上我们会看到，这
个volatile变量是ReentrantLock内存语义实现的关键。

&emsp;&emsp;ReentrantLock锁有公平锁和非公平锁，区别在于获取锁的先后顺序是否与申请锁的先后顺序一致。

### 公平锁

#### 加锁

获取锁源代码：

```java
 protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();   //获取锁的开始，读volatile变量state
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

解锁：

```java
 protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c); //释放锁，写volatile变量
            return free;
        }
```

&emsp;&emsp;公平锁在释放锁的最后写volatile变量state，在获取锁时首先读这个volatile变量。根据volatile的happens-before规则，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变得对获取锁的线程可见。
现在我们来分析非公平锁的内存语义的实现。

### 非公平锁

非公平锁的释放和公平锁完全一样，所以这里仅仅分析非公平锁的获取。

#### 加锁

```java
  final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) { //cas更新状态
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

> CAS:如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。此操作具有volatile读和写的内存语义。

因此，可以得知，锁的内存语义实现至少有下面两种方式：

1. 利用volitaile变量的写-读所具有的内存语义
2. 利用CAS所附带的volitaile变量的写-和读内存语义

## final域的内存语义和实现

对于final域，编译器和处理器要遵守两个重排序规则。

1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用
   变量，这两个操作之间不能重排序。
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能
   重排序。

**下面通过一些示例性的代码来分别说明这两个规则。**

```java
public class FinalExample {
	int i;　　　　　　　　　　 // 普通变量
	final int j;　　　　　　　　 // final变量
	static FinalExample obj;
	public FinalExample () {　　 // 构造函数
		i = 1;　　　　　　　　 // 写普通域
		j = 2;　　　　　　　　 // 写final域
	}
	public static void writer () {　 // 写线程A执行
		obj = new FinalExample ();
	}
	public static void reader () {　 // 读线程B执行
		FinalExample object = obj; // 读对象引用
		int a = object.i;　　　　　 // 读普通域
		int b = object.j;　　　　　 // 读final域
	}
}
```

这里假设一个线程A执行writer()方法，随后另一个线程B执行reader()方法。下面我们通过
这两个线程的交互来说明这两个规则。

### 写final域的重排序

&emsp;&emsp;写final域的重排序规则禁止把final域的写重排序到构造函数之外。这个规则的实现包含下面2个方面。
1）JMM禁止编译器把final域的写重排序到构造函数之外。
2）编译器会在final域的写之后，构造函数return之前，插入一个**StoreStore**屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。
&emsp;&emsp;现在让我们分析writer()方法。writer()方法只包含一行代码：finalExample=new FinalExample()。这行代码包含两个步骤，如下。
1）构造一个FinalExample类型的对象。
2）把这个对象的引用赋值给引用变量obj。
假设线程B读对象引用与读对象的成员域之间没有重排序（马上会说明为什么需要这个假
设），下图是一种可能的执行时序。
&emsp;&emsp;在下图中，写普通域的操作被编译器重排序到了构造函数之外，读线程B错误地读取了
普通变量i初始化之前的值。而写final域的操作，被写final域的重排序规则“限定”在了构造函数
之内，读线程B正确地读取了final变量初始化之后的值。
&emsp;&emsp;写final域的重排序规则可以**确保**：**在对象引用为任意线程可见之前，对象的final域已经被**
**正确初始化过了，而普通域不具有这个保障**。以上图为例，在读线程B“看到”对象引用obj时，
很可能obj对象还没有构造完成（对普通域i的写操作被重排序到构造函数外，此时初始值1还
没有写入普通域i）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417222124114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)

### 读final域的重排序规则

&emsp;&emsp;读final域的重排序规则是，在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止处理器重排序这两个操作（注意，这个规则仅仅针对处理器）。编译器会在读final域操作的前面插入一个LoadLoad屏障。
&emsp;&emsp;初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作。大多数处理器也会遵守间接依赖，也不会重排序这两个操作。但有少数处理器允许对存在间接依赖关系的操作做重排序（比如alpha处理器），这个规则就是专门用来针对这种处理器的。
reader()方法包含3个操作:

- 初次读引用变量obj。
- 初次读引用变量obj指向对象的普通域j。
- 初次读引用变量obj指向对象的final域i。
  现在假设写线程A没有发生任何重排序，同时程序在不遵守间接依赖的处理器上执行，下图所示是一种可能的执行时序。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417222144289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
  &emsp;&emsp;在上图中，读对象的普通域的操作被处理器重排序到读对象引用之前。读普通域时，该
  域还没有被写线程A写入，这是一个错误的读取操作。而读final域的重排序规则会把读对象
  final域的操作“限定”在读对象引用之后，此时该final域已经被A线程初始化过了，这是一个正
  确的读取操作。
  &emsp;&emsp;读final域的重排序规则可以**确保**：**在读一个对象的final域之前，一定会先读包含这个final**
  **域的对象的引用。**在这个示例程序中，如果该引用不为null，那么引用对象的final域一定已经
  被A线程初始化过了。

### final域为引用类型

&emsp;&emsp;上面我们看到的final域是基础数据类型，如果final域是引用类型，将会有什么效果？请看下列示例代码。

```java
public class FinalReferenceExample {
	final int[] intArray; // final是引用类型
	static FinalReferenceExample obj;
	public FinalReferenceExample () { // 构造函数
		intArray = new int[1]; // 1
		intArray[0] = 1; // 2
	}
	public static void writerOne () { // 写线程A执行
		obj = new FinalReferenceExample (); // 3
	}
	public static void writerTwo () { // 写线程B执行
		obj.intArray[0] = 2; // 4
	}
	public static void reader () { // 读线程C执行
		if (obj != null) { // 5
	int temp1 = obj.intArray[0]; // 6
		}
	}
}
```

&emsp;&emsp;本例final域为一个引用类型，它引用一个int型的数组对象。对于引用类型，写final域的重
排序规则对编译器和处理器增加了如下约束：在构造函数内对一个final引用的对象的成员域
的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之
间不能重排序。
&emsp;&emsp;对上面的示例程序，假设首先线程A执行writerOne()方法，执行完后线程B执行
writerTwo()方法，执行完后线程C执行reader()方法。图3-31是一种可能的线程执行时序。
在图3-31中，1是对final域的写入，2是对这个final域引用的对象的成员域的写入，3是把被
构造的对象的引用赋值给某个引用变量。这里除了前面提到的1不能和3重排序外，2和3也不
能重排序。
&emsp;&emsp;JMM可以确保读线程C至少能看到写线程A在构造函数中对final引用对象的成员域的写
入。即C至少能看到数组下标0的值为1。而写线程B对数组元素的写入，读线程C可能看得到，
也可能看不到。JMM不保证线程B的写入对读线程C可见，因为写线程B和读线程C之间存在数
据竞争，此时的执行结果不可预知。
如果想要确保读线程C看到写线程B对数组元素的写入，写线程B和读线程C之间需要使
用同步原语（lock或volatile）来确保内存可见性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417222207504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)

## 结论

&emsp;&emsp;JMM对正确同步的多线程程序的内存一致性做了如下保证:

1. 对于单线程程序：单线程程序不会出现数据可见性问题，JMM保证程序的执行结果和顺序一致性模型的执行结果一致。
2. 对于正确同步的多线程程序：JMM保证正确同步的多线程程序的执行结果和顺序一致性模型的执行结果一致。JMM通过添加内存屏障线程编程重排序和处理器重排序来保证这一点。
3. 对于为正确同步的多线程程序：JMM提供最小的保证：线程执行时读取到的值要么时之前某个线程写入的，要么是默认值(0,null,false)

> 如果程序是正确同步的，程序的执行将具有顺序一致性（Sequentially Consistent）——即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同。这对于程序员来说是一个极强的保证。这里的同步是指广义上的同步，包括对常用同步原语（synchronized、volatile和final）的正确使用。
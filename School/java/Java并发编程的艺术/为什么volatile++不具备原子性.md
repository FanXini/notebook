作者：dm_vincent 
来源：CSDN 
原文：https://blog.csdn.net/dm_vincent/article/details/79604716 
版权声明：本文为博主原创文章，转载请附上博文链接！

## 问题

在讨论原子性操作时，我们经常会听到一个说法：任意单个volatile变量的读写具有原子性，但是volatile++这种操作除外。

所以问题就是：为什么volatile++不是原子性的？

## 答案

因为它实际上是三个操作组成的一个符合操作。

首先获取volatile变量的值
将该变量的值加1
将该volatile变量的值写会到对应的主存地址
一个很简单的例子：

如果两个线程在volatile读阶段都拿到的是a=1，那么后续在线程对应的CPU核心上进行自增当然都得到的是a=2，最后两个写操作不管怎么保证原子性，结果最终都是a=2。每个操作本身都没啥问题，但是合在一起，从整体上看就是一个线程不安全的操作：发生了两次自增操作，然而最终结果却不是3.



我的理解：后面，经过我努力，我的理解是：i++这个操作是非原子性的，假设它里面有三个操作，a : 读取i的值，b : 将i增加1，c : 写入主存。这样的话，当我们线程1执行完a操作后，切换到线程2，待线程2的i++操作执行完后，线程1的缓存行失效，但是因为在线程1中，a操作(读取i的值)已经执行过了，无需再去读取i的值，所以缓存行是否更新也就无关紧要了，待线程1执行完b,c操作后，自然会得到错误的答案。

## 分析

结合内存屏障这个概念对volatile的读写操作深入理解的话：

第一步：读
在第一步操作的指令后，会增加两个内存屏障：

在Volatile读操作后插入LoadLoad屏障，防止前面的Volatile读与后面的普通读重排序
在Volatile读操作后插入LoadStore屏障，防止前面的Volatile读与后面的普通写重排序
因此第一个指令和它后续的普通读写操作会被保证没有重排序来捣乱。通常是去内存中去读。

那么问题又来了，为什么通常去内存中读？

其实这个问题要说细的话可以很细，大概就两个关键点吧：

volatile的写操作的缓存失效机制
最后一个对volatile变量执行写操作的CPU，由于在它对应的缓存中保有最新的值，因此可以不用再去主存里面获取
具体看下面第三步的分析。

第二步：自增
这个步骤没什么特别的，就是在CPU自身的高速缓存(寄存器，L1-L3 Cache)中完成。不涉及到缓存和内存的交互。

第三步：写
volatile写算是一个重点。

根据JMM对于volatile变量类型的语义规范：volatile在编译之后，会在变量写操作时添加LOCK前缀指令。这个LOCK前缀指令在多核处理器的环境中，有这样的作用：

通知CPU将当前处理器缓存行的数据写回到系统主存中
该写回操作将使其他CPU缓存了该内存地址的数据无效
另外，内存屏障在volatile的写操作中起到了很大的作用，来保证上面两点能够实现：

在Volatile写操作前插入StoreStore屏障，防止前面其他写与本次Volatile写重排序
在Volatile写操作后插入StoreLoad屏障，防止本次的Volatile写与后面的读操作重排序
延伸
那么为了解决volatile++这类复合操作的原子性，有什么方案呢？其实方案也比较多的，这里提供两种典型的：

使用synchronized关键字
使用AtomicInteger/AtomicLong原子类型
synchronized关键字
synchronized是比较原始的同步手段。它本质上是一个独占的，可重入的锁。当一个线程尝试获取它的时候，可能会被阻塞住，所以高并发的场景下性能存在一些问题。

在某些场景下，使用synchronized关键字和volatile是等价的：

写入变量值时候不依赖变量的当前值，或者能够保证只有一个线程修改变量值。
写入的变量值不依赖其他变量的参与。
读取变量值时候不能因为其他原因进行加锁。
加锁可以同时保证可见性和原子性，而volatile只保证变量值的可见性。

AtomicInteger/AtomicLong
这类原子类型比锁更加轻巧，比如AtomicInteger/AtomicLong分别就代表了整型变量和长整型变量。

在它们的实现中，实际上分别使用的volatile int/volatile long保存了真正的值。因此，也是通过volatile来保证对于单个变量的读写原子性的。

在此基础之上，它们提供了原子性的自增自减操作。比如incrementAndGet方法，这类方法相对于synchronized的好处是：它们不会导致线程的挂起和重新调度，因为在其内部使用的是CAS非阻塞算法。

CAS是什么
所谓的CAS全程为CompareAndSet。直译过来就是比较并设置。这个操作需要接受三个参数：

内存位置
旧的预期值
新值
这个操作的做法就是看指定内存位置的值符不符合旧的预期值，如果符合的话就将它替换成新值。它对应的是处理器提供的一个原子性指令 - CMPXCHG。

比如AtomicLong的自增操作：

```java
public final long incrementAndGet() {
    for (;;) {
        long current = get(); // Step 1
        long next = current + 1; // Step 2
        if (compareAndSet(current, next)) // Step 3
            return next;
    }
}

public final boolean compareAndSet(long expect, long update) {
    return unsafe.compareAndSwapLong(this, valueOffset, expect, update);
}
```

我们考虑两个线程T1和T2，同时执行到了上述Step 1处，都拿到了current值为1。然后通过Step 2之后，current在两个线程中都被设置为2。

紧接着，来到Step 3。假设线程T1先执行，此时符合CompareAndSet的设置规则，因此内存位置对应的值被设置成2，线程T1设置成功。当线程T2执行的时候，由于它预期current为1，但是实际上已经变成了2，所以CompareAndSet执行不成功，进入到下一轮的for循环中，此时拿到最新的current值为2，如果没有其它线程感染的话，再次执行CompareAndSet的时候就能够通过，current值被更新为3。

所以不难发现，CAS的工作主要依赖于两点：

无限循环，需要消耗部分CPU性能
CPU原子指令CompareAndSet
虽然它需要耗费一定的CPU Cycle，但是相比锁而言还是有其优势，比如它能够避免线程阻塞引起的上下文切换和调度。这两类操作的量级明显是不一样的，CAS更轻量一些。

总结
我们说对于volatile变量的读/写操作是原子性的。因为从内存屏障的角度来看，对volatile变量的单纯读写操作确实没有任何疑问。

由于其中掺杂了一个自增的CPU内部操作，就造成这个复合操作不再保有原子性。

然后，讨论了如何保证volatile++这类操作的原子性，比如使用synchronized或者AtomicInteger/AtomicLong原子类。


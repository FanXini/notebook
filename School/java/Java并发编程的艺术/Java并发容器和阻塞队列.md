---
typora-copy-images-to: ..\..\..\img
---

# 并发容器和阻塞队列

## 1 . ConcurrentHashMap

[ConcurrentHashMap源码分析(1.7和1.8)](https://www.cnblogs.com/study-everyday/p/6430462.html#autoid-2-2-3)

JDK1.8的实现没有再使用segment分段锁，而是使用synchronized和CAS机制来实现。

如果没有遇到Hash冲突，那么直接使用CAS完成入队操作

如果遇到了HASH冲突，则使用synchronized锁住链表的头节点，进行入队操作。

## 2 .ConcurrentLinkQueue

[参考博客：ConcurrentLinkedQueue的实现原理和源码分析](https://blog.csdn.net/u013991521/article/details/53068549)

在并发编程中，有时候需要使用线程安全的队列。如果要实现一个线程安全的队列有两种方式：一种是使用阻塞算法，另一种是使用非阻塞算法。==使用阻塞算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现。非阻塞的实现方式则可以使用循环CAS的方式来实现。==

本节让我们一起来研究一下Doug Lea是如何使用非阻塞的方式来实现线程安全队列ConcurrentLinkedQueue的，相信从大师身上我们能学到不少并发编程的技巧。

### 2.1结构

![1558927820943](F:\Typora\img\1558927820943.png)

&emsp;&emsp;ConcurrentLinkedQueue由head和tail节点组成。每个节点(Node)由节点元素(item)和指向下个节点的next引用组成。默认情况下head节点存储的元素为空，tail节点==head节点。

### 2.2 入队列的过程

&emsp;&emsp;入队列是将入队节点添加到队列的尾部。

&emsp;&emsp;示例：在一个队列中插入4个节点，其插入示意图如下所示：

![1558935671054](F:\Typora\img\1558935671054.png)

- 添加元素1。队列更新head节点的next节点为元素1节点。又因为tail节点默认情况下等于
  head节点，所以它们的next节点都指向元素1节点。
- 添加元素2。队列首先设置元素1节点的next节点为元素2节点，然后更新tail节点指向元素
  2节点。
- 添加元素3，设置tail节点的next节点为元素3节点。
- 添加元素4，设置元素3的next节点为元素4节点，然后将tail节点指向元素4节点。

&emsp;&emsp;综上我们发现，tail节点并不是一直指向尾节点的，这样设计的原因是什么呢？是为了减少CAS的开销。

**入队源码分析：**

```java
public boolean offer(E e) {
        checkNotNull(e);
        final Node<E> newNode = new Node<E>(e);

        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
                //p此时指向的是尾节点，因为下一个节点为Null
            if (q == null) {
                //使用CAS插入入队节点
                if (p.casNext(null, newNode)) {
                    //如果p!=t，说明tail节点后面有两个节点了，需要将
                    //tail节点指向新的尾节点
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.
                    return true;
                }
                // Lost CAS race to another thread; re-read next
            }
            // 如果p节点等于p的next节点，则说明p节点和q节点都为空，表示队列刚初始化，所以返回head节点
            else if (p == q)
                p = (t != (t = tail)) ? t : head;
            // Check for tail updates after two hops.
           //p有next节点，表示p的next节点是尾节点，则需要重新更新p后将它指向next节点
           //当在指向next节点之前要先判断t节点是否还指向tail节点，如果不在指向，则说明有其他线程更改过tail节点，因此，需要重新获取新的节点进行入队操作
            else
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
```

tail节点并不一直指向尾节点的设计意图：

> 让tail节点永远作为列表的尾节点，这样实现代码量非常少，而且逻辑清晰和易懂。但是，这么做的缺点是每次都需要循环CAS更新tail节点。如果能减少CAS更新tail节点的次数，就能提高入队的效率。因此设计者在tail节点后面有超过1个节点后才更新tail.当然，代价就是每次插入时需要定位尾节点。但是这样仍然能提高效率，因为从本质上来看它通过增加对volatile变量的读操作来减少对volatile变量的写操作。而volatile写的开销要远远大于读的开销。所以入队效率会有所提升。

### 2.3 出队列

&emsp;&emsp;出队列就是从队列里返回一个节点元素，并情况该节点对元素的引用。示意图如下：

![1558937716914](F:\Typora\img\1558937716914.png)

&emsp;&emsp;从图中可知，并不是每次出队时都更新head节点。当head节点里有元素时，直接弹出head节点中元素，而不会更新head的节点。只有当head节点没有元素时，出队操作才会更新head节点。这样设计的原理和入队一样，提高出队效率。

**源码分析**

```java
public E poll() {
        restartFromHead:
        for (;;) {
            for (Node<E> h = head, p = h, q;;) {
                E item = p.item;
				//如果item不为空，则说明 p时待出队节点，使用CAS将p的元素置为null
                if (item != null && p.casItem(item, null)) {
                    //如果p不是head，说明此时的头节点距离head节点有两个节点
                    if (p != h) // hop two nodes at a time
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                }
                //队列为空
                else if ((q = p.next) == null) {
                    updateHead(h, p);
                    return null;
                }
                //结点出队失败，重新跳到restartFromHead来进行出队
                else if (p == q)
                    continue restartFromHead;
                //将p指向p.next
                else
                    p = q;
            }
        }
    }
```

更新head节点的方法$updateHead$

```java
 final void updateHead(Node<E> h, Node<E> p) {
        if (h != p && casHead(h, p))
            h.lazySetNext(h);
    }
```

## 3 .阻塞队列

&emsp;&emsp;阻塞队列是一个支持两个附加操作的队列。这两个附加支持阻塞的插入和移除方法。

1. 阻塞插入：当队列满时，队列会阻塞插入元素的线程，直到队列不满
2. 阻塞移除：当队列空时，队列会阻塞移除元素的线程，直到线程不空

这两个附加操作有4种不同的实现方式，用来满足不同的需求：

| 方法/处理方式 | 抛出异常  | 返回特殊值 | 一直阻塞 | 超时退出           |
| ------------- | --------- | ---------- | -------- | ------------------ |
| 插入方法      | add(e)    | offer(e)   | put(e)   | offer(e,time,unit) |
| 移除方法      | remove()  | poll()     | take()   | poll(time,unit)    |
| 检查方法      | element() | peek()     | 不可用   | 不可用             |

- 抛出异常：当队列满时，如果再往队列里插入元素，会抛出IllegalStateException(”Queue full“)异常。当队列空时，从队列里获取元素会抛出NoSuchElementException异常。
- 返回特殊值：当往队列插入元素时，会返回元素是否插入成功，成功返回true。如果是移除方法，则是从队列里取出一个元素，如果没有则返回null。
- 一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者
  线程，直到队列可用或者响应中断退出。当队列空时，如果消费者线程从队列里take元素，队
  列会阻塞住消费者线程，直到队列不为空。
- 超时退出：当阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生产者线程
  一段时间，如果超过了指定的时间，生产者线程就会退出。

&emsp;&emsp;这两个附加操作的4种处理方式不方便记忆，所以我找了一下这几个方法的规律。put和
take分别尾首含有字母t，offer和poll都含有字母o。

阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。

### 3.1 java种的阻塞队列

JDK 7提供了7个阻塞队列，如下。

- ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。

- LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。

- PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。

- DelayQueue：一个使用优先级队列实现的无界阻塞队列。

- SynchronousQueue：一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。

- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

  想比于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。

  1. tryTransfer:如果当前有消费者正在等待接收元素（消费者使用take()方法或带时间限制的poll()方法时），transfer方法可以把生产者传入的元素立刻transfer（传输）给消费者。如果没有消费者在等待接收元素，transfer方法会将元素存放在队列的tail节点，并等到该元素被消费者消费了才返回。
  2. tryTransfer方法
     tryTransfer方法是用来试探生产者传入的元素是否能直接传给消费者。如果没有消费者等
     待接收元素，则返回false。和transfer方法的区别是tryTransfer方法无论消费者是否接收，方法立即返回，而transfer方法是必须等到消费者消费了才返回。对于带有时间限制的tryTransfer（E e，long timeout，TimeUnit unit）方法，试图把生产者传入的元素直接传给消费者，但是如果没有消费者消费该元素则等待指定的时间再返回，如果超时还没消费元素，则返回false，如果在超时时间内消费了元素，则返回true。

- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

### 3.2 阻塞队列的实现原理

&emsp;&emsp;java实现的方式的使用<font color='red'>通知等待模式.</font>

&emsp;&emsp;通知等待模式的实现之前介绍过，有两种实现方式。

&emsp;&emsp;1. 使用对象自带的wait()和signal()方法

&emsp;&emsp;2. 使用Lock种的Condition对象。

&emsp;&emsp;查看源码可以知道，JDK使用的是Condition方式。

以ArrayBlockingQueue部分源码为例进行分析：

```java
 private final Condition notEmpty;

 private final Condition notFull;

//检查为空操作
private static void checkNotNull(Object v) {
        if (v == null)
            throw new NullPointerException();
    }

//阻塞插入方法
public void put(E e) throws InterruptedException {
        checkNotNull(e);
    	//使用可重入锁保证同步
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                //阻塞该线程，等待通知
                notFull.await();
            //入队
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
//阻塞移除方法
public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                //阻塞该线程，等待通知
                notEmpty.await();
            //出队
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
    	//通知notFull对象种等待队列的线程
        notFull.signal();
        return x;
    }

private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        //通知notEmpty对象种等待队列中的线程
        notEmpty.signal();
    }
//返回特殊值的插入方法
public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }

//返回特殊值的移除方法
public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }

public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                if (nanos <= 0)
                    return null;
                nanos = notEmpty.awaitNanos(nanos);
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```

当往队列里插入一个元素时，如果队列不可用，那么阻塞生产者主要通过LockSupport.park（this）来实现。

```java
public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

继续进入源码，发现调用setBlocker先保存一下将要阻塞的线程，然后调用unsafe.park阻塞
当前线程。

```java
 public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```


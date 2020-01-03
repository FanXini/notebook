[TOC]

# java中的锁

## 1. java加锁的方式

&emsp;&emsp;java中的锁用来保证数据的可见性，一个锁能能够防止多个线程同时访问共享资源。
&emsp;&emsp;Java提供两种加锁的方式：

&emsp;&emsp;1. 通过synchronized关键字实现锁功能。

&emsp;&emsp;2. 通过Lock接口(jdk 1.5之后支持)

### 1.1 两者的区别

&emsp;&emsp;1. synchronized可以隐式的实现获取和释放锁，Lock必须要在同步代码块前后显示的进行锁的获取和释放。

&emsp;&emsp;2. Lock接口提高了对锁的可操作性，支持可中断的获取锁以及循环获取锁等多种sychronized关键字不具备的特性。

&emsp;&emsp;例如：针对一个场景，手把手进行锁获取和释放，先获得锁A，然后再获取锁B，当锁B获得后，释放锁A同时获取锁C，当锁C获得后，再释放B同时获取锁D，以此类推。这种场景下，synchronized关键字就不那么容易实现了，而使用Lock却容易许多。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506163738548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)

## 2. 锁的使用方法：

### 2.1 sychonized

sychonized的使用方式之前介绍过，不再重复

### 2.2 Lock

```java
Lock lock=new ReentrantLock();//重入锁，锁的一种实现
lock.lock();
try{
    代码块...
}finally{
		lock.unlock();
}
```

**LOCK的API**

| 方法名称                                                     | 描述                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| void lock();                                                 | 调用该方法后线程尝试获取锁，当锁获得后，从该方法返回。否则线程阻塞。 |
| void lockInterruptibly() throws InterruptedException;        | 可中断的获取锁，和lock()的区别在于在获取的过程中能线程能响应中断。 |
| boolean tryLock();                                           | 尝试非阻塞的获取锁，调用该方法后立刻返回，如果能够获取则放回true，否则返回false。 |
| boolean tryLock(long time, TimeUnit unit) throws InterruptedException; | 超时的获取锁，当前线程在以下三种情况会返回:<br />1. 当前线程在超时时间内获得了锁。<br />2. 当前线程在超时时间内被中断。<br />3. 超时时间结束，返回false。 |
| void unlock();                                               | 释放锁                                                       |
| Condition newCondition();                                    | 获取等待通知组件，该组件和当前的锁绑定，当前线程只用获得了锁，才能调用该组件的wait()方法，而调后，当前线程会释放锁。<br />注：实际是对java通知等待模式的一种封装，在之前java线程介绍中，有解释过怎么调用对象的wait()和notify()实现等待通知机制。 |

&emsp;&emsp;这里先简单介绍一下Lock接口的API，随后的章节会详细介绍同步器AbstractQueuedSynchronizer以及常用Lock接口的实现ReentrantLock。Lock接口的实现基本都是通过聚合了一个同步器的子类来完成线程访问控制的。

## 3. 队列同步器

&emsp;&emsp;队列同步器AbstractQueuedSynchronizer(AQS)，是一个用来创建锁或者其他同步组件的基础框架，它使用了一个int 成员变量status来表示同步状态，通过内置的FIFO(先入先出队列)来完成资源获取线程的排队工作。

&emsp;&emsp;同步器的字类推荐被定义为自定义同步组件的内部静态类，同步器暴露了几个抽象方法让程序员实现，而加锁的过程中会使用到者几个关键的抽象方法，从而来管理同步状态，实现自定义效果的同步组件。同步器支持独占式的获取同步状态，也支持共享式的获取同步状态。

### 3.1 AQS的接口与示例

同步器通过三个方法来防卫或者修改同步状态

```
private volatile int state;
```

- getStatus() :获取同步状态

  ```java
  protected final int getState() {
          return state;
  }
  ```

- setStats(int newStatus):设置同步状态

  ```java
  protected final void setState(int newState) {
          state = newState;
      }
  ```

- compareAndSetStatus(int expece,int update):使用CAS设置当前状态，该方法能保证状态设置的原子性。

  ```java
   protected final boolean compareAndSetState(int expect, int update) {
          return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
      }
  ```

  同步器暴露给开发者的几个方法：

  1. > ```java
     > //独占式获取同步状态，实现该方法需要查询当前状态并判判断同步状态是否符合预期，然后再进行CAS设置同步状态
     > protected boolean tryAcquire(int arg) {
     >     throw new UnsupportedOperationException();
     > }
     > ```

  2. ```java
     //独占式释放同步状态，等待获取同步状态的线程将有机会获取同步状态
     protected boolean tryRelease(int arg) {
             throw new UnsupportedOperationException();
         }
     ```

  3. ```java
     //共享式获取同步状态，返回大于等于0的值，表示获取成功，反正，获取失败
     protected int tryAcquireShared(int arg) {
             throw new UnsupportedOperationException();
         }
     ```

  4. ```java
     //共享式释放同步状态
     protected boolean tryReleaseShared(int arg) {
             throw new UnsupportedOperationException();
         }
     ```

  5. ```java
     //当前同步器是否在独占模式下被线程占用，一般该方法表示是否被当前线程所独占
     protected boolean isHeldExclusively() {
             throw new UnsupportedOperationException();
         }
     ```

     AQS设计者采用了模板模式来实现自定义组件的开发，例如，重入锁的公平锁实现加锁lock()方法会调用acquire(int newStatus)

     ```java
     static final class FairSync extends Sync {
     
             final void lock() {
                 acquire(1);
             }
         ...
     }
     ```

     而acquire会调用tryAcquire(int arg)方法：

     ```java
     public final void acquire(int arg) {
             if (!tryAcquire(arg) &&
                 acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                 selfInterrupt();
         }
     ```

     程序员通过自己实现的tryAcquire()方法，从而实现自定义组件的加锁特性。其他的模板方法如下：

     ![1557109720122](F:\Typora\img\1557109720122.png)

&emsp;&emsp;同步器提供的模板方法基本上分为3类：==**独占式获取与释放同步状态、共享式获取与释放同步状态和查询同步队列中的等待线程情况**==。自定义同步组件将使用同步器提供的模板方法来实现自己的同步语义。

### 3.2使用AQS实现自定义组件独占锁

```java
package xin.spring.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

public class Mutex implements Lock {
	private static class syn extends AbstractQueuedSynchronizer{
		/**
		 * 
		 */
		private static final long serialVersionUID = 1L;
		//判断是否处于占用状态
		protected boolean isHeldExclusively(){
			return getState()==1;      //if the state==1, mean the lock being monopolized
		}
		protected boolean tryAcquire(int acquired){
			if(compareAndSetState(0, 1)){
                //设置当前线程为独占线程
				setExclusiveOwnerThread(Thread.currentThread());
				return true;
			}
			return false;
		}
		protected boolean tryRelease(int releases){
			if(getState()==0){
				throw new IllegalMonitorStateException("can not release");
			}
			setState(0);
            //设置独占线程为Null
			setExclusiveOwnerThread(null);
			return true;
		}
		
	}
	syn syn=new syn();
	@Override
	public void lock() {
		syn.acquire(1);
	}

	@Override
	public void lockInterruptibly() throws InterruptedException {
		syn.acquireInterruptibly(1);
	}

	@Override
	public boolean tryLock() {
		return syn.tryAcquire(1);
	}

	@Override
	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
		return syn.tryAcquireNanos(1, unit.toNanos(time));
	}

	@Override
	public void unlock() {
		syn.release(1);
		
	}

	@Override
	public Condition newCondition() {
		return null;
	}

}

```

&emsp;&emsp;上述示例中，独占锁Mutex是一个自定义同步组件，它在同一时刻只允许一个线程占有锁。Mutex中定义了一个静态内部类，该内部类继承了同步器并实现了独占式获取和释放同步状态。在tryAcquire(int acquires)方法中，如果经过CAS设置成功（同步状态设置为1），则代表获取了同步状态，而在tryRelease(int releases)方法中只是将同步状态重置为0。用户使Mutex时并不会直接和内部同步器的实现打交道，而是调用Mutex提供的方法，在Mutex的实现中，以获取锁的lock()方法为例，只需要在方法实现中调用同步器的模板方法acquire(int args)即可，当前线程调用该方法获取同步状态失败后会被加入到同步队列中等待，这样就大大降低了实现一个可靠自定义同步组件的门槛。

## 4 .队列同步器的实现分析

&emsp;&emsp;接下来将从实现角度分析同步器是如何完成线程同步的，主要包括：同步队列、独占式同步状态获取与释放、共享式同步状态获取与释放以及超时获取同步状态等同步器的核心数据结构与模板方法。

### 4.1 同步队列

&emsp;&emsp;同步队列是一个服从FIFO的链表队列，当线程获取锁失败后，将会被构造成一个节点Node并加入到同步队列中，同时会阻塞当前线程。当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。AQS维护头节点head和尾节点tail。

#### 4.1.1 Node

Node是AQS中的一个静态内部类，是同步队列和等待队列中公用的一个节点类，成员变量有：

1. 当前获取锁的线程：volatile Thread thread;

2. 后续节点：volatile Node next;

3. 前继节点：volatile Node prev;

4. 等待队列中的后继节点：Node nextWaiter;如果当前节点是共享的，那么这个字段是一个SHARED常量，也就是说节点类型(独占或者共享)和等待队列中的后继节点共用同一个字段。

5. 等待状态：volatile int waitStatus;

   分别有：

   ```java
   //由于在同步队列中等待的线程等待超时或者被中断，需要从同步队列中取消等待，节点进入该状体将不会变化
   static final int CANCELLED =  1;
   //后继节点的线程处于等待状态，而当前节点的线程如果释放了锁或者被取消后需要通知后续节点，使后续节点得以运行。
    /** waitStatus value to indicate successor's thread needs unparking */
   static final int SIGNAL    = -1;
   //当前节点在等待队列中，节点线程等待在condition对象上，当其他线程调用了condition对象的signal方法后，节点会从等待队列转移到同步队列中尝试获取锁
   static final int CONDITION = -2;
   //表示下一次共享式同步状态会被无条件的传播下去
   static final int PROPAGATE = -3;
   ```

6. ```java
   ///** Marker to indicate a node is waiting in shared mode */
   //SHREAD常量
   static final Node SHARED = new Node();
   ```

7. ```java
   /** Marker to indicate a node is waiting in exclusive mode */
   static final Node EXCLUSIVE = null;
   ```

#### 4.1.2队列结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506163915575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;当线程获取锁失败后，会被加入到同步队列中来，为了保证线程安全，因此同步器提供了一个基于CAS的设置尾节点的方法来compareAndSetTail(Node expect,Node update)；

### 4.2 独占式同步状态获取与释放

#### 4.2.1 获取

源码分析：

&emsp;&emsp;通过调用同步器的acquire(int arg)方法可以获取同步状态，该方法对中断不敏感，也就是由于线程获取同步状态失败后进入同步队列中，后续对线程进行中断操作时，线程不会从同步队列中移出。

```java
//获取锁
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))//失败则添加到同步队列，并调用acquireQueued实现自旋获取
        selfInterrupt();
}
```

**addWaiter(Node node)** 

```java
private Node addWaiter(Node mode) {
    	//构建节点
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
    	//尝试一次快速的加入队列，并修改尾节点，允许失败，失败后调用end方法
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            //调用CAS方法将当前节点设置为尾节点
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
```

**end(Node node)**

```java
 private Node enq(final Node node) {
     //自旋，无限尝试，直到加入队列成功
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

&emsp;&emsp;最后调用acquireQueued(Node node,int arg)方法，使得该节点以“死循环”的方式获取同步状态。如果获取不到则阻塞节点中的线程，而被阻塞线程的唤醒主要依靠前驱节点的出队或阻塞线程被中断来实现。

**acquireQueued(Node node ,int newStatus)**

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                //只有前继节点是head节点时才会尝试获取锁
                if (p == head && tryAcquire(arg)) {
                    //设置首节点，因为已经获取到了锁，所以不需要使用CAS
                    setHead(node);
                    p.next = null; // help GC,方便垃圾回收
                    failed = false;
                    return interrupted;
                }
                // 防止节点的前节点中断或者删除了，导致永远永远无法被唤醒，这一部的作用就是如果前节点的状态是-1,则删除前节点，并更新前节点，直到前节点的状态<0
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt()) //调用park方法进行阻塞
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

&emsp;&emsp;在**acquireQueued(final Node node,int arg)**方法中，当前线程在“死循环”中尝试获取同步状态，而只有前驱节点是头节点才能够尝试获取同步状态，这是为什么？原因有两个，如下。
第一，头节点是成功获取到同步状态的节点，而头节点的线程释放了同步状态之后，将会
唤醒其后继节点，后继节点的线程被唤醒后需要检查自己的前驱节点是否是头节点。
第二，维护同步队列的FIFO原则。该方法中，节点自旋获取同步状态的行为如图5-4所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506163958652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;在图5-4中，由于非首节点线程前驱节点出队或者被中断而从等待状态返回，随后检查自
己的前驱是否是头节点，如果是则尝试获取同步状态。可以看到节点和节点之间在循环检查
的过程中基本不相互通信，而是简单地判断自己的前驱是否为头节点，这样就使得节点的释
放规则符合FIFO，并且也便于对过早通知的处理（过早通知是指前驱节点不是头节点的线程
由于中断而被唤醒）。独占式同步状态获取流程，也就是acquire(int arg)方法调用流程，如下图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506164132203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)

#### 4.2.2  释放

通过调用同步器的release方法就能释放同步状态。

源码分析：

```java
public final boolean release(int arg) {
    //调用tryRease（arg）方法
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

释放后，会调用unparkSuccessor(Node head)方法，唤醒后继节点。

```java
private void unparkSuccessor(Node node) {
        
        int ws = node.waitStatus;
    //如果状态为负（即，可能需要signal），则尝试清除预期信令。 如果失败或者等待线程改变了状态，则可以。
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        Node s = node.next;
     //需要唤醒在后续节点中，它通常只是下一个节点。但是，如果下一个节点为空或者状态是CANCELLED(1)，则从tail向后遍历以找到实际的非CANCELLED后继。
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

### 4.3 共享式同步状态获取与释放

共享式状态与独占式的区别是是否允许多个线程同时获取锁。

#### 4.3.1 获取

通过调用AQS的acquireShared方法获取同步状态

```java
public final void acquireShared(int arg) {
    //小于0表示获取共享锁失败
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

**doAcquireShared**

```java
private void doAcquireShared(int arg) {
        //添加到同步队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            //自旋获取锁
            for (;;) {
                final Node p = node.predecessor();
                //如果前继节点是head节点
                if (p == head) {
                    //尝试获取共享锁，如果返回值>0，则表示获取成功
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### 4.3.2 释放

**调用tryReleaseShared进行共享锁的释放**

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

该方法在释放同步状态之后，将会唤醒后续处于等待状态的节点。对于能够支持多个线
程同时访问的并发组件（比如Semaphore），==它和独占式主要区别在于tryReleaseShared(int arg)方法必须确保同步状态（或者资源数）线程安全释放==，一般是通过循环和CAS来保证的，因为释放同步状态的操作会同时来自多个线程。

### 4.4 独占式超时获取同步状态

&emsp;&emsp;在分析该方法的实现前，先介绍一下响应中断的同步状态获取过程。**在Java 5之前，当一个线程获取不到锁而被阻塞在synchronized之外时，对该线程进行中断操作，此时该线程的中断标志位会被修改，但线程依旧会阻塞在synchronized上，等待着获取锁。**在Java 5中，同步器提供了acquireInterruptibly(int arg)方法，这个方法在等待获取同步状态时，如果当前线程被中断，会立刻返回，并抛出InterruptedException。

源码分析：

```java
public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (!tryAcquire(arg))
            // 获取失败，则调用doAcquireInterruptibly（int arg）方法
            doAcquireInterruptibly(arg)
    }
```

**doAcquireInterruptibly（int arg）**

```java
private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    //如果线程被中断，抛出异常
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

超时获取同步状态过程可以被视作响应中断获取同步状态过程的**“增强版”，**
doAcquireNanos(int arg,long nanosTimeout)方法在支持响应中断的基础上，增加了超时获取的特性。针对超时获取，主要需要计算出需要睡眠的时间间隔nanosTimeout，为了防止过早通知，nanosTimeout计算公式为：nanosTimeout-=now-lastTime，其中now为当前唤醒时间，lastTime为上次唤醒时间，如果nanosTimeout大于0则表示超时时间未到，需要继续睡眠nanosTimeout纳秒，反之，表示已经超时。

调用顺序：**tryAcquireNanos——>doAcquireNanos**

```java
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) ||
        doAcquireNanos(arg, nanosTimeout);
}
```

**doAcquireNanos(arg, nanosTimeout);**

```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (nanosTimeout <= 0L)
            return false;
        final long deadline = System.nanoTime() + nanosTimeout;
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
                nanosTimeout = deadline - System.nanoTime();
                if (nanosTimeout <= 0L)
                    return false;
                if (shouldParkAfterFailedAcquire(p, node) &&
                    nanosTimeout > spinForTimeoutThreshold)
                    LockSupport.parkNanos(this, nanosTimeout);
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

作者使用了一个nanosTimeout表示还要等待的时间，如果nanosTimeout<0后还没有获取到锁，则放回false，如果nanosTimeout > spinForTimeoutThreshold，使用超时park(方法)，当nanosTimeout <spinForTimeoutThreshold后，如果再使用park方法，或导致时间不精确的问题，因此会使用快速自旋的方法来计时。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506164253840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)

## 5 LockSupport工具

回顾第4章，当需要阻塞或唤醒一个线程的时候，都会使用LockSupport工具类来完成相应
工作。LockSupport定义了一组的公共静态方法，这些方法提供了最基本的线程阻塞和唤醒功能，而LockSupport也成为构建同步组件的基础工具。LockSupport定义了一组以park开头的方法用来阻塞当前线程，以及unpark(Thread thread)方法来唤醒一个被阻塞的线程。Park有停车的意思，假设线程为车辆，那么park方法代表着停车，而unpark方法则是指车辆启动离开，这些方法以及描述如表5-10所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506164335102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
在Java 6中，LockSupport增加了park(Object blocker)、parkNanos(Object blocker,long nanos)和parkUntil(Object blocker,long deadline)3个方法，用于实现阻塞当前线程的功能，其中参数blocker是用来标识当前线程在等待的对象（以下称为阻塞对象），该对象主要用于问题排查和系统监控。

## 6 Condition接口

再之前讲述java线程 [Java并发编程基础之线程详解](https://blog.csdn.net/fanxin_i/article/details/89452133)的5.3讲到过等待/通知机制。而Condition是另外一种实现等待/通知机制的方法。当线程调用到这些方法时，需要先获取到Condition对象关联的锁。Condition对象是由Lock对象（调用Lock对象的newCondition()方法创
建出来的，换句话说，Condition是依赖Lock对象的。其部分方法如5-13所示.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506164515997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
使用示例，使用condition构造一个有界队列

```java
public class BoundedQueue<T> {
    private Object[] items;
    // 添加的下标，删除的下标和数组当前数量
    private int addIndex, removeIndex, count;
    private Lock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();
    public BoundedQueue(int size) {
        items = new Object[size];
    }
    // 添加一个元素，如果数组满，则添加线程进入等待状态，直到有"空位"
    public void add(T t) throws InterruptedException {
        //先加锁
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();
            items[addIndex] = t;
            if (++addIndex == items.length)
                addIndex = 0;
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    // 由头部删除一个元素，如果数组空，则删除线程进入等待状态，直到有新添加元素
    @SuppressWarnings("unchecked")
    public T remove() throws InterruptedException {
        //加锁
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            Object x = items[removeIndex];
            if (++removeIndex == items.length)
                removeIndex = 0;
            --count;
            notFull.signal();
            return (T) x;
        } finally {
            lock.unlock();
        }
    }
}
```

### 6.1 Condition的实现分析

Condition是AQS的一个内部类，它主要依赖一个等待队列来实现。

#### 6.1.1.等待队列

&emsp;&emsp;等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是
在Condition对象上等待的线程，如果一个线程调用了Condition.await()方法，那么该线程将会释放锁、构造成节点加入等待队列并进入等待状态。事实上，节点的定义复用了同步器中节点的定义，也就是说，同步队列和等待队列中节点类型都是同步器的静态内部类
AbstractQueuedSynchronizer.Node。一个Condition包含一个等待队列，Condition拥有首节点（firstWaiter）和尾节点（lastWaiter）。当前线程调用Condition.await()方法，将会以当前线程构造节点，并将节点从尾部加入等待队列，等待队列的基本结构如图5-9所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506164539305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
&emsp;&emsp;如图所示，Condition拥有首尾节点的引用，而新增节点只需要将原有的尾节点**nextWaiter**指向它，并且更新尾节点即可。上述节点引用更新的过程并没有使用CAS保证，原因在于调用await()方法的线程必定是获取了锁的线程，也就是说该过程是由锁来保证线程安全的。

&emsp;&emsp;在Object的监视器模型上，一个对象拥有一个同步队列和等待队列，而并发包中的
Lock（更确切地说是同步器）拥有一个同步队列和多个等待队列，其对应关系如图5-10所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506164601698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zhbnhpbl9p,size_16,color_FFFFFF,t_70)
### 6.2 等待

&emsp;&emsp;调用Condition的await()方法（或者以await开头的方法），会使当前线程进入等待队列并释放锁，同时线程状态变为等待状态。当从await()方法返回时，当前线程一定获取Condition相关联的锁。如果从队列（同步队列和等待队列）的角度看await()方法，当调用await()方法时，相当于同步队列的首节点（获取了锁的节点）移动到Condition的等待队列中。

源码分析:

```java
public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
    		//添加到等待队列中去
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
    		//如果该节点不在同步队列中，则阻塞
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

isOnSyncQueue(node)方法用来判断是节点是否在同步队列，只有condition被其他线程调用了signal()或signalAll()方法，节点才有可能从等待队列移到同步队列。

**addConditionWaiter()**

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    //如果尾节点是空或者状态为cancelled,则删除，找到正确的尾节点
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    //如果队列为空，则使用firstWaiter指向新节点
    if (t == null)
        firstWaiter = node;
    //否则尾节点的nextWaiter指向新结点
    else
        t.nextWaiter = node;
    //把新结点置为尾节点
    lastWaiter = node;
    return node;
}
```

#### 6.3 通知

&emsp;&emsp;调用Condition的signal()方法，将会唤醒在等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移到同步队列中。

源码分析:

```java
public final void signal() {
    //判断当前线程是否获取到了锁
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

 调用该方法的前置条件是当前线程必须获取了锁，可以看到signal()方法进行了
isHeldExclusively()检查，也就是当前线程必须是获取了锁的线程。接着获取等待队列的首节
点，将其移动到同步队列并使用LockSupport唤醒节点中的线程。

**doSignal(first)**

```java
private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
```

**transferForSignal(Node node)**

```java
final boolean transferForSignal(Node node) {
    /*
     * If cannot change waitStatus, the node has been cancelled.
     */
    //吧状态从CONDITION改为0
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
    //使用end方法将节点添加到同步队列中
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        //唤醒线程
        LockSupport.unpark(node.thread);
    return true;
}
```

&emsp;&emsp;通过调用同步器的enq(Node node)方法，等待队列中的头节点线程安全地移动到同步队列。当节点移动到同步队列后，当前线程再使用LockSupport唤醒该节点的线程。
被唤醒后的线程，将从await()方法中的while循环中退出（isOnSyncQueue(Node node)方法返回true，节点已经在同步队列中），进而调用同步器的acquireQueued()方法加入到获取同步状态的竞争中。
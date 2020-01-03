---
typora-copy-images-to: ..\..\..\img
---

Java中的原子操作包括：



1）除long和double之外的基本类型的赋值操作

2）所有引用reference的赋值操作

3）java.concurrent.Atomic.* 包中所有类的一切操作。



但是java对long和double的赋值操作是非原子操作！！long和double占用的字节数都是8，也就是64bits。在32位操作系统上对64位的数据的读写要分两步完成，每一步取32位数据。这样对double和long的赋值操作就会有问题：如果有两个线程同时写一个变量内存，一个进程写低32位，而另一个写高32位，这样将导致获取的64位数据是失效的数据。因此需要使用volatile关键字来防止此类现象。volatile本身不保证获取和设置操作的原子性，仅仅保持修改的可见性。但是java的内存模型保证声明为volatile的long和double变量的get和set操作是原子的。（from http://www.iteye.com/topic/213794）



举个例子来说：（example is from http://stackoverflow.com/questions/17481153/long-and-double-assignments-are-not-atomic-how-does-it-matter）

```java
public class UnatomicLongDemo implements Runnable {

    private static long test = 0;

    private final long val;

    public UnatomicLongDemo(long val) {
        this.val = val;
    }

    @Override
    public void run() {
        while (!Thread.interrupted()) {
            test = val;//两个线程同时断写test变量，如果test变量的读写操作是原子性的，那么test只能是-1或者0
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new UnatomicLongDemo(-1));
        Thread t2 = new Thread(new UnatomicLongDemo(0));

        System.out.println(Long.toBinaryString(-1));
        System.out.println(pad(Long.toBinaryString(0), 64));

        t1.start();
        t2.start();

        long switchVal;
        while ((switchVal = test) == -1 || switchVal == 0) {
            //如果test、switchVal的操作是原子性的,那么就应该是死循环，否则就会跳出该循环
            System.out.println("testing...");
        }

        System.out.println(pad(Long.toBinaryString(switchVal), 64));
        System.out.println(switchVal);

        t1.interrupt();
        t2.interrupt();
    }

    //将0补齐到固定长度
   private static String pad(String s, int targetLength) {
        int n = targetLength - s.length();
        for (int x = 0; x < n; x++) {
            s = "0" + s;
        }
        return s;
    }

}

```

测试结果为：

![1542682127566](..\..\..\img\1542682127566.png)

通过这个例子可以看出

在32位JVM上，对`long`型变量`test`的读取或者对`long`型变量`switchVal`的写入不是原子性的。

非原子性的读、写只是造成long、double类型变量的高、低32位数据不一致

这是由于在32位JVM中对64位的数据的读、写分两步，每一步读或者写32位的数据，这样就会造成两个线程对同一个变量的读写出现一个线程写高32位、另一 个线程写入低32位数据。这样此变量的数据就出现不一致的情况。这时候`volatile`关键字可以防止这种现象发生，因为java的内存模型保证了`valotile`修饰的`long`、`double`变量的读写是原子性的。

作者：Spground 
来源：CSDN 
原文：https://blog.csdn.net/u014532901/article/details/78580867 
版权声明：本文为博主原创文章，转载请附上博文链接！
[Java List的深度克隆](https://www.cnblogs.com/AkazaAkari/p/5940194.html)

# 关于java List的深度克隆

List是java容器中最常用的顺序存储数据结构之一。有些时候我们将一组数据取出放到一个List对象中，但是可能会很多处程序要读取他或者是修改他。尤其是并发处理的话，显然有的时候有一组数据有的时候是不够用的。这个时候我们通常会复制出一个甚至多个克隆List来执行更多的操作。

常见的List的克隆方式有很多，下面我们来列举几种常见的List复制的方式：

（首先还是构造一个简单的原始list对象）

```java
List<String> listString0 = new ArrayList<>();
listString0.add("xxx1");
listString0.add("xxx2");
listString0.add("xxx3");
```

 

克隆方法1：利用原list作为参数直接构造方法生成。

```java
List<String> listString1 = new ArrayList<>(listString0);
```

 

克隆方法2：手动遍历将原listString0中的元素全部添加到复制表中。

```java
List<String> listString2 = new ArrayList<>();
for(int i = 0, l = listString0.size(); i < l; i++)
    listString2.add(listString0.get(i));
```

 

克隆方法3：调用Collections的静态工具方法 Collections.copy

List<String> listString3 = new ArrayList<>(Arrays.asList(new String[listString0.size()]));
Collections.copy(listString3,listString0);

注：这种方法本身就有些鬼畜了，因为他需要保证copy双方的List的size值满足一定的条件，而List的size的计算方式......

 

克隆方法4：使用System.arraycopy方法进行复制

```java
String[] strs = new String[listString0.size()];
System.arraycopy(listString0.toArray(), 0, strs, 0, listString0.size());
List<String> listString4 = Arrays.asList(strs);
```

注：这种方法就更有些鬼畜了，因为严格来说System.arraycopy是用来copy系统自带的array的，转来转去的效率不多提。

 

好，先列举这么多，我们来测试一下我们想要的复制有没有达到效果：
我们试着改变listString0的某一个元素的值，如果其他列表中的值没有受到影响那么就是复制成功了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
listString0.set(0, "rock");
listString1.set(2, "deria");
        
for(int i = 0, l = listString0.size(); i < l; i++)
{
    System.out.println("listString0的第"+i+"个值为："+listString0.get(i));
    System.out.println("listString1的第"+i+"个值为："+listString1.get(i));
    System.out.println("listString2的第"+i+"个值为："+listString2.get(i));
    System.out.println("listString3的第"+i+"个值为："+listString3.get(i));
    System.out.println("listString4的第"+i+"个值为："+listString4.get(i));
    System.out.println("------------------------------------------------");
}        
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

输出结果：
listString0的第0个值为：rock
listString1的第0个值为：xxx1
listString2的第0个值为：xxx1
listString3的第0个值为：xxx1
listString4的第0个值为：xxx1
\------------------------------------------------
listString0的第1个值为：xxx2
listString1的第1个值为：xxx2
listString2的第1个值为：xxx2
listString3的第1个值为：xxx2
listString4的第1个值为：xxx2
\------------------------------------------------
listString0的第2个值为：xxx3
listString1的第2个值为：deria
listString2的第2个值为：xxx3
listString3的第2个值为：xxx3
listString4的第2个值为：xxx3
\------------------------------------------------

 

看来这几个方法都实现了list的复制，达到了我们想要的效果。修改本体并没有影响到复制体。但是故事远远没有结束。我们来试试下边的例子：
我们创建一个Pojo类：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
import java.io.Serializable;

public class PojoStr implements Serializable
{
    /**
     * 
     */
    private static final long serialVersionUID = 4394836462951175834L;
    
    private String str = "";

    public String getStr()
    {
        return str;
    }

    public void setStr(String str)
    {
        this.str = str;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个Pojo类只存储了一个字符串，同样可以模拟我们之前的几种复制方法，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class TestPojo
{
    public static void main(String[] args)
    {
        List<PojoStr> listPojoStr0 = new ArrayList<>();
        PojoStr p1 = new PojoStr();
        p1.setStr("xxx1");
        listPojoStr0.add(p1);
        PojoStr p2 = new PojoStr();
        p2.setStr("xxx2");
        listPojoStr0.add(p2);
        PojoStr p3 = new PojoStr();
        p3.setStr("xxx3");
        listPojoStr0.add(p3);

        List<PojoStr> listPojoStr1 = new ArrayList<>(listPojoStr0);

        List<PojoStr> listPojoStr2 = new ArrayList<>();
        for(int i = 0, l = listPojoStr0.size(); i < l; i++)
            listPojoStr2.add(listPojoStr0.get(i));

        List<PojoStr> listPojoStr3 = new ArrayList<>(Arrays.asList(new PojoStr[listPojoStr0.size()]));
        Collections.copy(listPojoStr3,listPojoStr0);

        PojoStr[] strs = new PojoStr[listPojoStr0.size()];
        System.arraycopy(listPojoStr0.toArray(), 0, strs, 0, listPojoStr0.size());
        List<PojoStr> listPojoStr4 = Arrays.asList(strs);
        
        listPojoStr0.get(0).setStr("rock");
        
        for(int i = 0, l = listPojoStr0.size(); i < l; i++)
        {
            System.out.println("listPojoStr0的第"+i+"个值为："+listPojoStr0.get(i).getStr());
            System.out.println("listPojoStr1的第"+i+"个值为："+listPojoStr1.get(i).getStr());
            System.out.println("listPojoStr2的第"+i+"个值为："+listPojoStr2.get(i).getStr());
            System.out.println("listPojoStr3的第"+i+"个值为："+listPojoStr3.get(i).getStr());
            System.out.println("listPojoStr4的第"+i+"个值为："+listPojoStr4.get(i).getStr());
            System.out.println("------------------------------------------------");
        }        
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行后我们居然惊讶的发现：

listPojoStr0的第0个值为：rock
listPojoStr1的第0个值为：rock
listPojoStr2的第0个值为：rock
listPojoStr3的第0个值为：rock
listPojoStr4的第0个值为：rock
\------------------------------------------------
由于本体表的某个数据的修改，导致后续的克隆表的数据全被修改了。而且四种方法全部阵亡......也就是说对于自定义POJO类而言上述的四种方法都未能实现对元素对象自身的复制。复制的只是对象的引用或是说地址。

我们知道List自身是一个对象，**他在存储类类型的时候，只负责存储地址。而存储基本类型的时候，存储的就是实实在在的值**。其实上边的程序也说明了这点，因为我们修改PojoStr-List的时候直接的修改了元素本身而不是使用的ArrayList的set(index,object)方法。所以纵然你有千千万万个List，元素还是那么几个。无论是重新构造，Collections的复制方法，System的复制方法，还是手动去遍历，结果都一样，这些方法都只改变了ArrayList对象的本身，简单的添加了几个指向老元素的地址。而没做深层次的复制。（压根没有没有 new新对象 的操作出现。）
当然有的时候我们确实需要将这些元素也都复制下来而不是只是用原来的老元素。然而很难在List层实现这个问题。毕竟依照java的语言风格，也很少去直接操作这些埋在堆内存中的数据，所有的操作都去针对能找到他们的地址了。地址没了自身还会被GC干掉。所以只好一点点的去遍历去用new创建新的对象并赋予原来的值。不过据说国外的某位大神可能觉得上述的做法略微鬼畜，所以巧用序列化对象让这些数据在IO流中360度跑了一圈，居然还真的成功复制了。其实把对象序列化到流中，java语言实在是妥协了，毕竟这把你不能再把地址给我扔进去吧？再说了io流是要和别的系统交互的，你发给别人一个地址让别人去哪个堆里找？所以不用多提肯定要新开辟堆内存的。
方法如下：（注：前提是 T如果是Pojo类的话，必须实现序列化接口，这是对象进入IO流的基本要求）。

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@SuppressWarnings("unchecked")
public static <T> List<T> deepCopyList(List<T> src)
{
    List<T> dest = null;
    try
    {
        ByteArrayOutputStream byteOut = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(byteOut);
        out.writeObject(src);
        ByteArrayInputStream byteIn = new ByteArrayInputStream(byteOut.toByteArray());
        ObjectInputStream in = new ObjectInputStream(byteIn);
        dest = (List<T>) in.readObject();
    }
    catch (IOException e)
    {

    }
    catch (ClassNotFoundException e)
    {

    }
    return dest;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

来试一下：

```java
List<PojoStr> listPojoStr5 = CopyUtils.deepCopyList(listPojoStr0);
listPojoStr0.get(0).setStr("rock");
```

输出结果
listPojoStr0的第0个值为：rock
listPojoStr1的第0个值为：rock
listPojoStr2的第0个值为：rock
listPojoStr3的第0个值为：rock
listPojoStr4的第0个值为：rock
listPojoStr5的第0个值为：xxx1

 

OK 成功
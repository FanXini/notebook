```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        int[] data = {4, 5, 3, 6, 2, 5, 1};
*// int[] 转 List<Integer>*
        List<Integer> list1 = Arrays.stream(data).boxed().collect(Collectors.toList());
*// Arrays.stream(arr) 可以替换成IntStream.of(arr)。*
*// 1.使用Arrays.stream将int[]转换成IntStream。*
*// 2.使用IntStream中的boxed()装箱。将IntStream转换成Stream<Integer>。*
*// 3.使用Stream的collect()，将Stream<T>转换成List<T>，因此正是List<Integer>。*
*// int[] 转 Integer[]*
        Integer[] integers1 = Arrays.stream(data).boxed().toArray(Integer[]::new);
*// 前两步同上，此时是Stream<Integer>。*
*// 然后使用Stream的toArray，传入IntFunction<A[]> generator。*
*// 这样就可以返回Integer数组。*
*// 不然默认是Object[]。*
*// List<Integer> 转 Integer[]*
        Integer[] integers2 = list1.toArray(new Integer[0]);
*//  调用toArray。传入参数T[] a。这种用法是目前推荐的。*
*// List<String>转String[]也同理。*
*// List<Integer> 转 int[]*
        int[] arr1 = list1.stream().mapToInt(Integer::valueOf).toArray();
*// 想要转换成int[]类型，就得先转成IntStream。*
*// 这里就通过mapToInt()把Stream<Integer>调用Integer::valueOf来转成IntStream*
*// 而IntStream中默认toArray()转成int[]。*
*// Integer[] 转 int[]*
        int[] arr2 = Arrays.stream(integers1).mapToInt(Integer::valueOf).toArray();
*// 思路同上。先将Integer[]转成Stream<Integer>，再转成IntStream。*
*// Integer[] 转 List<Integer>*
        List<Integer> list2 = Arrays.asList(integers1);
*// 最简单的方式。String[]转List<String>也同理。*
*// 同理*
        String[] strings1 = {"a", "b", "c"};
*// String[] 转 List<String>*
        List<String> list3 = Arrays.asList(strings1);
*// List<String> 转 String[]*
        String[] strings2 = list3.toArray(new String[0]);
    }
}
```

## String[] 和List的互转

### List to Array

　　List 提供了toArray的接口，所以可以直接调用转为object型数组

```
List<String> list = new ArrayList<String>();
Object[] array=list.toArray();
```

　　上述方法存在强制转换时会抛异常，下面此种方式更推荐：可以**指定类型**

```
String[] array=list.toArray(new String[list.size()]);
```

### Array to List

　　最简单的方法似乎是这样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
String[] array = {"java", "c"};
List<String> list = Arrays.asList(array);
//但该方法存在一定的弊端，返回的list是Arrays里面的一个静态内部类，该类并未实现add,remove方法，因此在使用时存在局限性

public static <T> List<T> asList(T... a) {
//  注意该ArrayList并非java.util.ArrayList
//  java.util.Arrays.ArrayList.ArrayList<T>(T[])
    return new ArrayList<>(a);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

解决方案：

　　1、运用ArrayList的构造方法是目前来说最完美的作法，代码简洁，效率高：**List<String> list = new ArrayList<String>(Arrays.asList(array));**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
List<String> list = new ArrayList<String>(Arrays.asList(array));

//ArrayList构造方法源码
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    size = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　2、运用Collections的addAll方法也也是不错的解决办法

```java
List<String> list = new ArrayList<String>(array.length);
Collections.addAll(list, array);
```

### Array or List 分隔

　　其实自己实现一个分隔list或者数组的方法也并不复杂，但强大的第三方库自然提供的有此类似的功能

```java
// org.apache.commons.lang3.StringUtils.join(Iterable<?>, String)
StringUtils.join(list, ",")
// org.apache.commons.lang3.StringUtils.join(Object[], String)
StringUtils.join(array, ",")
```
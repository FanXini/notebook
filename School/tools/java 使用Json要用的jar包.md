# java转换json需要导入的jar包，org apache commons lang exception NestableRuntimeException

缺少相应jar包都会有异常，根据异常找jar包导入......  

这里我说下lang包，因为这个包我找了好半天：

 

我用的是： commons-lang3-3.1.jar  出现异常：

java.lang.NoClassDefFoundError: org/apache/commons/lang/exception/NestableRuntimeException

可以看出是因为缺少jar包，但是很明显我已经导入了，为什么还会报这个错呢？

 

找了半天问题，终于明白了，看下图：

好多人留言说没图，没注意搞丢了，也懒的找了。  这个图是commons-lang3-3.1.jar 包的目录，懒的截了

 

在看下 commons-lang-2.4.jar 这个版本的jar包下面目录：

如下图：

  这个图是commons-lang-2.4.jar  包的目录，懒的截了

 

针对lang包，新版本居然包名都改了，这个真的没想到，暂时就看了这两个版本，其它版本是否有同样的问题，以后注意下就好了。。。。发个博客记录一下！

报错了差哪个包，对应去找，感觉包导入了还报错，打开包的目录看看有不有那个类，没有就换别的版本看看，lang3与lang目录有改变，所以会有错误。

以下是网上搜的，不想看可以忽略：

如果有类似错误可以参考，版本不同，记得看下里面包名是否和报错信息对应的上。

 ```java
commons-beanutils-1.8.0.jar不加这个包 

java.lang.NoClassDefFoundError: org/apache/commons/beanutils/DynaBean 

commons-collections.jar 不加这个包 

java.lang.NoClassDefFoundError: org/apache/commons/collections/map/ListOrderedMap

commons-lang-2.4.jar不加这个包 

java.lang.NoClassDefFoundError: org/apache/commons/lang/exception/NestableRuntimeException

commons-logging-1.1.1.jar不加这个包 

java.lang.NoClassDefFoundError: org/apache/commons/logging/LogFactory 

ezmorph-1.0.4.jar不加这个包 

java.lang.NoClassDefFoundError: net/sf/ezmorph/Morpher 

json-lib-2.3-jdk15.jar不加这个包 

java.lang.NoClassDefFoundError: net/sf/json/JSONObject 

相应jar包可到网上下载，也可以用下面提供的！ 
 ```

实例：

```java 
import java.util.ArrayList;

import java.util.List;

import net.sf.json.JSONArray;

public class JsonTest {

	/**
	 * 
	 * \* @param args
	 * 
	 */

	public static void main(String[] args) throws Exception {

		boolean[] boolArray = new boolean[] { true, false, true };

		JSONArray jsonArray = JSONArray.fromObject(boolArray);

		System.out.println(jsonArray);

		List list = new ArrayList();

		list.add("first");

		list.add("second");

		JSONArray jsonArray2 = JSONArray.fromObject(list);

		System.out.println(jsonArray2);

	}

}
```


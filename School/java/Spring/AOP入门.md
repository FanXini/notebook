# AOP入门

> AOP:Aspect-oriented Programming 面向切面编程。

在一个服务的流程中插入与业务无关的系统服务逻辑（Logging,Security）

这样的逻辑称为 **Cross-cutting concerns**（横切关注点）。将 **cross-cutting concerns**独立设计成为一个对象，这样的特殊对象就称为Aspect。

从代理机制入门AOP：

## 静态代理

例如有一个输出Hello类，要想在方法前后插入Log日志

```java
package team.fan.proxy;

public class HelloSpeaker{

	public HelloSpeaker() {
	}

	public void hello(String name) {
		System.out.println(name);
	}
}
```

但是如果这样设计的话，因为日志功能并不是业务逻辑的一部分，将日志程序加入业务逻辑中的话，会增加***HelloSpeaker***额外的职责。而且如果程序设计中每一个类中的方法都需要增加日志功能，则会大大增加代码的冗余度。如果需要的服务（service）不只是日志动作，有一些非对象本身职责的相关动作也混入了对象之中（权限检查、事务管理等），这会导致对象的负担进一步加重，甚至混淆了对象本身的职责。

另一方面，当要移除日志功能时，无法简单的将相关服务从程序中移除。

解决方法：使用代理（proxy）机制解决这个问题。

**程序清单1.先定义一个接口：**

```java
package team.fan.proxy;

public interface IHello{

	public void hello(String name);

}

```

**程序清单2.定义实现类（被代理对象），实现IHello接口**

```java
package team.fan.proxy;

/*业务逻辑部分 被代理对象*/
public class HelloSpeaker implements IHello{

	public HelloSpeaker() {
	}

	@Override  
	public void hello(String name) {
		System.out.println(name);
	}

}

```

**程序清单3.定义代理对象，同样也实现IHello接口**

```java
package team.fan.proxy;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
//代理对象
public class HelloProxy implements IHello{
	private static Log log=LogFactory.getLog(HelloProxy.class);
	private IHello helloSpeak;
	public HelloProxy(IHello helloSpeak) {
		this.helloSpeak=helloSpeak;
	}

	@Override
	public void hello(String name) {
		log.info("before");
		helloSpeak.hello(name); //调用被被代理对象
		log.info("after");
	}

}

```

**程序清单4.Main**

```java
package team.fan.proxy;

public class ProxyMain {
	public static void main(String args[]){
		HelloProxy helloProxy=new HelloProxy(new HelloSpeaker());
		helloProxy.hello("hello world");		
	}
}
```

**输出**

```java
十二月 26, 2018 3:12:52 下午 team.fan.proxy.HelloProxy hello
信息: before
hello world
十二月 26, 2018 3:12:52 下午 team.fan.proxy.HelloProxy hello
信息: after

```

&emsp;&emsp;这样，HelloSpeaker类中没有增加任何与其本身职责无关的代码，日志服务的实现被放在了代理对象之中，而功能的实现实际上是调用了代理类HelloProxy。

&emsp;&emsp;**<font color='red'>静态代理的缺点：当存在多个需要加入日志功能的类时，需要为每个类单独创建一个代理对象，复杂度也比较高。</font>**



## 动态代理

[原文章](http://blog.csdn.net/rokii/article/details/4046098)

动态代理其实就是java.lang.reflect.Proxy类动态的根据你指定的所有接口生成一个class byte，该class会继承Proxy类，并实现所有你指定的接口（您在参数中传入的接口数组）；然后再利用您指定的classloader将 class byte加载进系统，最后生成这样一个类的对象，并初始化该对象的一些值，如invocationHandler,以即所有的接口对应的Method成员。 初始化之后将对象返回给调用的客户端。这样客户端拿到的就是一个实现你所有的接口的Proxy对象。请看实例分析：

**一  业务接口类**

```java
package team.fan.proxy;

public interface Animal {  //接口

	public void eat(String foodName);
	public void action(String actionName);
}

```

**二 业务实现类**

```java
package team.fan.proxy;

public class Dog implements Animal{

	public Dog() {
	}

	@Override
	public void eat(String foodName) {
		System.out.println(foodName);
		
	}

	@Override
	public void action(String actionName) {
		System.out.println(actionName);	
	}
}
```

**三 业务代理类**

```java
package team.fan.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.logging.Logger;

public class LogHander implements InvocationHandler{
	
	private Logger log=Logger.getLogger(this.getClass().getName());
	
	////被代理对象
	private Object delegate; 
	
	 //实际上是根据了该对象实现的接口重新定义了一个代理对象
	public Object bind(Object delegate){
		this.delegate=delegate;
		return Proxy.newProxyInstance(delegate.getClass().getClassLoader(), delegate.getClass().getInterfaces(), this);
	}
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object result=null;
		try {
			log.info("Dynatic before"+proxy.getClass());
			result=method.invoke(delegate, args);
			log.info("Dynatic after"+proxy.getClass());
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}
}
```

**四 客户端应用类**

```java
package team.fan.proxy;

public class DogProxyMain {

	public static void main(String agrs[]){
		LogHander logHander=new LogHander();
		Animal dogProxy=(Animal) logHander.bind(new Dog()); 
		dogProxy.action("旺旺"); //调用的是bind函数生成的代理对象
		dogProxy.eat("哇哈哈");
	}
}
```

**五 打印结果：**

```java
十二月 26, 2018 3:33:53 下午 team.fan.proxy.LogHander invoke
信息: Dynatic beforeclass com.sun.proxy.$Proxy0
旺旺
十二月 26, 2018 3:33:53 下午 team.fan.proxy.LogHander invoke
信息: Dynatic afterclass com.sun.proxy.$Proxy0
十二月 26, 2018 3:33:53 下午 team.fan.proxy.LogHander invoke
信息: Dynatic beforeclass com.sun.proxy.$Proxy0
屎
十二月 26, 2018 3:33:53 下午 team.fan.proxy.LogHander invoke
信息: Dynatic afterclass com.sun.proxy.$Proxy0

```

&emsp;&emsp;通过结果我们就能够很简单的看出Proxy的作用了，它能够在你的核心业务方法前后做一些你所想做的辅助工作，如log日志，安全机制等等。现在我们来分析一下上面的类的工作原理。

&emsp;&emsp;类一二没什么好说的。先看看类三吧。 实现了InvocationHandler接口的invoke方法。其实这个类就是最终Proxy调用的固定接口方法。Proxy不管客户端的业务方法是怎么实现的。当客户端调用Proxy时，它只会调用InvocationHandler的invoke接口，所以我们的真正实现的方法就必须在invoke方法中去调用。关系如下：

```java
LogHander logHander=new LogHander();
Animal dogProxy=(Animal) logHander.bind(new Dog()); 
dogProxy.action("旺旺"); //调用的是bind函数生成的代理对象
dogProxy.eat("哇哈哈");
```

**dogProxy.eat()—>invocationHandler.invoke()—>new Dog().eat();**

那么**dogProxy**到底是怎么样一个对象呢。我们改一下main方法看一下就知道了：

```java
package team.fan.proxy;

public class DogProxyMain {

	public static void main(String agrs[]){
		LogHander logHander=new LogHander();
		Animal dogProxy=(Animal) logHander.bind(new Dog()); 
		dogProxy.action("旺旺"); //调用的是bind函数生成的代理对象
		dogProxy.eat("屎");
		System.out.println(dogProxy.getClass().getName());
	}
}
```

**输出结果：**

```java
十二月 26, 2018 3:39:56 下午 team.fan.proxy.LogHander invoke
信息: Dynatic beforeclass com.sun.proxy.$Proxy0
旺旺
十二月 26, 2018 3:39:56 下午 team.fan.proxy.LogHander invoke
信息: Dynatic afterclass com.sun.proxy.$Proxy0
十二月 26, 2018 3:39:56 下午 team.fan.proxy.LogHander invoke
信息: Dynatic beforeclass com.sun.proxy.$Proxy0
屎
十二月 26, 2018 3:39:56 下午 team.fan.proxy.LogHander invoke
信息: Dynatic afterclass com.sun.proxy.$Proxy0
com.sun.proxy.$Proxy0

```

&emsp;&emsp;**dogProxy**原来是个**$Proxy0**这个类的对象。那么这个类到底是长什么样子呢？好的。我们再写二个方法去把这个类打印出来看个究竟，是什么三头六臂呢？我们在main下面写如下两个静态方法。并再改写main方法：

```java
package team.fan.proxy;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

public class DogProxyMain {

	public static void main(String agrs[]) {
		LogHander logHander = new LogHander();
		Animal dogProxy = (Animal) logHander.bind(new Dog());
		dogProxy.action("旺旺"); // 调用的是bind函数生成的代理对象
		dogProxy.eat("屎");
		printClassDefinition(dogProxy.getClass());
	}

	public static String getModifier(int modifier) {

		String result = "";

		switch (modifier) {

		case Modifier.PRIVATE:
			result = "private";

		case Modifier.PUBLIC:
			result = "public";

		case Modifier.PROTECTED:
			result = "protected";

		case Modifier.ABSTRACT:
			result = "abstract";

		case Modifier.FINAL:
			result = "final";

		case Modifier.NATIVE:
			result = "native";

		case Modifier.STATIC:
			result = "static";

		case Modifier.SYNCHRONIZED:
			result = "synchronized";

		case Modifier.STRICT:
			result = "strict";

		case Modifier.TRANSIENT:
			result = "transient";

		case Modifier.VOLATILE:
			result = "volatile";

		case Modifier.INTERFACE:
			result = "interface";

		}

		return result;

	}

	public static void printClassDefinition(Class clz) {
		String clzModifier = getModifier(clz.getModifiers());
		if (clzModifier != null && !clzModifier.equals("")) {
			clzModifier = clzModifier + " ";
		}
		String superClz = clz.getSuperclass().getName();
		if (superClz != null && !superClz.equals("")) {
			superClz = "extends " + superClz;
		}

		Class[] interfaces = clz.getInterfaces();

		String inters = "";

		for (int i = 0; i < interfaces.length; i++) {

			if (i == 0) {
				inters += "implements ";
			}

			inters += interfaces[i].getName();

		}

		System.out.println(clzModifier + clz.getName() + " " + superClz + " " + inters);

		System.out.println("{");

		Field[] fields = clz.getDeclaredFields();

		for (int i = 0; i < fields.length; i++) {

			String modifier = getModifier(fields[i].getModifiers());

			if (modifier != null && !modifier.equals("")) {
				modifier = modifier + " ";
			}

			String fieldName = fields[i].getName();

			String fieldType = fields[i].getType().getName();

			System.out.println("    " + modifier + fieldType + " " + fieldName + ";");

		}

		System.out.println();

		Method[] methods = clz.getDeclaredMethods();

		for (int i = 0; i < methods.length; i++) {

			Method method = methods[i];

			String modifier = getModifier(method.getModifiers());

			if (modifier != null && !modifier.equals("")) {
				modifier = modifier + " ";
			}

			String methodName = method.getName();

			Class returnClz = method.getReturnType();

			String retrunType = returnClz.getName();

			Class[] clzs = method.getParameterTypes();

			String paraList = "(";

			for (int j = 0; j < clzs.length; j++) {
				paraList += clzs[j].getName();
				if (j != clzs.length - 1) {
					paraList += ", ";
				}
			}
			paraList += ")";

			clzs = method.getExceptionTypes();

			String exceptions = "";

			for (int j = 0; j < clzs.length; j++) {
				if (j == 0) {
					exceptions += "throws ";
				}
				exceptions += clzs[j].getName();
				if (j != clzs.length - 1) {
					exceptions += ", ";
				}
			}
			exceptions += ";";
			String methodPrototype = modifier + retrunType + " " + methodName + paraList + exceptions;

			System.out.println("    " + methodPrototype);

		}

		System.out.println("}");

	}

}
```

**现在我们再看看输出结果：**

```java
十二月 26, 2018 3:49:10 下午 team.fan.proxy.LogHander invoke
信息: Dynatic beforeclass com.sun.proxy.$Proxy0
旺旺
十二月 26, 2018 3:49:10 下午 team.fan.proxy.LogHander invoke
信息: Dynatic afterclass com.sun.proxy.$Proxy0
十二月 26, 2018 3:49:10 下午 team.fan.proxy.LogHander invoke
信息: Dynatic beforeclass com.sun.proxy.$Proxy0
屎
十二月 26, 2018 3:49:10 下午 team.fan.proxy.LogHander invoke
信息: Dynatic afterclass com.sun.proxy.$Proxy0
com.sun.proxy.$Proxy0 extends java.lang.reflect.Proxy implements team.fan.proxy.Animal
{
    java.lang.reflect.Method m1;
    java.lang.reflect.Method m2;
    java.lang.reflect.Method m4;
    java.lang.reflect.Method m3;
    java.lang.reflect.Method m0;

    boolean equals(java.lang.Object);
    java.lang.String toString();
    int hashCode();
    void action(java.lang.String);
    void eat(java.lang.String);
}
```

很明显，<font color='red'>Proxy.newProxyInstance(delegate.getClass().getClassLoader(), delegate.getClass().getInterfaces(), this);</font>方法会做如下几件事：

1. 根据传入的第二个参数interfaces动态生成一个类，实现interfaces中的接口，该例中即Animal接口的active和eat方法。并且继承了Proxy类，重写了hashcode,toString,equals等三个方法。具体实现可参看 ProxyGenerator.generateProxyClass(...); 该例中生成了$Proxy0类

2. 通过传入的第一个参数classloder将刚生成的类加载到jvm中。即将$Proxy0类load

3. 利用第三个参数，调用$Proxy0的$Proxy0(InvocationHandler)构造函数 创建$Proxy0的对象，并且用interfaces参数遍历其所有接口的方法，并生成Method对象初始化对象的几个Method成员变量

4. 将$Proxy0的实例返回给客户端。

现在好了。我们再看客户端怎么调就清楚了。

1. 客户端拿到的是$Proxy0$的实例对象，由于Proxy0实现了Animal，因此转化为Animal没任何问题。

```java
Animal dogProxy = (Animal) logHander.bind(new Dog());
```

2. dogProxy.eat()；

实际上调用的是$Proxy0.eat()$;那么Proxy0.eat()的实现就是通过InvocationHandler去调用invoke方法啦!
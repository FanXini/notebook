typora-copy-images-to: ..\..\img

# <center>面向切面的Spring</center>

## 1. 术语

&emsp;&emsp;与大多数技术一样，AOP已经形成了自己的术语。描述切面的常用术语有通知（advice）、切点（pointcut）和连接点（join point）。图4.2展示了这些概念是如何关联在一起的。

![1545811860928](..\..\..\..\img\1545811860928.png)

### 1.1 通知（Advice）

&emsp;&emsp;**通知定义了切面是做什么的以及何时使用**。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。它应该应用在某个方法被调用之前？之后？之前和之后都调用？还是只在方法抛出异常时调用？

**Spring切面可以应用5种类型的通知：**

- 前置通知（Before）：在目标方法被调用之前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
- 返回通知（After-returning）：在目标方法成功执行之后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

### 1.2 连接点（Join point）

&emsp;&emsp;连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

### 1.3 切点（Poincut）

&emsp;&emsp;**如果说通知定义了切面的“做什么”和“何时”的话，那么切点就定义了“何处”**。切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

### 1.4 切面（Aspect）

&emsp;&emsp;切面是**通知和切点**的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。

### 1.5 引入（Introduction）

&emsp;&emsp; 用来给一个类型声明额外的方法或属性（也被称为连接类型声明（inter-type declaration））。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用引入来使一个bean实现IsModified接口，以便简化缓存机制。

### 1.6 织入（Weaving）

&emsp;&emsp;织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入：

- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-time
  weaving，LTW）就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

## 2. AOP的4种实现

Spring提供了4种类型的AOP支持：

- 基于代理的经典Spring AOP；
- 纯POJO切面；
- @AspectJ注解驱动的切面；
- 注入式AspectJ切面（适用于Spring各版本）。

&emsp;&emsp;前三种都是Spring AOP实现的变体，Spring AOP构建在<font color='red'>**动态代理**</font>基础之上，因此，Spring对AOP的支持局限于方法拦截。**引入了简单的声明式AOP和基于注解的AOP之后，Spring经典的AOP看起来就显得非常笨重和过于复杂，直接使用ProxyFactory Bean会让人感觉厌烦。所以在本书中我不会再介绍经典的Spring AOP。**

&emsp;&emsp;Spring借鉴了AspectJ的切面，以提供注解驱动的AOP。**本质上，它依然是Spring基于代理的AOP，但是编程模型几乎与编写成熟的AspectJ注解切面完全一致。**这种AOP风格的好处在于能够不使用XML来完成功能。
&emsp;&emsp;如果你的AOP需求超过了简单的方法调用（如构造器或属性拦截），那么你需要考虑使用AspectJ来实现切面。在这种情况下，上文所示的第四种类型能够帮助你将值注入到AspectJ驱动的切面中。

## 3. Spring AOP 技术的关键知识

### 3.1 Spring通知是Java编写的

&emsp;&emsp;Spring所创建的通知都是用标准的Java类编写的。这样的话，我们就可以使用与普通Java开发一样的集成开发环境（IDE）来开发切面。而且，定义通知所应用的切点通常会使用注解或在Spring配置文件里采用XML来编写，这两种语法对于Java开发者来说都是相当熟悉的。

&emsp;&emsp;AspectJ与之相反。虽然AspectJ现在支持基于注解的切面，但AspectJ最初是以Java语言扩展的方式实现的。这种方式有优点也有缺点。通过特有的AOP语言，我们可以获得更强大和细粒度的控制，以及更丰富的AOP工具集，**但是我们需要额外学习新的工具和语法。**

### 3.2 Spring在运行时通知对象

&emsp;&emsp;通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。如图4.3所示，代理类封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。

![1545814084479](..\..\..\..\img\1545814084479.png)

**图4.3　Spring的切面由包裹了目标对象的代理类实现。代理类处理方法的调用，执行额外的切面逻辑，并调用目标方法**

### 3.3 Spring只支持方法级别的连接点

&emsp;&emsp;因为Spring基于动态代理，所以Spring只支持方法连接点。这与一些其他的AOP框架是不同的，例如**AspectJ和JBoss**，除了方法切点，它们还提供了**字段和构造器接入点**。Spring缺少对字段连接点的支持，无法让我们创建细粒度的通知，例如拦截对象字段的修改。而且它不支持构造器连接点，我们就无法在bean创建时应用通知。

&emsp;&emsp;但是方法拦截可以满足绝大部分的需求。如果需要方法拦截之外的连接点拦截功能，那么我们可以利用Aspect来补充Spring AOP的功能。

## 4 通过切点来选择连接点

### 4.1 语法

**表4.1列出了Spring AOP所支持的AspectJ切点指示器。**

![1544016339986](..\..\..\..\img\1544016339986.png)

&emsp;&emsp;在Spring中尝试使用AspectJ其他指示器时，将会抛出IllegalArgument-Exception异常。

&emsp;&emsp;注意只有**execution**指示器是实际执行匹配的，**而其他的指示器都是用来限制匹配的。**

### 4.2 编写切点

**定义一个Performance接口**

```java
package concert;

public interface Performance {
	
	public void perform();

}
```

图4.4展现了一个切点表达式，这个表达式能够设置当perform()方法执行时触发通知的调用。

![1545816108183](..\..\..\..\img\1545816108183.png)

&emsp;&emsp;我们使用**execution()**指示器选择Performance的perform()方法。方法表达式以“*”号开始，表明了**我们不关心方法返回值的类型**。然后，我们指定了全限定类名和方法名。对于方法参数列表，我们使用两个点号（..）表明切点要选择任意的perform()方法，**无论该方法的入参是什么**。

&emsp;&emsp;现在假设我们需要配置的切点仅匹配concert包。在此场景下，可以使用within()指示器来限制匹配，如图4.5所示。

![1545871144945](..\..\..\..\img\1545871144945.png)

上面使用了&&表示与操作。**操作符使用**

|          | 与   | 或   | 非   |
| -------- | ---- | ---- | ---- |
| Java配置 | &&   | \|\| | !    |
| Xml配置  | and  | or   | no   |

#### 4.2.1在切点中选择bean

&emsp;&emsp;<font color='red'>除了表4.1所列的指示器外，Spring还引入了一个新的bean()指示器</font>，它允许我们在切点表达式中使用bean的ID来标识bean。bean()使用bean ID或bean名称作为参数来限制切点只匹配特定的bean。

例如，考虑如下的切点：

![1545871741338](..\..\..\..\img\1545871741338.png)

&emsp;&emsp;在这里，我们希望在执行Performance的perform()方法时应用通知，但限定bean的ID为woodstock。

&emsp;&emsp;在某些场景下，限定切点为指定的bean或许很有意义，但我们还可以使用非操作为除了特定ID以外的其他bean应用通知：

![1545871828506](..\..\..\..\img\1545871828506.png)

&emsp;&emsp;在此场景下，切面的通知会被编织到所有ID不为woodstock的bean中。
&emsp;&emsp;现在，我们已经讲解了编写切点的基础知识，让我们再了解一下如何编写通知和使用这些切点声明切面。

## 4.3　使用注解创建切面

&emsp;&emsp;使用注解来创建切面是AspectJ 5所引入的关键特性。AspectJ 5之前，编写AspectJ切面需要学习一种Java语言的扩展，但是AspectJ面向注解的模型可以非常简便地通过少量注解把任意类转变为切面。

### 4.3.1 五个注解

AspectJ提供五个注解来定义通知，如表4.2所示。

![1545873045475](..\..\..\..\img\1545873045475.png)

**程序1 定义一个Performance 实现类：**

```java
package concert;

public class JChou implements Performance{

	@Override
	public void perform() {
		System.out.println("告白气球");		
	}
}
```

**定义切面**

**程序2 定义一个Audience(观众)类：**

```java
package concert;

import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class Audience {

	//表演之前-手机静音
	@Before("execution(**concert.Performance.perform(..))")
	public void silenceCellPhones() {
		System.out.println("Slilence cell phones");
	}
	//表演之前-坐好座位
	@Before("execution(**concert.Performance.perform(..))")
	public void takeSeats() {
		System.out.println("Taking seats");
	}
	//表演中间-鼓掌
	@AfterReturning("execution(**concert.Performance.perform(..))")
	public void applause() {
		System.out.println("CLAP CLAP CLAP");
	}
	//表演失败-要求退票
	@AfterThrowing("execution(**concert.Performance.perform(..))")
	public void demandRefund() {
		System.out.println("Demanding a refund");
	}
}

```

&emsp;&emsp;Audience类使用@AspectJ注解进行了标注。<font color='red'>该注解表明Audience不仅仅是一个POJO，还是一个切面。</font>

### @PointCut

&emsp;&emsp;Audience类中的方法都使用注解来定义切面的具体行为。上面同一个切点表达式重复使用了四次，可以通过定义一个表达式引用，没次要用的时候引用就行了，解决办法就是使用**@PointCut**:

**程序3 使用$@PointCut$定义切点表达式：**

```java
package concert;

import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class Audience {
	
	//定义切点
	@Pointcut("execution(**concert.Performance.perform(..))")
	public void performance() {}

	//表演之前-手机静音
	@Before("performance()")
	public void silenceCellPhones() {
		System.out.println("Slilence cell phones");
	}
	//表演之前-坐好座位
	@Before("performance()")
	public void takeSeats() {
		System.out.println("Taking seats");
	}
	//表演中间-鼓掌
	@AfterReturning("performance()")
	public void applause() {
		System.out.println("CLAP CLAP CLAP");
	}
	//表演失败-要求退票
	@AfterThrowing("performance()")
	public void demandRefund() {
		System.out.println("Demanding a refund");
	}
}
```

&emsp;&emsp;**performance()**方法的实际内容并不重要，在这里它实际上应该是空的。其实该方法本身只是一个标识，供@Pointcut注解依附。

### 4.3.2 使用JavaConfig启动切面

&emsp;&emsp;需要注意的是，除了注解和没有实际操作的performance()方法，Audience类依然是一个POJO。我们能够像使用其他的Java类那样调用它的方法，它的方法也能够独立地进行单元测试，这与其他的Java类并没有什么区别。<font color='red'>Audience只是一个Java类，只不过它通过注解表明会作为切面使用而已。</font>

&emsp;&emsp;像其他的Java类一样，它可以装配为Spring中的bean。

```java
@Bean
public Audience audience() {
	return new Audience();
}
```

&emsp;&emsp;但是如果只是这样定义的话，即便使用了$@AspectJ$注解，Spring页只会把它看作成一个简单的bean，而不会视为一个切面，这些注解不会解析，也不会创建将其转换为切面的代理。

&emsp;&emsp;如果你使用JavaConfig的话，可以在配置类的类级别上通过使用EnableAspectJ-AutoProxy注解启用自动代理功能。

**程序4：如何在JavaConfig中启用自动代理。**

```java
package concert;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@ComponentScan
@EnableAspectJAutoProxy   //启动AspectJ自动代理
public class AOPConfig {

	//声明 Audience bean
	@Bean
	public Audience audience() {
		return new Audience();
	}	
}
```

**测试**

```java
package concert;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=AOPConfig.class)
public class Main {
	@Autowired
	private Performance JChou;
	@Test
	public void Test() {
		JChou.perform();
	}
}
```

**输出结果**

```
Slilence cell phones
Taking seats
告白气球
CLAP CLAP CLAP
```

### 4.3.3 使用XML配置启动切面

![1545874845482](..\..\..\..\img\1545874845482.png)

&emsp;&emsp;不管你是使用JavaConfig还是XML，AspectJ自动代理都会为使用@Aspect注解的bean创建一个代理，这个代理会围绕着所有该切面的切点所匹配的bean。在这种情况下，将会为Concert bean创建一个代理，Audience类中的通知方法将会在perform()调用前后执行。

### 谨记

&emsp;&emsp;我们需要记住的是，<font color='red'>**Spring的AspectJ自动代理仅仅使用@AspectJ作为创建切面的指导，切面依然是基于代理的**</font>。在本质上，它依然是Spring基于代理的切面。这一点非常重要，因为这意味着尽管使用的是@AspectJ注解，但我们仍然限于代理方法的调用。如果想利用AspectJ的所有能力，我们必须在运行时使用AspectJ并且不依赖Spring来创建基于代理的切面。

### 4.3.4 创建环绕通知

&emsp;&emsp;环绕通知是最为强大的通知类型。它能够让你所编写的逻辑将被通知的目标方法完全包装起来。实际上就像在一个通知方法中同时编写前置通知和后置通知。为了阐述环绕通知，我们重写Audience切面。这次，我们使用一个环绕通知来代替之前多个不同的前置通知和后置通知。

**程序4　使用环绕通知重新实现Audience切面**

```java
package concert;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class Audience {

	// 定义切点
	@Pointcut("execution(**concert.Performance.perform(..)")
	public void performance() {
	}

	@Around("performance()")
	public void watchPerformance(ProceedingJoinPoint jp) {
		try {
			System.out.println("Silence cell phones");
			System.out.println("Taking seat");
			jp.proceed();
			System.out.println("CLAP CLAP CLAP");
		} catch (Throwable e) {
			System.out.println("Demanding a refund");
		}
	}
}

```

**输出结果**

```java
Silence cell phones
Taking seat
告白气球
CLAP CLAP CLAP
```

&emsp;&emsp;关于这个新的通知方法，你首先注意到的可能是它接受$ProceedingJoinPoint$作为参数。这个对象是必须要有的，因为你要在通知中通过它来调用被通知的方法。通知方法中可以做任何的事情，当要将控制权交给被通知的方法时，它需要调用**ProceedingJoinPoint**的**proceed()**方法。

&emsp;&emsp;有意思的是，你可以不调用proceed()方法，从而阻塞对被通知方法的访问，与之类似，你也可以在通知中对它进行多次调用。要这样做的一个场景就是实现重试逻辑，也就是在被通知方法失败后，进行重复尝试。

### 4.3.5 处理通知中的参数

&emsp;&emsp;如果切面所通知的方法有参数该怎么办呢？切面能访问和使用传递给被通知方法的参数吗？

&emsp;&emsp;比如，现在要记录每个班的学生进出班级的次数，可是记录次数并不属于学生出入的教室的业务逻辑，因此要定义一个切面进行出入次数记录的任务。

1. **定义Student类**

   ```java
   package ParamAdvice;
   
   import org.springframework.stereotype.Component;
   
   import lombok.Data;
   
   @Data  //使用了lombok插件，自动配置get set方法
   public class Student {
   	
   	private String sno;
   	public Student() {
   		
   	}
   	public Student(String sno) {
   		this.sno=sno;
   	}
   	public void enterClassRoom(String sno) {
   		System.out.println("学生："+sno+" 进入教室");
   	}
   	
   	public void outClassRoom(String sno) {
   		System.out.println("学生："+sno+" 出去教室");
   	}
   }
   ```

2. **定义ClassRoom类**

   ```java
   package ParamAdvice;
   
   import java.util.ArrayList;
   import java.util.List;
   
   import lombok.Data;
   @Data
   public class ClassRoom {
   	
   	private List<Student> students;
   	
   	public  ClassRoom() {
   		students=new ArrayList<>();
   
   	}	
   	public void restTime() {
   		for(int i=0;i<30;i++) {
   			Student student=students.get((int)(Math.random()*49)+1);
   			double j=Math.random();
   			if(j<=0.5) {
   				student.enterClassRoom(student.getSno());
   			}
   			else {
   				student.outClassRoom(student.getSno());
   			}
   		}
   	}
   
   }
   ```

3. **定义切面 CountAspect**

   ```java
   package ParamAdvice;
   
   import java.util.HashMap;
   import java.util.Map;
   
   import org.aspectj.lang.annotation.After;
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Pointcut;
   import lombok.Data;
   
   
   @Aspect
   @Data
   public class CountAspect {
   	
   	private Map<String, Integer> enterMap;
   	
   	private Map<String, Integer> outMap;
   	@Pointcut("execution(** ParamAdvice.Student.enterClassRoom(String))&&args(sno)")
   	public void enter(String sno) {
   		
   	}
   	
   	public CountAspect() {
   		enterMap=new HashMap<>();
   		outMap=new HashMap<>();
   	}
   	@After("enter(sno)")   //使用@PointCut配置
   	public void enterAction(String sno) {
   		if(enterMap.containsKey(sno)) {
   			enterMap.put(sno, enterMap.get(sno)+1);
   		}
   		else {
   			enterMap.put(sno,1);
   		}
   		System.out.println("student:"+sno+"进入教室");
   	}
   	@After("execution(** ParamAdvice.Student.outClassRoom(String))"+"&&args(sno)")
   	public void outAction(String sno) {
   		if(outMap.containsKey(sno)) {
   			outMap.put(sno, outMap.get(sno)+1);
   		}
   		else {
   			outMap.put(sno,1);
   		}
   		System.out.println("student:"+sno+"出去教室");
   	}
   }
   
   ```

4. **javaConfig文件**

   ```java
   package ParamAdvice;
   
   import static org.junit.Assert.assertArrayEquals;
   
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.EnableAspectJAutoProxy;
   import org.springframework.context.annotation.Scope;
   
   import com.sun.org.apache.xml.internal.resolver.helpers.PublicId;
   
   @Configuration
   @EnableAspectJAutoProxy  //开启AspectJ自动代理
   @ComponentScan(basePackageClasses= {Student.class})
   public class ParamAdviceConfig {
   
   	@Bean
   	public CountAspect countAspect() {
   		return new CountAspect();
   	}
   	
   	@Bean
   	public ClassRoom classRoom() {
   		ClassRoom classRoom= new ClassRoom();
   		for(int i=0;i<50;i++) {
   			classRoom.getStudents().add(studentFactory(String.valueOf(i)));
   		}
   		return classRoom;
   		
   	}
   	
   	@Bean
   	@Scope("prototype")
   	public Student studentFactory(String sno) {
   		return new Student(sno);
   	}
   	
   	@Bean
   	@Qualifier("noSnoStudent")
   	public Student noSnoStudent() {
   		return new Student();
   	}
   	
   	@Bean
   	@Qualifier("snoStudent")
   	public Student snoStudent() {
   		String sno=String.valueOf((int)(Math.random()*100));
   		return new Student(sno);
   	}
   }
   ```

5. **测试**

   ```java
   package ParamAdvice;
   
   import java.util.Map;
   
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.test.context.ContextConfiguration;
   import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
   
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(classes=ParamAdviceConfig.class)
   public class TestMain {
   	@Autowired
   	private ClassRoom classRoom;
   	@Autowired
   	private CountAspect aspect;
   	
   	@Autowired
   	@Qualifier("snoStudent")
   	private Student student;
   	
   	@Test
   	public void Test(){
   		classRoom.restTime();
   		for(Map.Entry<String, Integer> entry:aspect.getEnterMap().entrySet()) {
   			System.out.println(entry.getKey()+":"+entry.getValue());
   		}
   		for(Map.Entry<String, Integer> entry:aspect.getOutMap().entrySet()) {
   			System.out.println(entry.getKey()+":"+entry.getValue());
   		}		
   	}
   }
   
   ```

6. **输出**

   ```java
   学生：37 出去教室
   student:37出去教室
   学生：15 进入教室
   student:15进入教室
   学生：23 出去教室
   student:23出去教室
   学生：32 出去教室
   student:32出去教室
   学生：9 进入教室
   student:9进入教室
   学生：24 出去教室
   student:24出去教室
   学生：4 出去教室
   student:4出去教室
   学生：27 出去教室
   student:27出去教室
   学生：12 出去教室
   student:12出去教室
   学生：3 出去教室
   student:3出去教室
   学生：33 出去教室
   student:33出去教室
   学生：4 出去教室
   student:4出去教室
   学生：14 进入教室
   student:14进入教室
   学生：45 出去教室
   student:45出去教室
   学生：25 进入教室
   student:25进入教室
   学生：20 出去教室
   student:20出去教室
   学生：42 出去教室
   student:42出去教室
   学生：15 进入教室
   student:15进入教室
   学生：44 进入教室
   student:44进入教室
   学生：32 进入教室
   student:32进入教室
   学生：46 进入教室
   student:46进入教室
   学生：9 出去教室
   student:9出去教室
   学生：5 出去教室
   student:5出去教室
   学生：34 出去教室
   student:34出去教室
   学生：45 进入教室
   student:45进入教室
   学生：40 进入教室
   student:40进入教室
   学生：34 出去教室
   student:34出去教室
   学生：7 进入教室
   student:7进入教室
   学生：11 进入教室
   student:11进入教室
   学生：13 出去教室
   student:13出去教室
   *******************进入次数统计******************
   44:1
   11:1
   45:1
   46:1
   14:1
   25:1
   15:2
   7:1
   9:1
   40:1
   32:1
   *******************出去次数统计******************
   33:1
   23:1
   12:1
   45:1
   34:2
   24:1
   13:1
   37:1
   27:1
   3:1
   4:2
   5:1
   9:1
   20:1
   42:1
   32:1
   ```

## 5 通过注解增添新功能

&emsp;&emsp;前面介绍的都是使用切面给方法增加新功能，但是没有增加新方法。实际上，利用被称为引入的AOP概念，切面可以为Spring bean添加新方法。

&emsp;&emsp;回顾一下，在Spring中，切面只是实现了它们所包装bean相同接口的代理。如果除了实现这些接口，代理也能暴露新接口的话，会怎么样呢？那样的话，切面所通知的bean看起来像是实现了新的接口，即便底层实现类并没有实现这些接口也无所谓。图4.7展示了它们是如何工作的。

![1545892487363](..\..\..\..\img\1545892487363.png)

&emsp;&emsp;我们需要注意的是，当引入接口的方法被调用时，代理会把此调用委托给实现了新接口的某个其他对象。<font color='red'>实际上，一个bean的实现被拆分到了多个类中。</font>

实例，以上面Performance接口为例子，现在希望不修改原有代码的情况下，增加一个方法。

1. **增加一个接口**

   ```java
   package concert;
   
   public interface Encorable {
   	void performEncore();
   }
   ```

2. **增加一个对应的实现类**

   ```java
   package concert;
   
   public class DefaultEncoreable implements Encorable{
   
   	@Override
   	public void performEncore() {
   		System.out.println("开机音乐");
   	}
   }
   ```

3. **增加对应的切面**

   ```java
   package concert;
   
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.DeclareParents;
   @Aspect
   public class EncoreableIntroducer {
   
   	@DeclareParents(value="concert.Performance+",defaultImpl=DefaultEncoreable.class)
   	public static Encorable encorable;
   }
   
   ```

   &emsp;&emsp;可以看到，EncoreableIntroducer是一个切面。但是，它与我们之前所创建的切面不同，它并没有提供前置、后置或环绕通知，而是通过@DeclareParents注解，将Encoreable接口引入
   到Performance bean中。@DeclareParents注解由三部分组成：

   - value属性指定了哪种类型的bean要引入该接口。在本例中，也就是所有实现Performance的类型。（标记符后面的加号表示是Performance的所有子类型，而不是Performance身。）
   - defaultImpl属性指定了为引入功能提供实现的类。在这里，我们指定的是DefaultEncoreable提供实现。
   - @DeclareParents注解所标注的静态属性指明了要引入了接口。在这里，我们所引入的是Encoreable接口。

4. **JavaConfig**

   ```java
   package concert;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.EnableAspectJAutoProxy;
   
   @Configuration
   @EnableAspectJAutoProxy   //启动AspectJ自动代理
   public class AOPConfig {
   
   	//声明 Audience bean
   	@Bean
   	public Audience audience() {
   		return new Audience();
   	}
   	//声明 EncoreableIntroducer bean
   	@Bean
   	public EncoreableIntroducer encoreableIntroducer() {
   		return new EncoreableIntroducer();
   	}
   	@Bean 
   	public JChou jChou() {
   		return new JChou();
   	}	
   }
   ```

5. **测试**

   ```java
   package concert;
   
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.test.context.ContextConfiguration;
   import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
   
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(classes=AOPConfig.class)
   public class Main {
   	@Autowired
   	private Performance JChou;
   	@Test
   	public void Test() {
   		Encorable encorable=(Encorable)JChou; //类型强转
   		encorable.performEncore();
   	}
   
   }
   ```

6. **输出**

   ```java
   开机音乐
   ```

## 6 在XML中声明切面

**<font color='red'>见Spring实战第四版 160-168</font>**

## 7 注入AspectJ切面

**<font color='red'>见Spring实战第四版 168-171</font>**

## 6 总结

&emsp;&emsp;关于在Spring应用中如何使用切面，我们可以有多种选择。通过使用@AspectJ注解和简化的配置命名空间，在Spring中装配通知和切点变得非常简单。最后，当Spring AOP不能满足需求时，我们必须转向更为强大的AspectJ。对于这些场景，我们了解了如何使用Spring为AspectJ切面注入依赖。


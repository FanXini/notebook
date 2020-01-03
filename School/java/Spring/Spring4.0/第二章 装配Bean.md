---
typora-copy-images-to: ..\..\img
---

# <center>装配Bean</center>

## 装备Bean

- **声明bean**
- **构造器注入和Setter方法注入**
- **装配bean**
- **控制bean的创建和销毁**

## 背景

&emsp;&emsp;通常创建应用对象之间关联关系的传统方法（通过构造器或者查找）通常会导致结构复杂的代码。这些代码很难被复用或者单元测试。==而在Spring中，对象无需自己查找或创建与其所关联的其他对象。相反，容器负责将所需要相互协作的对象引用赋予各个对象==。**创建应用对象之间协作关系的行为通常称为装配（wiring），这也是依赖注入（DI）的本质**

## 1. 三种装配机制

- 在XML中进行显式配置。
- 在Java中进行显式配置。
- 隐式的bean发现机制和自动装配。	

&emsp;&emsp;Spring有多种可选方案来配置bean，这是非常棒的，但有时候你必须要在其中做出选择。这方面，并没有唯一的正确答案。你所做出的选择必须要适合你和你的项目。而且，谁说我们只能选择其中的一种方案呢？Spring的配置风格是可以互相搭配的，所以你可以选择使用XML装配一些bean，使用Spring基于Java的配置（JavaConfig）来装配另一些bean，而将剩余的bean让Spring去自动发现。

&emsp;&emsp;即便如此，我的建议是尽可能地使用自动配置的机制。显式配置越少越好。当你必须要显式配置bean的时候（比如，有些源码不是由你来维护的，而当你需要为这些代码配置bean的时候），我推荐使用类型安全并且比XML更加强大的JavaConfig。最后，只有当你想要使用便利的XML命名空间，并且在JavaConfig中没有同样的实现时，才应该使用XML。

## 2. 自动化装配Bean

Spring从两个角度来实现自动化装配

- 组件扫描（Component Scanning）:Spring会自动发现应用上下文中所创建的Bean
- 自动装配（AutoWiring）:Spring自动满足bean之间的依赖

### 程序清单 2.1 接口Animal定义

```java
package Bean;

public interface Animal {
	
	public void sound();
	
	public void function();

}

```

### 程序清单 2.2  带有*@Component*注解接口实现类 Dog

```java
package Bean;

import org.springframework.stereotype.Component;

@Component
public class Dog implements Animal{

	@Override
	public void sound() {
		// TODO Auto-generated method stub
		System.out.println("旺旺");
	}

	@Override
	public void function() {
		// TODO Auto-generated method stub
		System.out.println("看门");
	}
}
```

***@Component***这个简单的注解表明该类会作为**组件类**，并告知Spring要为这个类创建bean。没有必要显式配置Dog，因为这个类使用了@Component注解，所以Spring会为你把事情处理妥当。不过，组件扫描==默认是不启用==的。我们还需要显式配置一下Spring，从而命令它去寻找带有@Component注解的类，并为其创建bean。程序清单2.3的配置类展现了完成这项任务的最简洁配置。

### 程序清单2.3*@ComponentScan*注解启用了组件扫描

```java
package Bean;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class Config {

}
```

&emsp;&emsp;如果没有其他配置的话，***@ComponentScan***默认会扫描与配置类相同的包。因为**Conig**类在Bean包中，所以Spring会扫描这个包以及这个包下的所有子包，查找带有@Component逐渐的类，并自动为其创建一个bean。

**&emsp;&emsp;如果你更倾向于使用XML来启用组件扫描的话，那么可以使用Spring context命名空间的<context:component-scan>元素。程序清单2.4展示了启用组件扫描的最简洁XML配置。**

### 程序清单2.4 通过XML启用组件扫描

![1545029952728](..\..\..\..\img\1545029952728.png)

&emsp;&emsp;尽管我们可以通过XML的方案来启用组件扫描，但是在后面的讨论中，我更多的还是会使用基于Java的配置。如果你更喜欢XML的话，<context:component-scan>元素会有与@ComponentScan注解相对应的属性和子元素。

### 程序清单2.5 测试

```java
package Bean;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=Config.class)
public class JunitTest {
	@Autowired  //自动注入
	private Animal animal;
	@Test
	public void Test() {
		animal.function();
	}
}

//output:看门
```

&emsp;&emsp;JunitTest使用了Spring的SpringJUnit4ClassRunner，以便在测试开始的时候自动创建Spring的应用上下文。注解@ContextConfiguration会告诉它需要在Config类中加载配置。因为Config类加了@ComponentScan，因此最终的应用上下文会创建所在包的所有带@Component组件类的bean。

代码在animal上使用了@Autowire注解，Sping会为其自动注入一个合适的bean，测试中，如果结果输入了“看门”，说明对象不为空，注入成功。

### 2.6 组件Bean的命名

&emsp;&emsp;Spring应用上下文会为所用的bean都设置一个ID。在前面的例子中，虽然没有为Dog bean设置ID，但Spring会根据类名为其指定一个ID。具体来讲，这个bean所给定的ID为dog，也就是将类名的第一个字母变为小写。

&emsp;&emsp;如果相用自己定义的id，如果想将这个bean标识为lonelyHeartsClub：

![1545029927010](..\..\..\..\img\1545029927010.png)

### 2.7 设置组件扫描的基础包

&emsp;&emsp;如前面所示，Spring会自动设置配置类所在的包作为基础包(base package)来扫描组件。如果相扫描不同的包或者多个包，该怎么设置？

&emsp;&emsp;有一个原因会促使我们明确地设置基础包，那就是我们想要将配置类放在单独的包中，使其与其他的应用代码区分开来。如果是这样的话，那默认的基础包就不能满足要求了。

> 要满足这样的需求其实也完全没有问题！为了指定不同的基础包，你所需要做的就是在@ComponentScan的value属性中指明包的名称：

![1545030223936](..\..\..\..\img\1545030223936.png)

>  如果你想更加清晰地表明你所设置的是基础包，那么你可以通过basePackages属性进行配置：

![1545030260603](..\..\..\..\img\1545030260603.png)

>  如果想要配置多个包，只需要将basePackages属性设置为一个数组：

![1545030297177](..\..\..\..\img\1545030297177.png)

&emsp;&emsp;上面我们可以看出，包都是以String的类型描述，但这样是不安全的，不利于代码重构。因此，除了将包设置为简单的String类型之外，@ComponentScan还提供了另外一种方法，那就是将其指定为包中所包含的类或接口：![1545030617473](..\..\..\..\img\1545030617473.png)

&emsp;&emsp;可以看出，basePackages改成了basePackageClasses.。同时，我们不是再使用String类型的名
称来指定包，为basePackageClasses属性所设置的数组中包含了类。**这些类所在的包将会作为组件扫描的基础包。**==而且，建议创建一些专门用来扫描的空标记接口，而不要用组建类==。

### 2.8 自动化装配 Autowired的使用方法

> <font color='red'>简单来说，自动装配就是让Spring自动满足bean依赖的一种方法，在满足依赖的过程中，会在Spring**应用上下文中寻找匹配某个bean需求的其他bean。**为了声明要进行自动装配，我们可以借助Spring的@Autowired注解。</font>

@AutoWired可以用在以下地方

1. 直接用在类定义上
2. 构造器上
3. setter(方法名没有固定规定)方法上

```java
package Bean;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Family {
	//@Autowired   //直接用在对象定义上
	private  Animal pet;
	@Autowired     //用在构造器上
	public Family(Animal animal) {
		this.pet=animal;
	}
	
	public Animal getPet() {
		return pet;
	}
	
	//@Autowired       //用在setter方法上
	public void setPet(Animal pet) {
		this.pet=pet;
	}
}

```

&emsp;&emsp;不管是构造器、Setter方法还是其他的方法，Spring都会尝试满足方法参数上所声明的依赖。假如有且只有一个bean匹配依赖需求的话，那么这个bean将会被装配进来。**如果没有匹配的bean，那么在应用上下文创建的时候，Spring会抛出一个异常。**为了避免异常的出现，你可以将@Autowired的required属性设置为false：

![1545033754523](..\..\..\..\img\1545033754523.png)

&emsp;&emsp;如果有多个bean都能满足依赖关系的话，Spring将会抛出一个异常，表明没有明确指定要选择哪个bean进行自动装配。在第3章中，我们会进一步讨论自动装配中的歧义性。

#### 2.8.1 AutoWired的替代注解@Inject

![1545033979263](..\..\..\..\img\1545033979263.png)

## 3 通过Java代码装备bean

&emsp;&emsp;虽然很多场景推荐组件扫描和自动装备的方式，但有时候自动化的配置行不通，因此需要显式配置。并且有两个方案进行显式配置：

- Java（推荐的方案）
- XML

&emsp;&emsp;就像我之前所说的，在进行显式配置时，JavaConfig是更好的方案，因为它更为强大、类型安全并且对重构友好。因为它就是Java代码，就像应用程序中的其他Java代码一样。

&emsp;&emsp;虽然JavaConfig也是java代码，但因为它是配置代码，所以不应该把业务逻辑写进去。建议将javaConfig代码放到单独的包。

&emsp;&emsp;接下来看如何使用javaConfig显式配置Spring。

#### 3.1 创建配置类

```java
package 显式配置;

import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaConfig {

}
```

&emsp;&emsp;创建JavaConfig类的关键在于为其添加@Configuration注解，@Configuration注解表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。

#### 3.2　声明简单的bean

要在JavaConfig中声明bean，我们需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加@Bean注解。

```java
package 显式配置;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaConfig {
	@Bean
	public Flower getRose() {
		return new Rose();
	}

}
```

##### 3.2.1 Bean命名问题

==&emsp;&emsp;默认情况下，bean的ID与带有@Bean注解的方法名是一样的==,比如上例得到的bean ID为getRose。如果想自定义名字"rose":

```java
@Bean(name= "bean")
	public Flower getRose() {
		return new Rose();
	}
```

&emsp;&emsp;如果想取多个名字，取集合的方式

```java
@Bean(name= {"rose","rose1"})
	public Flower getRose() {
		return new Rose();
	}
```

#### 3.3 借助JavaConfig实现注入

##### 3.3.1创建一个Vase花瓶类，依赖于Flower接口

```java
package 显式配置;

public class Vase {

	private Flower flower;

	public Vase() {

	}

	public Vase(Flower flower) {
		this.flower=flower;
	}

	public Flower getFlower() {
		return flower;
	}

	public void setFlower(Flower flower) {
		this.flower = flower;
	}

}
```

&emsp;&emsp;在3.2中声明的bean比较简单，没有依赖关系。但现在这个Bean依赖于Flower，怎么使用JavaConfig将他们装配在一起？

- 方法1：引用创建Bean的方法

```java
@Configuration
public class JavaConfig {
	@Bean(name= {"rose","rose1"})
	public Flower getRose() {
		return new Rose();
	}
	@Bean
	public Vase getVase() {
		return new Vase(getRose());  //调用获取rose bean的方法
	}

}
```

<font color='red'>注意</font>:在Vase构造器中再一次调用getRose()，并没有生成一个新的实例。因为getRose()方法上面使用了@Bean注解,Spring会拦截对他的所用调用，并确保直接返回该方法所创建的bean。==默认情况下下，Spring的bean都是单例模式的==。

- 方法二：带参的方法

```java
@Bean 
Vase getVase3(Flower flower) {
		return new Vase(flower);
	}
```

&emsp;&emsp;在这里，getVase3()方法请求一个Flower作为参数。当Spring调用getVase3()创建Vase bean的时候，它会自动装配一个flower到配置方法之中。然后，方法体就可以按照合适的方式来使用它。==通过这种方式引用其他的bean通常是最佳的选择，因为它不会要求
将Flower声明到同一个配置类之中。==

- 方法3：setter方法

```java
public Vase getVase1() {
		Vase vase=new Vase();
		vase.setFlower(getRose());
		return vase;
	}
```

**==再次强调一遍，带有@Bean注解的方法可以采用任何必要的Java功能来产生bean实例。构造器和Setter方法只是@Bean方法的两个简单样例。这里所存在的可能性仅仅受到Java语言的限制。==**

## 4 通过XML装配bean

> 我希望本节的内容只是用来帮助你维护已有的XML配置，在完成新的Spring工作时，希望你会使用自动化配置和JavaConfig。

### 4.1 创建XML配置规范

在使用XML为Spring装配bean之前，你需要创建一个新的配置规范。在使用JavaConfig的时候，这意味着要创建一个带有@Configuration注解的类，而在XML配置中，这意味着要创建一个XML文件，并且要以<beans>元素为根。比如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans   
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd   
    http://www.springframework.org/schema/tx   
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd   
    http://www.springframework.org/schema/aop    
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
</beans>
```

![1545047734206](..\..\..\..\img\1545047734206.png)

### 4.2 声明一个简单的bean

```xml
<bean class="Bean.Dog" />
```

&emsp;&emsp;这个简单的bean是通过class属性来指定的。==并且要全限定的类名。==在这个bean中，没有设定ID,因此系统会将它命名为“Bean.Dog#0”,如果声明了另一个，则为“Bean.Dog#1”。

&emsp;&emsp;如果想要用自定义的命名，则用id属性设置,对象的引用都是使用ID去获取的。

```xml
<bean id="dog" class="Bean.Dog" />
```

**注意：**

Spring 发现上面这个dog Bean的时候，会调用它的默认构造器来创建bean，而不要自己去创建一个实例，但是在JavaConfig中，你可以随意的按照你的想法去创建bean。

### 4.3 借助构造器注入初始化bean

&emsp;&emsp;在Spring XML配置中，只有一种声明bean的方式：使用<bean>元素并指定class属性。Spring会从这里获取必要的信息来创建bean。

&emsp;&emsp;但是，在XML中声明DI时，会有多种可选的配置方案和风格。具体到构造器注入，有两种基本的配置方案可供选择：

- <constructor-arg>元素
- 使用Spring 3.0所引入的c-命名空间

两者的区别在很大程度就是是否冗长烦琐。，<constructor-arg>元素比使用c-命名空间会更加冗长，从而导致XML更加难以读懂。<font color='red'>另外，有些事情<constructor-arg>可以做到，但是使用c-命名空间却无法实现。</font>

#### 4.3.1注入引用

##### 4.3.1.1使用< constructor-arg>元素注入引用

```xml
<bean id="dog" class="Bean.Dog" />

<bean id="family" class="Bean.Family">
    	<constructor-arg ref="dog" />  //将ID为dog的Bean注入到构造器中
</bean>
```

##### 4.3.2.1使用c-命名空间注入引用

要使用它的话，必须要在XML的顶部声明其模式，如下所示：

```xml
xmlns:c="http://www.springframework.org/schema/c"	
```

```xml
<bean id="rose" class="Java显式配置.Rose" />

<bean id="vase" class="Java显式配置.Vase" c:flower-ref="rose" />
```

&emsp;&emsp;在这里，我们使用了c-命名空间来声明构造器参数，它作为<bean>元素的一个属性，不过这个属性的名字有点诡异。图2.1描述了这个属性名是如何组合而成的。

![1545098251165](..\..\..\..\img\1545098251165.png)

&emsp;&emsp;属性名以“c:”开头，也就是命名空间的前缀。接下来就是要装配的**构造器参数名**，在此之后是“-ref”，这是一个命名的约定，它会告诉Spring，正在装配的是一个bean的引用，这个bean的名字
是compactDisc，而不是字面量“compactDisc”。

&emsp;&emsp;在编写前面的样例时，关于c-命名空间，有一件让我感到困扰的事情就是它直接引用了构造器参数的名称。引用参数的名称看起来有些怪异，因为这需要在编译代码的时候，将调试标志（debug symbol）保存在类代码中。如果你优化构建过程，将调试标志移除掉，那么这种方式可能就无法正常执行了。

替代的方案是我们使用参数在整个参数列表中的位置信息：

![1545098752849](..\..\..\..\img\1545098752849.png)

#### 4.3.2 将字面量(常量)注入到构造器中

**先创建一个POJO:Person**

```java
package Bean;

import java.util.List;

public class Person {
	
	private String name;
	private int age;
	private List<Animal> petList;
	
	public List<Animal> getPetList() {
		return petList;
	}

	public void setPetList(List<Animal> petList) {
		this.petList = petList;
	}

	public Person() {
		
	}
	
	public Person(String name,int age) {
		this.name=name;
		this.age=age;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

}

```

注入name：Lucy,age:20

##### 4.3.2.1 使用< constructor-arg>元素

```xml
<bean id="fanxin" class="Bean.Person">
		<constructor-arg value="Fanxin" />
		<constructor-arg value="20"/>
	</bean>
```

&emsp;&emsp;我们再次使用<constructor-arg>元素进行构造器参数的注入。但是这一次我们没有使用**“ref”**属性来引用其他的bean，而是使用了**value**属性，通过该属性表明给定的值要以字面量的形式注入到构造器之中。

##### 4.3.2.2使用 c-命名空间

```xml
<bean id="Uzi" 
	class="Bean.Person" 
	c:name="Uzi" 
	c:age="20" />
```

可以看到，装配字面量与装配引用的区别在于属性名中去掉了“-ref”后缀。与之类似，我们也可以通过参数索引装配相同的字面量值，如下所示：

```xml
<bean id="Uzi" 
		class="Bean.Person" 
		c:_0="Uzi" 
		c:_1="20" />
```

####  4.3.3装配集合    (只有< constructor-arg>能实现)

##### 4.3.3.1  List

![1545101914732](..\..\..\..\img\1545101914732.png)

&emsp;&emsp;其中，<list>元素是<constructor-arg>的子元素，这表明一个包含值的列表将会传递到构造器中。其中，<value>元素用来指定列表中的每个元素。

&emsp;&emsp;与之类似，我们也可以使用<ref>元素替代<value>，实现bean引用列表的装配。例如，假设你有一个Discography类，它的构造器如下所示：

![1545101991761](..\..\..\..\img\1545101991761.png)

##### 4.3.3.2 Set

![1545102053198](..\..\..\..\img\1545102053198.png)

### 4.4 设置属性

&emsp;&emsp;4.3介绍的都是使用构造器注入方式，没有使用属性Setter方法。

&emsp;&emsp;<font color='red'>该选择构造器注入还是属性注入呢？作为一个通用的规则，**我倾向于对强依赖使用构造器注入，而对可选性的依赖使用属性注入。**</font>

#### 4.4.1 使用< property>属性进行注入

```xml
<!-- 使用setter 方法进行属性注入 -->
	
	<bean id="family1" class="Bean.Family">
		<property name="pet" ref="dog"></property>
	</bean>

	<bean id="Zero" class="Bean.Person">
		<property name="name" value="zero" />
		<property name="age" value="21" />
	</bean>
```

&emsp;&emsp;<property>元素为属性的Setter方法所提供的功能与<constructor-arg>元素为构造器所提供的功能是一样的。使用<property>属性的前提是注入的属性一定要有对应的set方法。

#### 4.4.2 使用p- 命名空间属性进行注入

&emsp;&emsp;我们已经知道，Spring为<constructor-arg>元素提供了c-命名空间作为替代方案，与之类似，Spring提供了更加简洁的p-命名空间，作为<property>元素的替代方案。首先加入：

```xml-dtd
xmlns:p="http://www.springframework.org/schema/p"
```

![1545112059414](..\..\..\..\img\1545112059414.png)

```xml
<!-- 使用 p- 命名空间 -->
	<bean id="family2" class="Bean.Family" p:pet-ref="dog"/>
	
	<bean id="MLXG" class="Bean.Person" p:name="MLXG" p:age="22" />
```

&emsp;&emsp;与c-命名空间一样，装配bean引用与装配字面量的唯一区别在于是否带有“-ref”后缀。如果没有“-ref”后缀的话，所装配的就是字面量。

#### 4.4.3 属性注入集合(只能使用< property>

```xml
<bean id="White" class="Bean.Person" p:name="55K" p:age="24" >
		<property name="petList">
			<list>
				<ref bean="dog"/>
				<ref bean="dog"/>
				<ref bean="dog"/>
				<ref bean="dog"/>
			</list>
		</property>
	</bean>
```

如果是基本类型，不是引用对象，则不用ref，用<value>...</value>

### 4.5使用Spring util-命名空间建立集合

我们可以使用Spring util-命名空间中的一些功能来简化.uitl所提供的功能

![1545112650001](..\..\..\..\img\1545112650001.png)

首先添加命名空间

```xml-dtd
xmlns:util="http://www.springframework.org/schema/util"

<!--并且加上-->
http://www.springframework.org/schema/util
http://www.springframework.org/schema/util/spring-util-4.3.xsd
```

util-命名空间所提供的功能之一就是<util:list>元素，它会创建一个列表的bean。借助<util:list>，我们可以将磁道列表转移到persion bean之外，并将其声明到单独的bean之中，如下所示：

```xml
<!-- util 创建集合bean -->
	<util:list id="petList">
		<ref bean="dog" />
		<ref bean="dog" />
		<ref bean="dog" />
		<ref bean="dog" />
	</util:list>
	<!-- 使用bean-petList 创建 jklove Bean -->
	<bean id="jklove" class="Bean.Person" p:name="55K" p:age="24"
		p:petList-ref="petList" />
```

|        元素        |                        描述                        |
| :----------------: | :------------------------------------------------: |
|   util:constant    |  引用某个类型的public static域，并将其暴露为bean   |
|     util:list      | 创建一个java.util.List类型的bean，其中包含值或引用 |
|      util:map      | 创建一个java.util.Map类型的bean，其中包含值或引用  |
|  util:properties   |       创建一个java.util.Properties类型的bean       |
| util:property-path | 引用一个bean的属性（或内嵌属性），并将其暴露为bean |
|      util:set      | 创建一个java.util.Set类型的bean，其中包含值或引用  |

## 5 导入和混合配置

&emsp;&emsp;在典型的Spring应用中，我们可能会同时使用自动化和显式配置。即便你更喜欢通过JavaConfig实现显式配置，但有的时候XML却是最佳的方案。

&emsp;&emsp;关于混合配置，第一件需要了解的事情就是在自动装配时，<font color='red'>它并不在意要装配的bean来自哪里。自动装配的时候会考虑到Spring容器中所有的bean，不管它是在JavaConfig或XML中声明的还是通过组件扫描获取到的。</font>

### 5.1 @Import

```java
package Java显式配置;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import Bean.Flower;
import Bean.Rose;
import Bean.Vase;

@Configuration
public class JavaConfig {
	@Bean(name= {"rose","rose1"})
	public Flower getRose() {
		return new Rose();
	}
	@Bean
	public Vase getVase() {
		return new Vase(getRose());  //调用获取rose bean的方法
	}
	@Bean 
	public Vase getVase1() {
		Vase vase=new Vase();
		vase.setFlower(getRose());
		return vase;
	}
	@Bean Vase getVase3(Flower flower) {
		return new Vase(flower);
	}

}
```

&emsp;&emsp;在上面这个例子中，一个JavaConfig配置代码定义了Flower的bean和Vase的bean。现在想这这两个类型的Bean单独定义，则要对上面的代码进行拆分。

**FlowerJavaConfig**

```java
package Java显式配置;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import Bean.Flower;
import Bean.Rose;

@Configuration
public class FlowerJavaConfig {
	@Bean(name= {"rose","rose1"})
	public Flower getRose() {
		return new Rose();
	}
}
```

**VaseJavaConfig**

```java
package Java显式配置;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import Bean.Flower;
import Bean.Vase;

@Configuration
public class VaseJavaConfig {
	@Bean
	Vase getVase3(Flower flower) {
		return new Vase(flower);
	}
}

```

如果直接运行测试代码，则需要导入这两个配置类，==现在想将这两个类组合起来，则使用@Import注解==。

```java
@Configuration
@Import(FlowerJavaConfig.class)
public class VaseJavaConfig {
	@Bean
	Vase getVase3(Flower flower) {
		return new Vase(flower);
	}
}

```

或者使用一个单独的类，用两个@Import注解将它们组合在一起。

```java
package Java显式配置;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import({FlowerJavaConfig.class,VaseJavaConfig.class})
public class ConfigCenter {

}
```

### 5.2 在JavaConfig中引用XML配置

使用@ImportResource注解，假设Vase使用的是JavaConfig配置，Flower使用的是ApplicationContext.xml配置。

```java
@Configuration
@Import(VaseJavaConfig.class)
@ImportResource("classpath:Spring/ApplicationContext.xml")
public class ConfigCenter {

}
```

两个bean——配置在JavaConfig中的Vase以及配置在XML中Flower——都会被加载到Spring容器之中。因为Vase中带有@Bean注解的方法接受一个flower作为参数，因此Flower将会装配进来，此时与它是通过XML配置的没有任何关系。

## 5.3 在XML配置中引用JavaConfig

&emsp;&emsp;在JavaConfig配置中，我们已经展现了如何使用@Import和@ImportResource来拆分JavaConfig类。在XML中，我们可以使用import元素来拆分XML配置。

![1545116216845](..\..\..\..\img\1545116216845.png)

&emsp;&emsp;< import>元素只能导入其他的XML配置文件，并没有XML元素能够导入JavaConfig类。

&emsp;&emsp;但是，有一个你已经熟知的元素能够用来将Java配置导入到XML配置中：**< bean>**元素。为了将JavaConfig类导入到XML配置中，我们可以这样声明bean：

![1545116376269](..\..\..\..\img\1545116376269.png)

&emsp;&emsp;不管使用JavaConfig还是使用XML进行装配，我通常都会创建一个<font color='red'>根配置（root configuration）</font>，也就是这里展现的这样，这个配置会将两个或更多的装配类和/或XML文件组合起来。我也会在根配置中启用组件扫描（通过<context:component-scan>或@ComponentScan）。你会在本书的很多例子中看到这种技术。

## 6 bean的scope

&emsp;&emsp;Scope描述的是Spring容器如何新建Bean的实例的。Spring的Scope有以下几种，通过@Scope注解实现。

1. Singleton:一个Spring容器只有一个Bean实例，此为Spring的默认配置。全容器共享一个实例。
2. Prototype:每次调用都获取一个新的Bean实例。
3. Requst:Web项目中，给每一个http request新建一个Bean实例。
4. Session:Web项目中，给每一个http session 新建一个Bean实例
5. GlobalSession:这个只在portal应用中有用，给每一个gloabl http session新建一个Bean实例。



## 6 小结

==**Spring框架的核心是Spring容器。容器负责管理应用中组件的生命周期，它会创建这些组件并保证它们的依赖能够得到满足，这样的话，组件才能完成预定的任务。**==

&emsp;&emsp;在本章中，我们看到了在Spring中装配bean的三种主要方式：**自动化配置、基于Java的显式配置以及基于XML的显式配置**。不管你采用什么方式，这些技术都描述了Spring应用中的组件以及这些组件之间的关系。
&emsp;&emsp;**我同时建议尽可能使用自动化配置，**以避免显式配置所带来的维护成本。但是，如果你确实需要显式配置Spring的话，应该优先选择基于Java的配置，它比基于XML的配置更加强大、类型安全并且易于重构。在本书中的例子中，当决定如何装配组件时，我都会遵循这样的指导意见。
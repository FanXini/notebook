# <center>访问权限修饰符</center>

类可以有**public**类或者缺省访问控制级。

使用**public**访问控制修饰符，这个类就是公共的，公共类对任何地方都是可见的。

使用缺省访问控制修饰符，这个类就是缺省访问级别，它只能被同一个包中的其他类访问。但是对于其他包中的类是不可见的。

 

类成员访问控制修饰符

| 访问级别  | 从其他包中类 | 从同一个包中的类 | 从子类 | 从同一个类 |
| --------- | ------------ | ---------------- | ------ | ---------- |
| Public    | 是           | 是               | 是     | 是         |
| Protected | 否           | 是               | 是     | 是         |
| 缺省      | 否           | 是               | 否     | 是         |
| Private   | 否           | 否               | 否     | 否         |

## Private

关键字Private的意思是，出了包含该成员的类之外，其他任何类都无法访问这个成员。

当你可以肯定该成员只是该类中的一个"助手"成员时，你就可以将它设置为Private，以确保不会在包的其他地方误用到它。

事实证明，Private的使用是有多么的重要，在多线程的环境下尤其如此。

示例：

```java
package xin.kaihuoguodian.privilege;

class Dog {
	private Dog() {
		
	}	
	static Dog createDog() {
		return new Dog();
	}
}

public class PrivateTest {
	public static void main(String agrsp[]) {
		Dog dog=Dog.CreateDog();
	}
}

```

该示例中禁止用构造器创建对象，而必须用静态方法createDog创建。

## Protected

&emsp;&emsp;关键字**Protected**修饰符处理的是继承的概念。同一个包中继承的子类，可以访问所有的拥有包访问权限的成员，但是如果是其他包继承的子内，只能访问父类中public修饰的成员，但是父类的创建者希望有某个特定成员，把对它的访问权限只赋予给派生类而不是所有类。这个时候就需要用Protected字段来完成这个工作了，当然，**Protected**也提供包访问权限，也就是说，相同包内的其他内也可以访问**Protected**元素。

举例：

先在**xin.kaihuoguodian.privilege**创建一个Animal类：

```java
package xin.kaihuoguodian.privilege;

public class Animal {
	
	protected String name="Animal";	
	private int level=1;
}

```

再在**xin.kaihuoguodian.permission**创建一个Dog类，继承Animal

```java
package xin.kaihuoguodian.permission;

import xin.kaihuoguodian.privilege.Animal;

public class Dog extends Animal{
	public static void main(String agrs[]) {
		Dog dog=new Dog();
		System.out.println(dog.name);
        //System.out.println(dog.level);  报错不能访问父类中的私有成员
		
	}
}

输出  Animal
```

在**xin.kaohuoguodian.permission**创建一个Robot类，不能访问Animal中的Protected成员。

```java
package xin.kaihuoguodian.permission;

import xin.kaihuoguodian.privilege.Animal;

public class Robot {
	
	public static void main(String args[]) {
		Animal animal=new Animal();
        // System.out.println(Animal.name);  报错
	}
}

```



**Protected 和缺省的区别:**

> Protected允许被其他包中的子类访问，但是缺省不允许，它只允许被同一个包中的类访问。
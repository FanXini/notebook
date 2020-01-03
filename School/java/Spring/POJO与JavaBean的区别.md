# <center>POJO和JavaBean的区别</center>

&emsp;&emsp;POJO 和JavaBean是我们常见的两个关键字，一般容易混淆，POJO全称是Plain Ordinary Java Object / Pure Old Java Object，中文可以翻译成：普通Java类，具有一部分getter/setter方法的那种类就可以称作POJO，但是JavaBean则比 POJO复杂很多， Java Bean 是可复用的组件，对 Java Bean 并没有严格的规范，理论上讲，任何一个 Java 类都可以是一个 Bean 。但通常情况下，由于 Java Bean 是被容器所创建（如 Tomcat) 的，所以 Java Bean 应具有一个无参的构造器，另外，通常 Java Bean 还要实现 Serializable 接口用于实现 Bean 的持久性。 Java Bean 是不能被跨进程访问的。JavaBean是一种组件技术，就好像你做了一个扳子，而这个扳子会在很多地方被拿去用，这个扳子也提供多种功能(你可以拿这个扳子扳、锤、撬等等)，而这个扳子就是一个组件。一般在web应用程序中建立一个数据库的映射对象时，我们只能称它为POJO。POJO(Plain Old Java Object)这个名字用来强调它是一个普通java对象，而不是一个特殊的对象，其主要用来指代那些没有遵从特定的Java对象模型、约定或框架（如EJB）的Java对象。理想地讲，一个POJO是一个不受任何限制的Java对象（除了Java语言规范。



### 错误的认识

POJO是这样的一种“纯粹的”JavaBean，在它里面除了JavaBean规范的方法和属性没有别的东西，即private属性以及对这个属性方法的public的get和set方法。我们会发现这样的JavaBean很“单纯”，它只能装载数据，作为数据存储的载体，而不具有业务逻辑处理的能力。



### 真正的意思

POJO = "Plain Old Java Object"，是MartinFowler等发明的一个术语，用来表示普通的Java对象，不是JavaBean, EntityBean 或者 SessionBean。POJO不担当任何特殊的角色，也不实现任何特殊的Java框架的接口如，[EJB](https://baike.baidu.com/item/EJB)，[JDBC](https://baike.baidu.com/item/JDBC)等等。

即POJO是一个简单的普通的Java对象，它不包含[业务逻辑](https://baike.baidu.com/item/%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91)或持久逻辑等，但不是JavaBean、EntityBean等，**不具有任何特殊角色和不继承或不实现任何其它Java框架的类或接口。**

下面是摘自Martin Fowler个人网站的一句话：

"We wondered why people were so against using regular objects in their systems and concluded that it was because simple objects lacked a fancy name. So we gave them one, and it's caught on very nicely."－－Martin Fowler

我们疑惑为什么人们不喜欢在他们的系统中使用普通的对象，我们得到的结论是——普通的对象缺少一个响亮的名字，因此我们给它们起了一个，并且取得了很好的效果。——Martin Fowler
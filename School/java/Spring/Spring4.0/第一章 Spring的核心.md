---
typora-copy-images-to: ..\..\img
---

# Spring的核心

纵览全书，读者会发现Spring 可以做非常多的事情。但归根结底，支 撑Spring的仅仅是少许的基本理念，所有的理念都可以追溯到Spring 最根本的使命上：简化Java开发。

为了降低Java开发的复杂性，Spring采取了以下4种**关键策略**：

- **基于POJO的轻量级和最小侵入性编程；**
- **通过依赖注入和面向接口实现松耦合；**
- **基于切面和惯例进行声明式编程；**
- **通过切面和模板减少样板式代码。**

## Spring容器

在基于Spring的应用中，你的应用对象生存于Spring容器（container）中。如图1.4所示，**<font color='red'>Spring容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡（在这里，可能就是new到finalize()）</font>**

![1542712053622](..\..\..\..\img\1542712053622.png)

&emsp;&emsp;Spring容器并不是只有一个。Spring自带了多个容器实现，可以归为两种不同的类型。bean工厂（由org.springframework. beans.factory.BeanFactory接口定义）是最简单的容器，提供基本的DI
支持。应用上下文（由org.springframework.context.ApplicationContext接口定义）基于BeanFactory构建，并提供应用框架级别的服务，例如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者。

&emsp;&emsp;虽然我们可以在bean工厂和应用上下文之间任选一种，但bean工厂对大多数应用来说往往太低级了，因此，<font color="red">应用上下文</font>要比bean工厂更受欢迎。我们会把精力集中在应用上下文的使用上，不再浪费时间讨论bean工厂。

### 使用应用上下文

Spring自带了多种类型的应用上下文。下面罗列的几个是你最有可能遇到的。

- AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。
- AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载Spring Web应用上下文。
- ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
- FileSystemXmlapplicationcontext：从文件系统下的一个或多个XML配置文件中加载上下文定义。
- XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。

### bean的生命周期

下图展示了bean装载到Spring应用上下文中的一个典型的生命周期过程。

![1542712895264](..\..\..\..\img\1542712895264.png)图1.5　bean在Spring容器中从创建到销毁经历了若干阶段，每一阶段都可以针对Spring如何管理bean进行个性化定制正如你所见，在bean准备就绪之前，bean工厂执行了若干启动步骤。
我们对图1.5进行详细描述：
1．Spring对bean进行实例化；
2．Spring将值和bean的引用注入到bean对应的属性中；
3．如果bean实现了BeanNameAware接口，Spring将bean的ID传递给setBean-Name()方法；
4．如果bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入；
5．如果bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传入进来；
6．如果bean实现了BeanPostProcessor接口，Spring将调用它们postProcessBeforeInitialization()方法；
7．如果bean实现了InitializingBean接口，Spring将调用它们的after-PropertiesSet()方法。类似地，如果bean使用init_method声明了初始化方法，该方法也会被调用；
8．如果bean实现了BeanPostProcessor接口，Spring将调用它们的post-ProcessAfterInitialization()方法；
9．此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；
10．如果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用。

&emsp;&emsp;现在你已经了解了如何创建和加载一个Spring容器。但是一个空的容器并没有太大的价值，在你把东西放进去之前，它里面什么都没有。为了从Spring的DI中受益，我们必须将应用对象装配进Spring容器中。我们将在第2章对bean装配进行更详细的探讨。

### Spring风景线

![1545024895720](..\..\..\..\img\1545024895720.png)

![1542714150981](..\..\..\..\img\1542714150981.png)

### 小结

&emsp;&emsp;现在，你应该对Spring的功能特性有了一个清晰的认识。Spring致力于简化企业级Java开发，促进代码的松散耦合。成功的关键在于依赖注入和AOP。

&emsp;&emsp;在本章，我们先体验了Spring的DI。DI是组装应用对象的一种方式，借助这种方式对象无需知道依赖来自何处或者依赖的实现方式。不同于自己获取依赖对象，对象会在运行期赋予它们所依赖的对象。依赖对象通常会通过接口了解所注入的对象，这样的话就能确保低耦合。

&emsp;&emsp;除了DI，我们还简单介绍了Spring对AOP的支持。AOP可以帮助应用将散落在各处的逻辑汇集于一处——切面。当Spring装配bean的时候，这些切面能够在运行期编织起来，这样就能非常有效地赋予bean新的行为。

&emsp;&emsp;依赖注入和AOP是Spring框架最核心的部分，因此只有理解了如何应用Spring最关键的功能，你才有能力使用Spring框架的其他功能。在本章，我们只是触及了Spring DI和AOP特性的皮毛。在以后的几章，我们将深入探讨DI和AOP。
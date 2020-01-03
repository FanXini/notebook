---
typora-copy-images-to: ..\..\img
---

# spring容器初始化源代码解析

**Spring自带了多种类型的IOC容器(应用上下文)。下面罗列的几个是你最有可能遇到的。**

- AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。
- AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载Spring Web应用上下文。
- ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
- FileSystemXmlapplicationcontext：从文件系统下的一个或多个XML配置文件中加载上下文定义。
- XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。

下面，以$AnnotationConfigApplicationContext$为例子，解析容器初始化流程

首先，看$AnnotationConfigApplicationContext$的继承链

![AnnotationConfigApplicationContext](..\..\..\..\img\AnnotationConfigApplicationContext-1561525170812.png)

从该继承体系可以看出：

1. BeanFactory 是一个 bean 工厂的最基本定义，里面包含了一个 bean 工厂的几个最基本的方 法，getBean(…) 、 containsBean(…) 等 ,是一个很纯粹的bean工厂，不关注资源、资源位置、事件等。 ApplicationContext 是一个容器的最基本接口定义，它继承了 BeanFactory, 拥有工厂的基本方法。同时继承了 ApplicationEventPublisher 、 MessageSource ResourcePatternResolver 等接口， 使其 定义了一些额外的功能，如资源、事件等这些额外的功能。
2. AbstractBeanFactory 和 AbstractAutowireCapableBeanFactory 是两个模 板抽象工厂类。AbstractBeanFactory 提供了 bean 工厂的抽象基类，同时提供 了 ConfigurableBeanFactory 的完整实现。AbstractAutowireCapableBeanFactory 是继承 了 AbstractBeanFactory 的抽象工厂，**里面提供了 bean 创建的支持**，**包括 bean 的创建、依赖注入、检查等等功能，是一个 核心的 bean 工厂基类。**



## 2 源码分析

本文主要记录Spring容器创建 源码分析过程。

首先贴上一张流程图![AnnotationConfigApplicationContext(..\..\..\..\MarkDown\img\AnnotationConfigApplicationContext(AppConfig.class).png)](E:\Users\fanxin\Desktop\AnnotationConfigApplicationContext(AppConfig.class).png)

接下来，贴上一份测试代码，这里使用AnnotationConfigApplicationContext来初始化Spring容器

**测试代码**

```java
	@Test
	public void test1() {
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		System.out.println("spring容器初始化结束");
		Person person = (Person) ctx.getBean("person");
		System.out.println(person.toString());
	}

```

**配置类**

```java
@Configuration
public class AppConfig {

	@Bean(value="person")
	public Person getPerson() {
		Person person = new Person();
		person.setName("zhangsan");
		return person;
	}
}

```

**Person.java**

```java
public class Person {

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	@Override
	public String toString() {
		return "Person [name=" + name + "]";
	}
}
```



代码看完了，直接进入正题，开始Debug。

## 2.1 构造器方法

Debug从AnnotationConfigApplicationContext的构造器方法开始，找到该构造方法。
![在这里插入图片描述](https://img-blog.csdn.net/20181011095752836?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
往下继续走，进入到AbstractApplicationContext的refresh()方法，Spring容器的初始化过程就在该方法中完成的。
本文会进入每个方法，看看方法里面的代码，但可能不会很细。
![在这里插入图片描述](https://img-blog.csdn.net/20181011101058246?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![在这里插入图片描述](https://img-blog.csdn.net/20181011101842606?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2.2 prepareRefresh()

1、接下来，继续Debug，进入第一个方法prepareRefresh(),该方法主要是进行刷新前的预处理操作：记录容器启动事件和标记

![在这里插入图片描述](https://img-blog.csdn.net/20181011102108344?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2.3   obtainFreshBeanFactory()

进入第二个方法ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory()，该方法主要是获取beanFactory。

![在这里插入图片描述](https://img-blog.csdn.net/20181011102332872?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
进入refreshBeanFactory()，会跳到GenericApplicationContext的refreshBeanFactory()方法。这里讲一下**GenericApplicationContext有个构造方法，会new一个新的$DefaultListableBeanFactory$。**DefaultListableBeanFactory才是真正的bean工厂，AnnotationConfigApplicationContext其实是个代理类
![在这里插入图片描述](https://img-blog.csdn.net/20181011102544476?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
而进入refreshBeanFactory()先当于new一个DefaultListableBeanFactory，并设置一个序列化值。
![在这里插入图片描述](https://img-blog.csdn.net/20181011102658476?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

返回接在再看getBeanFactory()方法，跳到GenericApplicationContext的getBeanFactory(),返回上一步创建的DefaultListableBeanFactory
![在这里插入图片描述](https://img-blog.csdn.net/20181011102807251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
最后将该beanFactory(DefaultListableBeanFactory)返回。

## 2.4 prepareBeanFactory(beanFactory)

接着进入第三个方法prepareBeanFactory(beanFactory)，该方法主要是beanFactory的预准备工作,也就是对beanFactory进行一些初始化后的设置;

1. 设置BeanFactory的类加载器、支持表达式解析器…

   ```java
   beanFactory.setBeanClassLoader(getClassLoader());
   beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader())); //1)
   beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
   ```

2. 添加部分BeanPostProcessor（ApplicationContextAwareProcessor）

   ```java
   beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this)); // 2)
   ```

3. 设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware等；

   ```java
   beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
   		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
   		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
   		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
   beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
   		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
   ```

4. 注册可以解析的自动装配，我们能直接在任何组件中自动注入：
   BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext

   ```java
   beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
   beanFactory.registerResolvableDependency(ResourceLoader.class, this);	beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
   beanFactory.registerResolvableDependency(ApplicationContext.class, this);
   ```

5. 添加BeanPostProcessor【ApplicationListenerDetector】

   ```java
   beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
   ```

6. 添加编译时的AspectJ；

   ```java
   if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
   	beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
   	// Set a temporary ClassLoader for type matching.
   	beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   }
   ```

7. 往BeanFactory中注册组件；environment、	systemProperties、systemEnvironment。

   ```java
   if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
   		beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
   		}
   if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
   		beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
   }
   if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
   		beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
   }
   ```

## 2.5 postProcessBeanFactory(beanFactory)

接着往下走,进入第4个方法postProcessBeanFactory(beanFactory)，发现该方法是交给子类重写的,子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置

```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {}
1
```

--------------------------------以上是BeanFactory的创建及预准备工作-----------------------------

-------

## 2.6  invokeBeanFactoryPostProcessors(beanFactory)

继续进入第5个方法invokeBeanFactoryPostProcessors(beanFactory)，该方法主要是调用beanFactory的后置处理器，在BeanFactory标准初始化之后执行的。`PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());`这个方法的第二个参数是获取到的Spring容器里的BeanFactoryPostProcessors对象集合。**接下来就是今天的核心方法了：**

```java
public static void invokeBeanFactoryPostProcessors(
            ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

        // Invoke BeanDefinitionRegistryPostProcessors first, if any.
        Set<String> processedBeans = new HashSet<String>();
（1）第一大部分
    /**
    1. 判断beanFactory是否继承了BeanDefinitionRegistry类
	2. 调用  invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);方法注册@Component修饰类的beanDefinition.
	3. 获取BeanDefinitionRegistryPostProcessor的集合
	4. 将集合的后置器分类，然后各自分顺序执行。
    **/
        if (beanFactory instanceof BeanDefinitionRegistry) {
            BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
            List<BeanFactoryPostProcessor> regularPostProcessors = new LinkedList<BeanFactoryPostProcessor>();
            List<BeanDefinitionRegistryPostProcessor> registryPostProcessors =
                    new LinkedList<BeanDefinitionRegistryPostProcessor>();

            for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
                if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
                    BeanDefinitionRegistryPostProcessor registryPostProcessor =
                            (BeanDefinitionRegistryPostProcessor) postProcessor;
                    registryPostProcessor.postProcessBeanDefinitionRegistry(registry);
                    registryPostProcessors.add(registryPostProcessor);
                }
                else {
                    regularPostProcessors.add(postProcessor);
                }
            }

            // Do not initialize FactoryBeans here: We need to leave all regular beans
            // uninitialized to let the bean factory post-processors apply to them!
            // Separate between BeanDefinitionRegistryPostProcessors that implement
            // PriorityOrdered, Ordered, and the rest.
            String[] postProcessorNames =
                    beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);

            // First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
            List<BeanDefinitionRegistryPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanDefinitionRegistryPostProcessor>();
            for (String ppName : postProcessorNames) {
                if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                    priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                }
            }
            sortPostProcessors(beanFactory, priorityOrderedPostProcessors);
            registryPostProcessors.addAll(priorityOrderedPostProcessors);
            invokeBeanDefinitionRegistryPostProcessors(priorityOrderedPostProcessors, registry);

            // Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            List<BeanDefinitionRegistryPostProcessor> orderedPostProcessors = new ArrayList<BeanDefinitionRegistryPostProcessor>();
            for (String ppName : postProcessorNames) {
                if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
                    orderedPostProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                }
            }
            sortPostProcessors(beanFactory, orderedPostProcessors);
            registryPostProcessors.addAll(orderedPostProcessors);
            invokeBeanDefinitionRegistryPostProcessors(orderedPostProcessors, registry);

            // Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
            boolean reiterate = true;
            while (reiterate) {
                reiterate = false;
                postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
                for (String ppName : postProcessorNames) {
                    if (!processedBeans.contains(ppName)) {
                        BeanDefinitionRegistryPostProcessor pp = beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class);
                        registryPostProcessors.add(pp);
                        processedBeans.add(ppName);
                        pp.postProcessBeanDefinitionRegistry(registry);
                        reiterate = true;
                    }
                }
            }

            // Now, invoke the postProcessBeanFactory callback of all processors handled so far.
            invokeBeanFactoryPostProcessors(registryPostProcessors, beanFactory);
            invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
        }

        else {
            // Invoke factory processors registered with the context instance.
            invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
        }
（2）第二大部分，我们要分析的主体
        // Do not initialize FactoryBeans here: We need to leave all regular beans
        // uninitialized to let the bean factory post-processors apply to them!
//获取所有继承了BeanFactoryPostProcessor接口的beanNames
        String[] postProcessorNames =
                beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

        // Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
        // Ordered, and the rest.
        List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
        List<String> orderedPostProcessorNames = new ArrayList<String>();
        List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
//把所有的BeanFactoryPostProcessors进行分类
        for (String ppName : postProcessorNames) {
            if (processedBeans.contains(ppName)) {
                // skip - already processed in first phase above
            }
            else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
            }
            else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
                orderedPostProcessorNames.add(ppName);
            }
            else {
                nonOrderedPostProcessorNames.add(ppName);
            }
        }
按顺序执行不同类的后置器
        // First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
        sortPostProcessors(beanFactory, priorityOrderedPostProcessors);
        invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

        // Next, invoke the BeanFactoryPostProcessors that implement Ordered.
        List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
        for (String postProcessorName : orderedPostProcessorNames) {
            orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
        }
        sortPostProcessors(beanFactory, orderedPostProcessors);
        invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

        // Finally, invoke all other BeanFactoryPostProcessors.
        List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
        for (String postProcessorName : nonOrderedPostProcessorNames) {
            nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
        }
        invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

        // Clear cached merged bean definitions since the post-processors might have
        // modified the original metadata, e.g. replacing placeholders in values...
        beanFactory.clearMetadataCache();
    }
```

上面代码中，总共标注了两大部分。

**第一部分：**

> 1. 判断beanFactory是否继承了BeanDefinitionRegistry类
>
> 2. 调用invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);方法注册@Component修饰类的beanDefinition.
>
> 3. 获取BeanDefinitionRegistryPostProcessor的集合
> 4. 将集合的后置器分类，然后各自分顺序执行。

这里我们主要跟踪一下AppConfig.class中person bean的生成。看下图，继续进入跟踪

![在这里插入图片描述](https://img-blog.csdn.net/20180926144604482?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在processConfigBeanDefinitions(BeanDefinitionRegistry registry)中找到下代码，该代码就是加载person bean定义的。往后就不继续了。

```java
parser.parse(candidates); //扫描配置类@ComponentScan中的basebackpage,得到所有的bean定义
parser.validate();

Set<ConfigurationClass> configClasses = new LinkedHashSet <(parser.getConfigurationClasses());  //获取bean的config class
configClasses.removeAll(alreadyParsed);

			// Read the model and create bean definitions based on its content
if (this.reader == null) {
	this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.resourceLoader,    this.environment,this.importBeanNameGenerator, parser.getImportRegistry());
}
this.reader.loadBeanDefinitions(configClasses); //加载bean 定义
```



执行完这句代码，可以看一下beanFactory中的beanDefinitionMap属性，已经多了一个person bean 定义
![在这里插入图片描述](https://img-blog.csdn.net/2018092614572565?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**第二部分也就是执行我们定义的后置器了 (大家可以对照源码里的注释来看下面的步骤)：**

> 1.通过**beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);**方法获取beanFactory里继承了BeanFactoryPostProcessor接口的name的集合；
>  2.把这些BeanFactoryPostProcessors进行分类，源码如下

```java
for (String ppName : postProcessorNames) {
            if (processedBeans.contains(ppName)) {
                // skip - already processed in first phase above
            }
            else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
            }
            else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
                orderedPostProcessorNames.add(ppName);
            }
            else {
                nonOrderedPostProcessorNames.add(ppName);
            }
}
```

> 3.把后置器beans分为PriorityOrdered、Ordered、nonOrdered三大类，前两类是增加了排序条件的后置器；
>  4.前两类后置器执行`sortPostProcessors`和`invokeBeanFactoryPostProcessors`方法，**也就是先执行排序方法，后执行invoke方法。**
>  5.最后一类也就是我们例子中定义的，直接执行**invoke**就可以。

到这里我们已经了解了refresh()中的**invokeBeanFactoryPostProcessors(beanFactory);**方法，**它的作用是注册并执行我们在bean.xml里定义的BeanFactoryPostProcessors，这里也就解释了为什么Spring初始化先执行BeanFactoryPostProcessors后执行构造函数、init-method等其它方法了。**
 后续我们继续跟进spring的初始化源码。

**考虑到篇幅，以下几个步骤就不展示代码，只说一下方法里面的操作。**

## 2.7 registerBeanPostProcessors(beanFactory)

该方法主要往beanFactory注册Bean的后置处理器。不同接口类型的BeanPostProcessor，在Bean创建前后的执行时机是不一样的。有以下几个步骤：
一、获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
二、先注册PriorityOrdered优先级接口的BeanPostProcessor；
把每一个BeanPostProcessor；添加到BeanFactory中
beanFactory.addBeanPostProcessor(postProcessor);
三、再注册Ordered接口的
四、 最后注册没有实现任何优先级接口的
五、最终注册MergedBeanDefinitionPostProcessor；
六、注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是
applicationContext.addApplicationListener((ApplicationListener<?>) bean);

可以看到我们`registerBeanPostProcessors`就是我们要解析的方法，含义是**注册BeanPostProcessors，并没有和BeanFactoryPostProcessor的注册过程一样，注册完就执行，这里只是注册了BeanPostProcessors并没有invoke。**
 我们跟进`registerBeanPostProcessors(beanFactory);`

```java
/**
     * Instantiate and invoke all registered BeanFactoryPostProcessor beans,
     * respecting explicit order if given.
     * <p>Must be called before singleton instantiation.
     */
    protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
    }

    /**
     * Instantiate and invoke all registered BeanPostProcessor beans,
     * respecting explicit order if given.
     * <p>Must be called before any instantiation of application beans.
     */
    protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
    }
```

这里我留了两个方法，**第一个方法是上篇的BeanFactoryPostProcessor的执行方法，第二个是BeanPostProcessor的执行过程，这两个后置器都是通过PostProcessorRegistrationDelegate这个委托类来实现各自的过程的，**我们打开这个委托类就会看到我们上篇的核心方法，当然也有本篇的核心方法，分别是

```java
public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors)
public static void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) 
```

我们今天分析`registerBeanPostProcessors`，这个方法的入参没有**后置器集合**，而是把`AbstractApplicationContext` 实例作为参数传了进去

```java
public static void registerBeanPostProcessors(
            ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
第一步
        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

        // Register BeanPostProcessorChecker that logs an info message when
        // a bean is created during BeanPostProcessor instantiation, i.e. when
        // a bean is not eligible for getting processed by all BeanPostProcessors.
        int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
        beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

        // Separate between BeanPostProcessors that implement PriorityOrdered,
        // Ordered, and the rest.
        List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
        List<BeanPostProcessor> internalPostProcessors = new ArrayList<BeanPostProcessor>();
        List<String> orderedPostProcessorNames = new ArrayList<String>();
        List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
第二步
        for (String ppName : postProcessorNames) {
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
                priorityOrderedPostProcessors.add(pp);
                if (pp instanceof MergedBeanDefinitionPostProcessor) {
                    internalPostProcessors.add(pp);
                }
            }
            else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
                orderedPostProcessorNames.add(ppName);
            }
            else {
                nonOrderedPostProcessorNames.add(ppName);
            }
        }
第三步
        // First, register the BeanPostProcessors that implement PriorityOrdered.
        sortPostProcessors(beanFactory, priorityOrderedPostProcessors);
        registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

        // Next, register the BeanPostProcessors that implement Ordered.
        List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
        for (String ppName : orderedPostProcessorNames) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            orderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        sortPostProcessors(beanFactory, orderedPostProcessors);
        registerBeanPostProcessors(beanFactory, orderedPostProcessors);

        // Now, register all regular BeanPostProcessors.
        List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
        for (String ppName : nonOrderedPostProcessorNames) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            nonOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

        // Finally, re-register all internal BeanPostProcessors.
        sortPostProcessors(beanFactory, internalPostProcessors);
        registerBeanPostProcessors(beanFactory, internalPostProcessors);
第四步
        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
    }
```

我在源码里标注了四个步骤，接下里分别解释：

> 1.通过**beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);**方法获取beanFactory里继承了BeanPostProcessor接口的name的集合；
>  2.把后置器beans分为PriorityOrdered、Ordered、nonOrdered三大类，前两类是增加了排序条件的后置器；
>  3.前两类后置器执行`sortPostProcessors`和`registerBeanPostProcessors`方法，**也就是先执行排序方法，后执行注册方法。**
>  4.最后一步用到了上面提到的`BeanPostProcessor`和`BeanFactoryPostProcessor`的入参不同的`AbstractApplicationContext`，**这一步执行了什么呢？**

```java
public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {
        Assert.notNull(beanPostProcessor, "BeanPostProcessor must not be null");
        this.beanPostProcessors.remove(beanPostProcessor);
        this.beanPostProcessors.add(beanPostProcessor);
        if (beanPostProcessor instanceof InstantiationAwareBeanPostProcessor) {
            this.hasInstantiationAwareBeanPostProcessors = true;
        }
        if (beanPostProcessor instanceof DestructionAwareBeanPostProcessor) {
            this.hasDestructionAwareBeanPostProcessors = true;
        }
    }
```

可以看到，在addBeanPostProcessor方法里把BeanPostProcessor注册进了AbstractBeanFactory，这也就是**为什么BeanFactoryPostProcessor执行了后置接口实现类，而BeanPostProcessor仅仅执行了注册，而没有执行的原因。**

## 2.8 initMessageSource()。

该方法主要初始化MessageSource组件（国际化功能；消息绑定，消息解析）。有以下几个步骤：
一、 获取BeanFactory
二、看容器中是否有id为messageSource的，类型是MessageSource的组件
如果有赋值给messageSource，如果没有自己创建一个DelegatingMessageSource；
MessageSource：取出国际化配置文件中的某个key的值；能按照区域信息获取；
三、 把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource；
beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);

## 2.9 initApplicationEventMulticaster();

该方法主要初始化事件派发器；有以下几个步骤：
一、获取BeanFactory
二、 从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster；
三、如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
四、 将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入

## 2.10 onRefresh()

该方法主要在容器刷新的时候可以自定义逻辑，留给子类实现。

## 2.11 registerListeners()

该方法主要在容器中将所有项目里面的ApplicationListener注册进来。有以下几个步骤：
一、从容器中拿到所有的ApplicationListener
二、 将每个监听器添加到事件派发器中；
getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
三、派发之前步骤产生的事件；

## 2.12 finishBeanFactoryInitialization(beanFactory);

接下来重点看一下这个方法finishBeanFactoryInitialization(beanFactory);该方法作用是实例化所有剩下的懒加载单实例。
![在这里插入图片描述](https://img-blog.csdn.net/20180927104235785?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
重点看一下下面这段代码

```java
	@Override
	public void preInstantiateSingletons() throws BeansException {
		if (this.logger.isDebugEnabled()) {
			this.logger.debug("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		// 获取容器中的所有Bean定义名字
		List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			1.先获取bean定义
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			2.判断Bean是不是抽象的，是不是单实例的，是不是懒加载
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				 判断是否是FactoryBean；
				if (isFactoryBean(beanName)) {
					final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
					boolean isEagerInit;
					是否是实现FactoryBean接口的Bean
					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
						isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
							@Override
							public Boolean run() {
								return ((SmartFactoryBean<?>) factory).isEagerInit();
							}
						}, getAccessControlContext());
					}
					else {
						isEagerInit = (factory instanceof SmartFactoryBean &&
								((SmartFactoryBean<?>) factory).isEagerInit());
					}
					if (isEagerInit) {
						getBean(beanName);
					}
				}
				else {
				    //我们重点看一下这个方法。不是工厂bean，通过该方法获取创建bean实例
					getBean(beanName);
				}
			}
		}
}
```

当循环的beanName 为person的时候，我们debug进这个getBean(beanName)方法。
![在这里插入图片描述](https://img-blog.csdn.net/20180927112112718?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
继续debug，进入方法。因为person实例还没有创建，beanFactory中的singletonObjects肯定找不到。一些判断这里就不展示，直接跳过了。找到下面代码,其中this.getSingleton方法第二个参数使用了匿名类，待会会回调该匿名类的getObjetct方法。
![1561637595742](..\..\..\..\img\1561637595742.png)

进入`this.getSingleton(beanName, new ObjectFactory<Object>()`方法，找到

![1561638045907](..\..\..\..\img\1561638045907.png)

进入`singletonFactory.getObject();`方法，也就是上面提到的匿名类实现的getObject方法

![1561638160780](..\..\..\..\img\1561638160780.png)

继续debug，进入createBean(beanName, mbd, args),找到
![1561638285086](..\..\..\..\img\1561638285086.png)
找到下面代码，进入doCreateBean(beanName, mbdToUse, args);
debug了半天，终于要创建bean实例了。这里Spring容器使用BeanWrappe。BeanWrapper是对Bean的包装，大部分情况下是在spring ioc内部进行使用，用来设置获取被包装的bean对象，获取被包装bean的属性描述器等。
![在这里插入图片描述](https://img-blog.csdn.net/201809271427519?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Debug进入createBeanInstance(beanName, mbd, args);
![1561638407973](..\..\..\..\img\1561638407973.png)
进入debug，进入this.instantiateBean(beanName, mbd)方法，找到箭头指向的方法

![1561638580567](..\..\..\..\img\1561638580567.png)

继续debug，进入。找到图中的代码，箭头标记的方法，就是最后生成bean的方法，通过工具类的静态方法调用生成实例bean，其中constructorToUse是构造器类。
![1561638656686](..\..\..\..\img\1561638656686.png)

利用反射的技术通过构造函数生成实例：

![1561638756965](..\..\..\..\img\1561638756965.png)
一路返回，返回到
AbstractAutowireCapableBeanFactory的 doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)方法。想方便点的话，断点直接打到箭头标记的那行，就不需要一路return回去了。这时候可以看到实例bean的信息。
![在这里插入图片描述](https://img-blog.csdn.net/2018101015423389?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
往下继续走，找到下图的代码，箭头中的方法是对实例bean进行赋值、依赖注入。populateBean里面有赋值之前的一些工厂后置处理器的处理和利用setter方法等为属性进行赋值操作。initializeBean会执行后置处理器的before方法，init_method方法以及后置处理器的after方法。
![1561644041371](..\..\..\..\img\1561644041371.png)
进入到initializeBean(beanName, exposedObject, mbd)方法。在该方法中进行bean的初始化。里面也是一些后置处理器的操作。也就是说，后置处理器的方法在bean创建完成后调用
![1561644547540](..\..\..\..\img\1561644547540.png)

我们进去看看invokeAwareMethods方法，实现aware接口的方法在这里被执行。

![1561644424685](..\..\..\..\img\1561644424685.png)

继续返回出去，来到AbstractBeanFactory的doGetBean方法。
![1561639271935](..\..\..\..\img\1561639271935.png)



## 2.13 finishRefresh()

最后看一下refresh() 方法里面的最后一个方法finishRefresh()。该方法主要完成BeanFactory的初始化创建工作。这样IOC容器就创建完成了。


![在这里插入图片描述](https://img-blog.csdn.net/20181010164258400?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMzQzNzg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**经过上面n多步骤，spring ioc容器初始化过程就结束了**

------

```
Person person = (Person) ctx.getBean("person");
1
```

接下来这个获取person bean就简单了。它的原理就是从beanFactory的singletonObjects(map)中，通过key，获取value了。源码上面也看过了。

--------

学习源码的过程是枯燥的，由于是全英文，可能看着也比较累，但是学习完，感觉收获还是颇多了。由于这个初始化过程步骤比较多，涉及到的东西也比较多，这里看一下，那里看一下，多个类之间切来切去，可能看着也会比较乱一点。但是多看几遍，相信大家也会对这个过程比较了解了。

个人总结：从上面几个步骤可以看出，IOC容器初始化的过程，大部分都是往ioc容器的诸多Map中添加值，方便后续读取使用。读取的时候只需要从Map中根据key取值就可以了。

到此，本篇文章也就结束了。该篇文章主要是记录本人学习spring ioc容器源码的一个过程。中间可能存在一些问题,或有一些不够严谨完善的地方,希望大家体谅体谅。有问题的话,欢迎大家留意,交流交流。

## 总结：初始化顺序

1. 容器预处理
2. 创建并调用工厂后置处理器
3. 创建后置处理器
4. 创建剩下的单例bean，创建执行构造器创建bean,执行setter方法、加载依赖，执行后置处理器之before方法，执行init_method方法，执行后置处理器after方法。

## 参考博客

[Spring IOC容器初始化过程 源码分析](https://blog.csdn.net/u013034378/article/details/82657904)

[Spring中bean工厂后置处理器（BeanFactoryPostProcessor）使用](https://www.jianshu.com/p/b45efc018bcc)

[Spring中bean后置处理器BeanPostProcessor](https://www.jianshu.com/p/f80b77d65d39)

[Spring后置工厂处理器postProcess的源码解析](https://www.jianshu.com/p/f1b2f20981d1)

[Spring后置器postProcess的源码解析](https://www.jianshu.com/p/94bb61bc53d8)


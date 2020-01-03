---
typora-copy-images-to: ..\..\img
---

# <center>第5部分 构建Spring Web应用程序 </center>

## 1. SpringMVC

&emsp;&emsp;SpringMVC是Spring的一个Web框架。SpringMVC基于==模型-视图-控制器（Model-View-Controller，MVC）模式==实现，它能够帮你构建像Spring框架那样灵活和松耦合的Web应用程序。

&emsp;&emsp;你见到过孩子们的捕鼠器游戏吗？这真是一个疯狂的游戏，它的目标是发送一个小钢球，让它经过一系列稀奇古怪的装置，最后触发捕鼠器。小钢球穿过各种复杂的配件，从一个斜坡上滚下来，被跷跷板弹起，绕过一个微型摩天轮，然后被橡胶靴从桶中踢出去。经过这些后，小钢球会对那只可怜又无辜的橡胶老鼠进行捕获。

&emsp;&emsp;乍看上去，你会认为Spring MVC框架与捕鼠器有些类似。Spring将请求在**调度Servlet**、**处理器映射（handler mapping）、控制器以及视图解析器（view resolver）之间移动**，而捕鼠器中的钢球则会在各种斜坡、跷跷板以及摩天轮之间滚动。但是，不要将Spring MVC与捕鼠器游戏做过多比较。每一个Spring MVC中的组件都有特定的目的，并且它也没有那么复杂。

让我们看一下请求是如何从客户端发起，经过Spring MVC中的组件，最终再返回到客户端的。

### 1.1　跟踪Spring MVC的请求

当用户在web页面点击一个链接或者提交表单的时候，一个请求就开始工作了。请求从请求触发到处理完成反馈给用户会经过很多的站点。图5.1展示了请求使用SpringMVC所经历的所有站点。

![1544960933078](..\..\..\..\img\1544960933078.png)

1. 请求离开浏览器，携带用户请求内容的信息(URL或者表单信息)，到达Spring的$DispatcherServlet$。与大多数基于Java的Web框架一样，Spring MVC所有的请求都会通过一个**前端控制器（front  controller）Servlet**。前端控制器是常用的Web应用程序模式，在这里一个==单实例的Servlet==将请求委托给应用程序的其他组件来执行实际的处理。在Spring MVC中，$DispatcherServlet$就是前端控制器。
2. $DispatcherServlet$的任务是将请求发送到SpringMVC控制器**($Controller$)。$Controller$**是一个处理请求的的Spring组件。在一个正常的应用程序中，会存在多个$Controller$以处理不同需求的请求。因此，$DispatcherServlet$需要知道应该将请求发送到哪一个对应的Controller。所以$DispatcherServlet$会查询一个或多个==**处理器映射(Handler Mapping)**==来确定请求的下一站在哪里。处理器映射会根据请求所携带的URL信息进行决策。
3. 当**处理器映射(Handler Mapping)**为请求选择了对应的$Controller$之后，$DispatcherServlet$就会将请求发送到对应的$Controller$上来进去处理。到了$Controller$后，请求会卸下负载(携带的信息)并等待$Controller$进行处理。（实际上，设计良好的控制器本身只处理很少甚至不处理工作，而是将业务逻辑委托给一个或多个服务对象进行处理。）
4. $Controller$在完成请求的业务逻辑后，往往会产生一些信息用来反馈给用户并在浏览器上显示，这些信息称为==**模型($model$)**==。同时要用一些技术将数据格式化后才能展现给用户，一般是HTML格式。因此，信息需要发送给一个用来渲染数据的==**视图($view$),一般会是JSP**。==因此$Controller$最后要做的就是将模型数据打包，并且表示出用于渲染输出的视图名。它接下来会将请求连同模型数据和视图名发送回$DispatcherServlet$。
5. 于是$Controller$不会和特定的视图相耦合，实际上，传递给$DispatcherServlet$的视图名只是一个逻辑名称，它甚至不能用来确定view层就是JSP.$DispatcherServlet$会使用**==视图解析器(view resolver)==**来将逻辑视图名匹配为一个特定的视图实现，它可能是也可能不是JSP。
6. $DispatcherServlet$将模型数据发送给对应的视图。
7. 视图将使用模型数据渲染输出，这个输出会通过响应对象传递给客户端（不会像听上去那样硬编码） 。

&emsp;&emsp;可以看到，请求要经过很多的步骤，最终才能形成返回给客户端的响应。大多数的步骤都是在Spring框架内部完成的，也就是图5.1所示的组件中。尽管本章的主要内容都关注于如何编写控制器，但在此之前我们首先看一下如何搭建Spring MVC的基础组件。

## 2. 搭建SpringMVC

&emsp;&emsp;虽然图5.1看上去SpringMVC有很多重要的组件要配置，得益于Spring新版本的功能增强，现在搭建一个基本可用的SpringMVC变得简单得多了。现在我们用简单的方式配置SpringMVC：所要实现的功能仅限于运行我们所创建的$Controller$控制器。

### 2.1 配置$DispatcherServlet$，继承AbstractAnnotationConfigDispatcherServletInitializer方式

&emsp;&emsp;$DispatcherServlet$是Spring MVC的核心。在这里请求会第一次接触到框架，它要负责将请求路由到其他的组件之中。传统的配置方式里，一般$DispatcherServlet$这样的Servlet会放到web.xml里面。但在这里我们使用Java的配置方式。

程序清单 5.1 配置$DispatcherServlet$

```java
package config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * 
 * @author fancy
 * @description:配置Dispacher和contextListenLoader
 * 这个文件和Web.xml种的文件只能二选一，因为实现了WebApplicationInitializer的类就相当于web.xml
 */
public class MyFirstWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[]{RootConfig.class};
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[]{WebConfig.class};  //指定配置类
	}

	@Override
	protected String[] getServletMappings() { 
		return new String[] {"/"};    //将DispatcherServlet映射到"/"
	} 
}

```

在程序清单5.1中，我们看到处理化类$MyFirstWebAppInitializer$仅仅扩展了$AbstractAnnotationConfigDispatcherServletInitializer$类，并重写了三个方法。==而$AbstractAnnotationConfigDispatcherServletInitializer$会自动地配置$DispatcherServlet$和Spring应用上下文，Spring的应用上下文会位于应用程序的Servlet上下文之中。==

#### 2.1.1 AbstractAnnotationConfigDispatcherServletInitializer剖析

在Servlet 3.0环境中，容器会在类路径中查找实现$javax.servlet.ServletContainerInitializer$接口的类，如果能发现的话，就会用它来配置Servlet容器。

而Spring提供了这个接口的实现，名为$SpringServletContainerInitializer$,而这个类会反过来查找实现$WebApplicationInitializer$的类并将配置的任务交给它们来完成。

而在Spring3.2中引入了一个便利的$WebApplicationInitializer$基础实现，也就是上面我们的扩展类$AbstractAnnotationConfigDispatcherServletInitializer$类。

因为我们创建的$MyFirstWebAppInitializer$扩展了$AbstractAnnotationConfigDispatcherServletInitializer$类，所以也就实现了$WebApplicationInitializer$，因此当部署到Servlet 3.0容器中的时候，容器会自动发现它，并用它来配置Servlet上下文。

&emsp;&emsp;在程序清单5.1中我们重写了三个方法：

- getServletMappings()

  > <font color='red'>这个方法可以设置将什么样的路径请求映射到$DispatcherServlet$上，在本例中，它映射的是“/”,这表示它会是应用的默认Servlet。它会处理应用的所有请求。</font>

&emsp;&emsp;要理解下面这两个方法，我们首先要理解$DispatcherServlet$和一个Servlet监听器（也就是$ContextLoaderListener$）的关系。

&emsp;&emsp;当$DispatcherServlet$启动的时候，它会创建Spring应用上下文，并加载配置文件或配置类中所声明的bean。在程序清单5.1的getServletConfigClasses()方法中，我们要求$DispatcherServlet$加载应用上下文时，使用定义在WebConfig配置类（使用Java配置）中的bean。

&emsp;&emsp;但是在Spring Web应用中，通常还会有另外一个应用上下文。另外的这个应用上下文是由$ContextLoaderListener$创建的。

&emsp;&emsp;我们希望$DispatcherServlet$加载包含Web组件的bean，如控制器、视图解析器以及处理器映射，而$ContextLoaderListener$要加载应用中的其他bean。这些bean通常是驱动应用后端的中间层和数据层组件。

&emsp;&emsp;实际上，$AbstractAnnotationConfigDispatcherServletInitializer$会同时创建$DispatcherServlet$和$ContextLoaderListener$。$getServletConfigClasses()$方法返回的带有@Configuration注解的类将会用来定义$DispatcherServlet$应用上下文中的bean。getRootConfigClasses()方法返回的带有@Configuration注解的类将会用来配置$ContextLoaderListener$创建的应用上下文中的bean。

在本例中，根配置定义在RootConfig中，DispatcherServlet的配置声明在WebConfig中。稍后我们将会看到这两个类的内容。

<font color='red'>注意：这种配置方案只能支持Servlet3.0的服务器，如Tomcat7.0及以上的版本。</font>

- getServletConfigClasses()

  > <font color='red'>要求DispatcherServlet在加载应用上下文时，使用定义的配置类WebConfig。</font>

- getRootConfigClasses()

  > <font color='red'>要求ContextLoaderListener在加载应用上下文时，使用定义的配置类RootConfig。</font>

#### 2.1.2 启动SpringMVC

- XML方式：使用< mvc:annotation-driven>启动注解驱动SpringMVC
- Java方式：@EnableWebMvc注解

#### 2.1.3 WebConfig（Spring MVC配置）

程序清单5.2 最简单的SpringMVC配置

```java
package config;

import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;


@Configurable
@EnableWebMvc

public class WebConfig extends WebMvcConfigurerAdapter{
	

}

```

上面的这个SpringMVC配置文件可以运行起来，但有几个问题：

1. 没有配置视图解析器。Spring默认会使用BeanNameView-Resolver解析视图。
2. 没有启动组建扫描。
3. 这样配置的话，DispatcherServlet会映射为应用的默认Servlet，所以它会处理所有的请求，包括对静态资源的请求，如图片和样式表（在大多数情况下，这可能并不是你想要的效果）

程序清单5.3 最小但可用的SpringMVC配置

```java
package config;

import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Import;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configurable
@EnableWebMvc
@ComponentScan(basePackages= {"Controller"})
public class WebConfig extends WebMvcConfigurerAdapter{
	@Bean
	public ViewResolver viewResolver() { //配置JSP视图解析器
		InternalResourceViewResolver resolver=new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/jsp/");
		resolver.setSuffix(".jsp");
		resolver.setExposeContextBeansAsAttributes(true);
		return resolver;
	}
	@Override     //配置静态资源的处理
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}
}
```

&emsp;&emsp;在这里使用了@ComponentScan注解扫描Controller包中的控制器组件。并且定义了一个jsp视图解析器。最后，新的WebConfig类还扩展了WebMvcConfigurerAdapter并重写了其configureDefaultServletHandling()方法。通过调用DefaultServlet-HandlerConfigurer的enable()方法，我们要求DispatcherServlet将对静态资源的请求转发到Servlet容器中默认的Servlet上，而不是使用DispatcherServlet本身来处理此类请求。

#### 2.1.5 RootConfig

```java
package config;

import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.FilterType;
import org.springframework.context.annotation.Import;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configurable
@Import(RedisConfigByJavaBean.class)
@ComponentScan(basePackages = {"entity"},excludeFilters= {@Filter(type=FilterType.ANNOTATION,value=EnableWebMvc.class)})
public class RootConfig {

}
```

配置完后，将项目add入tomcat中，项目就能正常启动了。

### 2.2 原生继承WebApplicationInitializer方式

#### 2.2.1  配置springMVC

```java
@Configuration
@EnableWebMvc// 1 开启springMVC注解
@ComponentScan("com.wisely.highlight_springmvc4")
public class MyMvcConfig  {

	@Bean //配置视图解析器bean
	public InternalResourceViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		viewResolver.setPrefix("/WEB-INF/views/");
		viewResolver.setSuffix(".jsp");
		viewResolver.setViewClass(JstlView.class);
		return viewResolver;
	}
}
```

#### 2.2.2 Web配置类

```java
package com.wisely.highlight_springmvc4;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration.Dynamic;

import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

public class WebInitializer implements WebApplicationInitializer {//1 //实现了WebApplicationInitializer就相当于web.xml

	@Override
	public void onStartup(ServletContext servletContext)
			throws ServletException {
        //新建WebApplicationContext,注册配置类，并将其和当前servletContext关联
		AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(MyMvcConfig.class);
        ctx.setServletContext(servletContext); 
        
        Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx)); //3 注册SpringMVC的DispatcherServlet
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);

	}
}
```



## 3. SpringMVC的注解

### 3.1 @Controller

&emsp;&emsp;在类的上面使用该注解，则表示该类的实例是一个控制器(用来处理前端请求，相当于之前项目的Action类的作用)。所以在factory.controller中的每一个类，我们都要加上这个@Controller注解。添加了该注解后，如果系统开启了自动扫描，则会加载到容器中成为候选Bean。

![1546599366012](..\..\..\..\img\1546599366012.png)

### 3.2@RequestMapping

&emsp;&emsp;见名知意，@RequestMapping注解就是将前端请求和处理方法进行映射的，它指示了用哪一个类或方法来处理请求动作。

![img](file:///C:/Users/fanxin/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)![img](..\..\..\..\img\clip_image003.jpg)

​       如上图所示，类名(UserLoginController)上面用了两个注解，@Controller和@RequestMapping两个注解，@Controller表明该类是一个处理请求类，@RequestMapping是这个类对应的url前缀，比如某个请求的处理方法为public String loginForm(…),那么前端的请求url则为

![img](..\..\..\..\img\clip_image005.jpg)

如果在jsp中访问，则省略项目名：

![img](..\..\..\..\img\clip_image007.jpg)

其中userLogin，是类的@RequestMapping的value,

private/loging是方法的@RequestMapping的value。

再介绍@RequestMapping常用的两个属性:

| 属性   | 类型         | 是否必要 | 说明                                                   |
| ------ | ------------ | -------- | ------------------------------------------------------ |
| value  | String       | 否       | 用于将指定请求的实际地址映射到方法上                   |
| method | RequestMehod | 否       | 映射指定请求的方法类型，包括GET,POST,HEAD,OPTIONS,PUT… |

使用方法：

![img](..\..\..\..\img\clip_image009.jpg)

如果只用到value一个属性，可以省略属性名，直接输入value的值即可。如

![img](..\..\..\..\img\clip_image011.jpg)

### 3.3  @RequestParam

该注解的作用是将指定的请求参数赋值给方法中的形参，举个例子

![img](..\..\..\..\img\clip_image002-1546599527960.jpg)

在上面我们对两个形参：loginUsername,loginPassword用了@RequestParam注解修饰，再看前端是怎么访问这个方法的：

![img](..\..\..\..\img\clip_image004-1546599527960.jpg)

url中带了两个参数，loginUserName和loginPassword，是不是和上面@RequestParam括号里面的变量名一致？因此，通过用@RequestParam，可以直接将url中的变量映射到方法中的形参变量中来。

还有一种用法就是直接将@RequestParam中的变量名设置成和表单一样，这样提交的时候就会自动将表单的值传递到方法形参中来了。

###  3.4 @PathVariable

这个注解很简单，用于方便获得url的动态参数。举个例子就知道了

![img](..\..\..\..\img\clip_image002-1546599574911.jpg)

@RequestMapping中花括号中的{formName}是一个可以动态变化的值，比如说，我要访问login.jsp页面：

![img](..\..\..\..\img\clip_image004-1546599574911.jpg)

这个时候formName的值就是“login”了

比如说我要访问main.jsp页面

![img](..\..\..\..\img\clip_image006.jpg)

这个时候formName就是“main”了

至于为什么return “login”，就跳转到了login.jsp页面,return”main”就跳转到了main.jsp页面，后面会介绍，这里只介绍@PathVariable的用法。

### 3.5 Model和ModelView

这两个对象很重要，他们两个的作用我理解为他们是连接前端和后端的数据通道。

​       对于MVC框架，控制器(Controller)执行业务逻辑，用于产生模型数据(Model)，而视图(View)用于渲染模型数据。（通俗的讲，就是将数据库查到的数据，映射到model对象上(ORM的功能)，再将model对象的数据显示在Jsp页面上）。

​       如何将模型数据传递给视图是SpringMVC框架的重要工作,SpringMVC提供了以下几种途径输出模型数据。

- Model和ModelMap(ModelMap其实就是Model的Map形式，ModelMap=Model.asMap())

- ModelView

- @ModelAttribute
- @SessionAttributes

先看看@ModelAttribute和Model的用法，被这个注解修饰的方法会在Controller每个方法执行前被执行，因此要慎重使用，它的用法有5种，我这里只介绍两种常用的

#### @ModelAttribute(value=”\***”)修饰方法

   用法实例：

​       先建立一个loginForm1.jsp页面，提交的地址为login1,表单的name为“loginname”

![img](..\..\..\..\img\clip_image002-1546599626671.jpg)

![img](..\..\..\..\img\clip_image004-1546599626671.jpg)

点击提交之后，则应该要调用匹配的方法public String login1()，但是在这个Controller类中，public String userModel1()方法别@ModelAtttribute(“name”)修饰了，因此，在执行login1方法之前，我们要先执行useModel1方法。

![img](..\..\..\..\img\clip_image006-1546599626671.jpg)

首先我们看到useModel1方法中的形参被@RequestParam修饰了，其名字和表单name一致，因此，表单的值就直接传进形参中了,所以此时形参loginname的值就是“teamluo”，然后我们发现方法return的是loginname，它的作用就是在Model里面添加了以@ModelAttribute(“name”)中“name”为key,以形参loginname的值为value的键值对。

​       执行完useModel1方法后，我们才执行请求url对应的login1()方法，这个直接返回的是视图的值“result1”，也就是跳转到了result.jsp。

![img](..\..\..\..\img\clip_image008.jpg)

![img](..\..\..\..\img\clip_image010.jpg)

​       可以看到，在request作用域中，我们访问到了前面在Model中存入的name值

#### @ModelAttribute直接修饰方法

​       使用实例：

​       我们先建立loginForm2.jsp页面，代码如下

![img](..\..\..\..\img\clip_image012.jpg)

![img](..\..\..\..\img\clip_image014.jpg)

点击提交按钮后，同理先进入@ModelAttribute修饰的方法，和之前一样，表单的值直接映射到形参上面来了，但和上面用法不同的是，因为@ModelAttribute后面没有使用括号引出变量名了，因此需要手动添加想存入到Model中的值，所以，userModel2方法中还多了一个参数Model model,通过addAttribute(key,value)方法将loginname和password添加到model容器中。

![img](..\..\..\..\img\clip_image016.jpg)

然后再执行url对应的login2()方法,跳转到了result2.jsp页面中。代码如下

![img](..\..\..\..\img\clip_image018.jpg)

![img](..\..\..\..\img\clip_image020.jpg)

可以发现，在request作用域中我们访问到了存入到Model中的两个变量。

#### ModelView的用法

其实它的用法和model的用法基本一致，只是它多了一个setViewName()的功能，增加对象的函数变为了addObject()了：举个例子(其他代码和之前一样，不再列举了)

![img](..\..\..\..\img\clip_image022.jpg)

​       我们看到ModelView增加了返回的视图路径“result3”，因此return mv后，跳转到了result3.jsp页面

![img](..\..\..\..\img\clip_image024.jpg)

![img](..\..\..\..\img\clip_image026.jpg)

同时这个例子我们可以知道，Model和ModelView中不仅可以增加String对象，还可以增加自己定义的javaBean对象。

### @3.6 SessionAttributes

&emsp;&emsp;这个注解类型允许我们有选择地指定Model中地哪些属性需要转存到HttpSession对象中。且这个注解和@Controller一样，只能声明在类上，不能声明在方法上。

​     使用实例：（之前表单地代码基本一致，不在列举了）

![img](..\..\..\..\img\clip_image002-1546600141292.jpg)

我们发现SessionAttribute类上增加了@SesstionAttributes注解，意味着Model/ModelView中变量名为“user”的变量将会被session，我们在login4方法中将user对象add到ModelView中后，user则被session了，因此，我们可以在result4.jsp中在session作用域中访问到这个User对象。

![img](..\..\..\..\img\clip_image004-1546600141293.jpg)

![img](..\..\..\..\img\clip_image006-1546600141293.jpg)

### 3.7 @RequestBody

这个注解的作用和@RequestParam很相似，但功能要强大很多，最常用的是在传输Json数据上面。

#### 1.它能够自动将前端传递过来的数据转化为对象。

以注册用户为例子：

![img](..\..\..\..\img\clip_image002-1546600187117.jpg)

![img](..\..\..\..\img\clip_image004-1546600187117.jpg)

点击注册按钮后，触发js

![img](..\..\..\..\img\clip_image006-1546600187117.jpg)

我们把data传递到userLogin/register对应的方法public Object registerCount（）方法中，该方法有被@RequestBody注解修饰过的User对象，则该对象会自动根据前端传递过来的data中的值构建这个对象，也就是说此时函数中的user自动被初始化了，并且赋予了前端传过来data中对应的值，是不是很方便？

![img](..\..\..\..\img\clip_image008-1546600187117.jpg)

 

#### 2.第二个用法是往前端传值

这个时候@RequestBody要修饰方法，而不是变量了，还是用上面这个例子

![img](..\..\..\..\img\clip_image010-1546600187117.jpg)

我们构建了一个Map,并直接返回map,这个map就不是返回视图的意思了，而返回的视图默认为请求这个方法的jsp，并把数据返回给这个jsp（其实，这就是Ajax一把刷新的使用方法）。那前端怎么读这个数据呢？，很简单：

![img](..\..\..\..\img\clip_image012-1546600187124.jpg)

直接data.key就行了。
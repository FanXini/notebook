# spring EL 和调用外部资源

Spring El-Spring表达式域语言。支持在XML和注解中使用表达式，类似于JSP的EL表达式

Spring在开发是会遇到调用各种资源的情况，包括普通文件、网址、配置文件、系统环境变量等，我们可以使用Spring的表达式语言实现资源的注入

Spring主要在注解@Value的参数中使用表达式。

本节演示实现以下几种情况：

1. 注入普通字符串
2. 注入操作系统属性
3. 注入表达式运行结果
4. 注入其他Bean的属性
5. 注入文件内容
6. 注入网址内容
7. 注入属性文件

## spring引入

注入配置文件需要使用@PropertySource指定文件地址

1. 在resourse文件夹新建test.properties和test.txt

   test.properties

   ```
   book.author=wangyunfei
   book.name=spring boot
   ```

   test.txt

   ```
   测试文件
   ```

2. 需要被注入的Bean

   ```java
   package com.wisely.highlight_spring4.ch2.el;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Service;
   
   @Service
   public class DemoService {
   	@Value("其他类的属性") //1 注入普通字符串
       private String another;
   
   	public String getAnother() {
   		return another;
   	}
   
   	public void setAnother(String another) {
   		this.another = another;
   	}	
   }
   ```

3. 配置类

   ```java
   package com.wisely.highlight_spring4.ch2.el;
   
   import org.apache.commons.io.IOUtils;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.PropertySource;
   import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
   import org.springframework.core.env.Environment;
   import org.springframework.core.io.Resource;
   
   @Configuration
   @ComponentScan("com.wisely.highlight_spring4.ch2.el")
   @PropertySource("classpath:test.properties")//7使用@PropertySource注解引入文件资源
   public class ElConfig {
   	
   	@Value("I Love You!") //1 注入普通字符串
       private String normal;
   
   	@Value("#{systemProperties['os.name']}") //2 注操作系统属性
   	private String osName;
   	
   	@Value("#{ T(java.lang.Math).random() * 100.0 }") //3注入表达式结果
       private double randomNumber;
   
   	@Value("#{demoService.another}") //4注入其他bean的属性
   	private String fromAnother;
   
   	@Value("classpath:test.txt") //5注入文件资源
   	private Resource testFile;
   
   	@Value("http://www.baidu.com") //6  //注入网址资源
   	private Resource testUrl;
   
   	@Value("${book.name}") //7  注入配置文件
   	private String bookName;
   
   	@Autowired
   	private Environment environment; //7 注入配置文件
   	
   	@Bean //7 
   	public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
   		return new PropertySourcesPlaceholderConfigurer();
   	}
   	
   
   
   	public void outputResource() {
   		try {
   			System.out.println(normal);
   			System.out.println(osName);
   			System.out.println(randomNumber);
   			System.out.println(fromAnother);
   			
   			System.out.println(IOUtils.toString(testFile.getInputStream()));
   			System.out.println(IOUtils.toString(testUrl.getInputStream()));
   			System.out.println(bookName);
               //8 还可以从Environment获取配置属性
   			System.out.println(environment.getProperty("book.author"));
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   
   	}
   
   	
   }
   
   ```

4. 运行

   ```java
   package com.wisely.highlight_spring4.ch2.el;
   
   import org.springframework.context.annotation.AnnotationConfigApplicationContext;
   
   public class Main {
   	
   	public static void main(String[] args) {
   		 AnnotationConfigApplicationContext context =
   	                new AnnotationConfigApplicationContext(ElConfig.class);
   		 
   		 ElConfig resourceService = context.getBean(ElConfig.class);
   		 
   		 resourceService.outputResource();
   		 
   		 context.close();
   	}
   }
   ```

5. 运行结果

   ```java
   I Love You!
   Windows 10
   63.139004452672495
   其他类的属性
   测试文件
   <!DOCTYPE html>
   <!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>鐧惧害涓?涓嬶紝浣犲氨鐭ラ亾</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=鐧惧害涓?涓? class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>鏂伴椈</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>鍦板浘</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>瑙嗛</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>璐村惂</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>鐧诲綍</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">鐧诲綍</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">鏇村浜у搧</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>鍏充簬鐧惧害</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>浣跨敤鐧惧害鍓嶅繀璇?</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>鎰忚鍙嶉</a>&nbsp;浜琁CP璇?030173鍙?&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
   
   spring boot
   wangyunfei
   
   ```

   

## SpringBoot中引入

上面介绍过，在Spring中注入配置文件需要使用@PropertiesSource指明文件的位置，然后通过@Value注入值。而在SpringBoot中，只需要在application.properties中增加属性，直接用@Value注入即可。

若配置有许多个，频繁的使用@Value会很麻烦，因此，SpringBoot支持基于类型安全的配置方式。通过@ConfigurationProperties将properties属性和一个Bean关联，从而实现类型安全的配置。

spring boot 1.5版本以前可以通过location属性指定配置文件路径，1.5以后configurationProperties删除了location属性，可以用@PropertySource来指定要读取的配置文件。

1. 创建book.properties

   ```java
   book.name=springboot
   book.author=wisely
   ```

   

```java
package xin.study.springboot.entity;

import lombok.Data;
import org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Data
@Component
@PropertySource("classpath:/book.properties") //没有指定默认是application.properties
@ConfigurationProperties(prefix ="book")
public class Book {


    private  String name;

    private String author;

    public Book(){}

    public Book(String name,String author){
        this.name=name;
        this.author=author;
    }
}

```

@configurationProperties也可以和@Bean配置和使用

```java
@Bean
@ConfigurationProperties(prefix = "person")
public Person getPerson() {
    return new Person();
}
```

---------------------
[参考博客](https://blog.csdn.net/xdkb159/article/details/79359331 )


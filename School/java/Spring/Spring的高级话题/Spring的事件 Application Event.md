# 事件 Application Event

spring 的事件为Bean与Bean之间的消息通信提供了支持。当Bean处理完一个任务之后，喜欢另一个Bean知道并能做相应的处理，这时就需要一个Bean监听另一个Bean所发送的事件。

Spring的Bean需要遵循以下流程：

1. 自定义事件，继承ApplicationEvent.
2. 定义事件监听器，实现ApplicationListenter.
3. 使用容器发布事件.

## 示例

1. 定义事件

   ```java
   package xin.spring.event;
   
   import org.springframework.context.ApplicationEvent;
   
   public class DemoEvent extends ApplicationEvent {
   
       private String msg;
   
       public String getMsg() {
           return msg;
       }
   
       public void setMsg(String msg) {
           this.msg = msg;
       }
   
       public DemoEvent(Object object, String msg){
           super(object);
           this.msg=msg;
       }
   }
   
   ```

2. 定义事件监听器

   ```java
   package xin.spring.event;
   
   import org.springframework.context.ApplicationListener;
   import org.springframework.stereotype.Component;
   
   @Component
   public class DemoListener implements ApplicationListener<DemoEvent> {
       public void onApplicationEvent(DemoEvent demoEvent) {
           String msg=demoEvent.getMsg();
           System.out.println("接受到"+msg);
       }
   }
   ```

3. 定义事件发布类别

   ```java
   package xin.spring.event;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.context.ApplicationContext;
   import org.springframework.stereotype.Component;
   
   @Component
   public class DemoPublish {
       @Autowired
       ApplicationContext applicationContext;
   
       //使用容器发布
       public void publish(String msg){
           applicationContext.publishEvent(new DemoEvent(this,msg));
       }
   }
   
   ```

4. 配置类：

   ```java
   package xin.spring.event;
   
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   @ComponentScan(basePackageClasses = DemoListener.class)
   public class EventConfig {
   }
   
   ```

5. 测试类：

   ```java
   package xin.spring.event;
   
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.annotation.AnnotationConfigApplicationContext;
   
   public class Test {
   
       public static void main(String agrs[]){
           ApplicationContext context=new AnnotationConfigApplicationContext(EventConfig.class);
           DemoPublish demoPublish=context.getBean(DemoPublish.class);
           demoPublish.publish("hello world");
       }
   }
   
   ```


```
输出：接受到hello world
```




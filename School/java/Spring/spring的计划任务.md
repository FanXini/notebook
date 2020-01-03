# 计划任务

&emsp;&emsp;从Spring3.1开始，计划任务在spring中的实现就变得异常简单。首先通过在配置类注解@EnableSchedule来开启对计划任务的支持，然后在要执行计划任务的方法上注解@Schedule，声明这是一个计划任务。

&emsp;&emsp;spring通过@schedule支持多种类型的计划任务，包含cron、fixDelay、fixRate等。

## 示例

1. 计划任务执行类：

   ```java
   package com.wisely.highlight_spring4.ch3.taskscheduler;
   
   import java.text.SimpleDateFormat;
   import java.util.Date;
   
   import org.springframework.scheduling.annotation.Scheduled;
   import org.springframework.stereotype.Service;
   
   @Service
   public class ScheduledTaskService {
   	
   	  private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");
   
   	  @Scheduled(fixedRate = 5000) //1 每隔固定事件执行
   	  public void reportCurrentTime() {
   	       System.out.println("每隔五秒执行一次 " + dateFormat.format(new Date()));
   	   }
   
   	  @Scheduled(cron = "0 28 11 ? * *"  ) //2 每天的11.28分执行
   	  public void fixTimeExecution(){
   	      System.out.println("在指定时间 " + dateFormat.format(new Date())+"执行");
   	  }
   
   }
   
   ```

2. 配置类

   ```java
   package com.wisely.highlight_spring4.ch3.taskscheduler;
   
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.scheduling.annotation.EnableScheduling;
   
   @Configuration
   @ComponentScan("com.wisely.highlight_spring4.ch3.taskscheduler")
   @EnableScheduling //1 开启对计划任务的支持
   public class TaskSchedulerConfig {
   
   }
   
   ```

3. 运行

   ```java
   package com.wisely.highlight_spring4.ch3.taskscheduler;
   
   import org.springframework.context.annotation.AnnotationConfigApplicationContext;
   
   public class Main {
   	public static void main(String[] args) {
   		 AnnotationConfigApplicationContext context =
   	                new AnnotationConfigApplicationContext(TaskSchedulerConfig.class);
   		 
   	}
   
   }
   
   ```

4. 执行结果

   ```
   每隔五秒执行一次 08:53:16
   每隔五秒执行一次 08:53:21
   每隔五秒执行一次 08:53:26
   每隔五秒执行一次 08:53:31
   每隔五秒执行一次 08:53:36
   每隔五秒执行一次 08:53:41
   每隔五秒执行一次 08:53:46
   ```

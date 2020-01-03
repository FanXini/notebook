- https://www.cnblogs.com/juddhu/archive/2013/07/14/3189177.html(原文链接)
- 先说下我的需求

1，可以记录日记在我们的java开发项目周期中；

2，很简单即可输出日志；

3，每天按照时间将不同的日志输出到不同的文件中，每天输出日志到一个带有当前时间戳的文件中；

4，可以修改当前输出日志的文件名，文件名后缀是当前的日期，而不需要等待log4j的项目到第二天这个文件名才能生成带有时间戳的文件；

6，按不同的日志等级输出日志到不同的文件中，例如error文件中只有输出的log级别为error的日志，info级别的日志只输出到info文件（所以这里需要用的是log4j的xml配置文件而不是使用log4j.properties文件)

先说下log4j的几种log级别的等级：

日志记录器（Logger）的行为是分等级的。如下表所示：

分 为OFF、FATAL、ERROR、WARN、INFO、[DEBUG](https://www.baidu.com/s?wd=DEBUG&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)、ALL或者您定义的级别，这些级别是从高到低的级别。Log4j建议只使用四个级别，优先级从高到低分别是 ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关。比如在这里定义了INFO级别， 则应用程序中所有DEBUG级别的日志信息将不被打印出来

优先级从高到低分别是ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关。比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。程序会打印高于或等于所设置级别的日志，设置的日志等级越高，打印出来的日志就越少。如果设置级别为INFO，则优先级高于等于INFO级别（如：INFO、WARN、ERROR）的日志信息将可以被输出,小于该级别的如DEBUG将不会被输出。

所以从上面来说我们最好设置的日志级别为INFO ，或者是DEBUG级别。推荐使用DEBUG级别，可以显示所有的日志。

- 需要准备的jar包
- log4j.jar 包，可以直接在官方下载到：<http://logging.apache.org/log4j/1.2/download.html>
- log4j-extr包，主要是立即生成自定义的文件名（原始的log4j生成的文件后缀名只能在第二天名称才会变）

<http://mvnrepository.com/artifact/log4j/apache-log4j-extras>（之所以选择这个地址是因为原始的log4j extra的URL <http://www.apache.org/dyn/closer.cgi/logging/log4j/companions/extras/1.1/apache-log4j-extras-1.1.zip> 下载地址现在都失效了，不晓得是不是被墙了？？？）

下面的xml文件中的这行代码就是用到了log4j-extra中的类，千万主要了如果没有将这个jar加入到当前项目的classpath中，下面的这行代码运行的时候会报错的：

```csharp
<appender name="log4jDebug"  class="org.apache.log4j.rolling.RollingFileAppender">  
  <!-- 这里的class="org.apache.log4j.rolling.RollingFileAppender",而没有使用log4j的原始的：class="org.apache.log4j.DailyRollingFileAppender" -->
        <param name="Append" value="true"/>
        <rollingPolicy  class="org.apache.log4j.rolling.TimeBasedRollingPolicy">  
               <param name="FileNamePattern" value="./log/log_%d{yyyy-MM-dd}.log" />  
        </rollingPolicy>  

```

将上面的两个jar文件放到项目的classpath中，如下：

[![image](https://images0.cnblogs.com/blog/345627/201307/14104435-15702069f7214ff2ae177ceae64cebf7.png)](https://images0.cnblogs.com/blog/345627/201307/14104428-0ac75f966a4a442cb7a11acf79144e51.png)

 

配置文件

只需要一个配置文件我们这里就可以使用log4j的所以的功能，是不是很简单，所以主要的重心就是放在log4j配置文件上。但是如果是使用的是log4j的properties文件，它输出的日志记录可能不是你需要的，就想上面提到的日志级别，如果使用properties文件，那么所有的error和warn级别的日志都会输出到INFO级别的日志中，如果我们这里设置log的级别为info的话。

所以这样的话，一个文件中可能有info级别的，error级别的，warn级别的，里面还是很混乱的，很难快速查找定位对应级别的日志。

log4j提出了一个方案就是使用xml配置文件起配置log4j这样就可以使用不同的log4j级别输出不同的文件中。

 

以下是一个log4j的xml文件配置详情，可以满足以上所有的需求(如果需要的话你还可以定制其中的日志输出路径等等，另外说下，经自己试验，发现log4j是可以使用相对路径来输出对应的log日志文件的，并不像有的人说的不能使用相对路径只能使用绝对路径，这个是错的，我这里下面使用的就是相对路径，是相对当前的项目路径）。

使用的时候不管是properties文件或者是xml配置文件，最好一定要放在项目根目录下面，就是项目中跟src目录同级别里面。

[![image](https://images0.cnblogs.com/blog/345627/201307/14104437-8a654559ff634811b69e9cac11f3b8ab.png)](https://images0.cnblogs.com/blog/345627/201307/14104436-3645328034f144e980cfe2f7842401c8.png)

log4j.xml配置文件源代码如下：

```csharp
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">  
    
<log4j:configuration  debug="true" xmlns:log4j='http://jakarta.apache.org/log4j/' >  
 
    <!-- ========================== 自定义输出格式说明================================ -->
      <!-- %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL -->
      <!-- %r 输出自应用启动到输出该log信息耗费的毫秒数  -->
      <!-- %c 输出所属的类目，通常就是所在类的全名 -->
      <!-- %t 输出产生该日志事件的线程名 -->
      <!-- %n 输出一个回车换行符，Windows平台为“/r/n”，Unix平台为“/n” -->
      <!-- %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921  -->
      <!-- %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlo4.main(TestLog4.java:10)  -->
      <!-- ========================================================================== -->
 
      <!-- ========================== 输出方式说明================================ -->
      <!-- Log4j提供的appender有以下几种:  -->
      <!-- org.apache.log4j.ConsoleAppender(控制台),  -->
      <!-- org.apache.log4j.FileAppender(文件),  -->
      <!-- org.apache.log4j.DailyRollingFileAppender(每天产生一个日志文件), -->
      <!-- org.apache.log4j.RollingFileAppender(文件大小到达指定尺寸的时候产生一个新的文件),  -->
      <!-- org.apache.log4j.WriterAppender(将日志信息以流格式发送到任意指定的地方)   -->
  <!-- ========================================================================== -->
   
    <appender name="CONSOLE" class="org.apache.log4j.ConsoleAppender">
         <!-- <param name="Target" value="System.out"/> -->
         <layout class="org.apache.log4j.PatternLayout">
                 <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c  Method: %l ]%n%p:%m%n"/>
         </layout>
        <!--  <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="LevelMin" value="DEBUG"/>
            <param name="LevelMax" value="DEBUG"/>
        </filter> -->
    </appender>
    <!-- output the debug   -->
   <!--  <appender name="log4jDebug" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="log_"/>    
        <param name="MaxFileSize" value="KB"/> 
        <param name="MaxBackupIndex" value="2"/> -->
   <appender name="log4jDebug"  class="org.apache.log4j.rolling.RollingFileAppender">  
        <param name="Append" value="true"/>
        <rollingPolicy  class="org.apache.log4j.rolling.TimeBasedRollingPolicy">  
               <param name="FileNamePattern" value="./log/log_%d{yyyy-MM-dd}.log" />  
        </rollingPolicy>  
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c Method: %l ]%n%p:%m%n"/>
        </layout>
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="LevelMin" value="DEBUG"/>
            <param name="LevelMax" value="DEBUG"/>
        </filter>
    </appender>
   <!--  <appender name="log4jInfo" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="log_"/>       
        <param name="DatePattern" value="'.log'yyyy-MM-dd"/>
        <param name="Append" value="true"/>
       <param name="MaxFileSize" value="5KB"/>
        <param name="MaxBackupIndex" value="2"/> -->
    <appender name="log4jInfo"  class="org.apache.log4j.rolling.RollingFileAppender">  
        <param name="Append" value="true"/>
        <rollingPolicy  class="org.apache.log4j.rolling.TimeBasedRollingPolicy">  
               <param name="FileNamePattern" value="./log/log_%d{yyyy-MM-dd}.log" />  
        </rollingPolicy> 
        <layout class="org.apache.log4j.PatternLayout">
             <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c  Method: %l ]%n%p:%m%n"/>
        </layout>
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="LevelMin" value="INFO"/>
            <param name="LevelMax" value="INFO"/>
        </filter>
    </appender>
   <!--  <appender name="log4jWarn" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="/log_"/>      
        <param name="DatePattern" value="'.log'yyyy-MM-dd"/>
        <param name="Append" value="true"/>
        <param name="MaxFileSize" value="5KB"/>
        <param name="MaxBackupIndex" value="2"/> -->
    <appender name="log4jWarn" class="org.apache.log4j.rolling.RollingFileAppender">  
        <param name="Append" value="true"/>
        <rollingPolicy  class="org.apache.log4j.rolling.TimeBasedRollingPolicy">  
               <param name="FileNamePattern" value="./log/log_%d{yyyy-MM-dd}.log" />  
        </rollingPolicy> 
        <layout class="org.apache.log4j.PatternLayout">
             <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c Method: %l ]%n%p:%m%n"/>
        </layout>
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="LevelMin" value="WARN"/>
            <param name="LevelMax" value="WARN"/>
        </filter>
    </appender>
  <!--  <appender name="log4jError" class="org.apache.log4j.DailyRollingFileAppender"> -->
   <appender name="log4jError"  class="org.apache.log4j.rolling.RollingFileAppender">  
       <!--  <param name="File" value="/error_"/>    
        <param name="DatePattern" value="'.log'yyyy-MM-dd"/> -->
        <param name="Append" value="true"/>
        <rollingPolicy  class="org.apache.log4j.rolling.TimeBasedRollingPolicy">  
               <param name="FileNamePattern" value="./log/error_%d{yyyy-MM-dd}.log" />  
        </rollingPolicy> 
        
      <!--   <param name="MaxFileSize" value="5KB"/> -->
      <!--   <param name="MaxBackupIndex" value="2"/> -->
        <layout class="org.apache.log4j.PatternLayout">
             <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c Method: %l ]%n%p:%m%n"/>
        </layout>
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="LevelMin" value="ERROR"/>
            <param name="LevelMax" value="ERROR"/>
        </filter>
    </appender>
 <!--通过<category></category>的定义可以将各个包中的类日志输出到不同的日志文件中-->
    <!--     <category name="com.gzy">
            <priority value="debug" />
            <appender-ref ref="log4jTestLogInfo" />
            <appender-ref ref="log4jTestDebug" />
        </category> -->
  <appender name="MAIL"     
      class="org.apache.log4j.net.SMTPAppender">     
      <param name="threshold" value="debug" />     
      <!-- 日志的错误级别     
       <param name="threshold" value="error"/>     
      -->     
      <!-- 缓存文件大小，日志达到512K时发送Email -->     
      <param name="BufferSize" value="512" /><!-- 单位K -->     
      <param name="From" value="test@163.com" />     
      <param name="SMTPHost" value="smtp.163.com" />     
      <param name="Subject" value="juyee-log4jMessage" />     
      <param name="To" value="test@163.com" />     
      <param name="SMTPUsername" value="test" />     
      <param name="SMTPPassword" value="test" />     
      <layout class="org.apache.log4j.PatternLayout">     
       <param name="ConversionPattern"     
        value="%-d{yyyy-MM-dd HH:mm:ss.SSS a} [%p]-[%c] %m%n" />     
      </layout>     
   </appender> 
    
    
     <root>
        <priority value="debug"/>
        <appender-ref ref="CONSOLE" /> 
        <appender-ref ref="log4jDebug" /> 
        <appender-ref ref="log4jInfo" />
        <appender-ref ref="log4jWarn" />
        <appender-ref ref="log4jError" />
        <!-- <appender-ref ref="MAIL" /> -->
    </root>
</log4j:configuration>

```

是不是很简单啊，呵呵。

完成以上的配置后，现在就可以试试使用log4j了。新建一个java类，源代码中如下使用即可：

这里附上另一种配置log4j的文件，是使用log4j的properties文件，上面也说到了，它是有缺陷的，就是里面的日志等级可能都会输出到一个文件中，高级别的日志信息也会在低级别的日志文件中出现，有点混乱。
log4j.properties 文件：

```properties
# priority  :debug<info<warn<error
#you cannot specify every priority with different file for log4j 
log4j.rootLogger=debug,stdout,info,debug,warn,error 
 
#console
log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern= [%d{yyyy-MM-dd HH:mm:ss a}]:%p %l%m%n
#info log
log4j.logger.info=info
log4j.appender.info=org.apache.log4j.DailyRollingFileAppender 
log4j.appender.info.DatePattern='_'yyyy-MM-dd'.log'
log4j.appender.info.File=./src/com/hp/log/info.log
log4j.appender.info.Append=true
log4j.appender.info.Threshold=INFO
log4j.appender.info.layout=org.apache.log4j.PatternLayout 
log4j.appender.info.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c >> Method: %l ]%n%p:%m%n
#debug log
log4j.logger.debug=debug
log4j.appender.debug=org.apache.log4j.DailyRollingFileAppender 
log4j.appender.debug.DatePattern='_'yyyy-MM-dd'.log'
log4j.appender.debug.File=./src/com/hp/log/debug.log
log4j.appender.debug.Append=true
log4j.appender.debug.Threshold=DEBUG
log4j.appender.debug.layout=org.apache.log4j.PatternLayout 
log4j.appender.debug.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c >> Method: %l ]%n%p:%m%n
#warn log
log4j.logger.warn=warn
log4j.appender.warn=org.apache.log4j.DailyRollingFileAppender 
log4j.appender.warn.DatePattern='_'yyyy-MM-dd'.log'
log4j.appender.warn.File=./src/com/hp/log/warn.log
log4j.appender.warn.Append=true
log4j.appender.warn.Threshold=WARN
log4j.appender.warn.layout=org.apache.log4j.PatternLayout 
log4j.appender.warn.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c >> Method: %l ]%n%p:%m%n
#error
log4j.logger.error=error
log4j.appender.error = org.apache.log4j.DailyRollingFileAppender
log4j.appender.error.DatePattern='_'yyyy-MM-dd'.log'
log4j.appender.error.File = ./src/com/hp/log/error.log 
log4j.appender.error.Append = true
log4j.appender.error.Threshold = ERROR 
log4j.appender.error.layout = org.apache.log4j.PatternLayout
log4j.appender.error.layout.ConversionPattern = %d{yyyy-MM-dd HH:mm:ss a} [Thread: %t][ Class:%c >> Method: %l ]%n%p:%m%n

```



是不是很简单啊，呵呵。

完成以上的配置后，现在就可以试试使用log4j了。新建一个java类，源代码中如下使用即可：

>  **private static Logger logger=Logger.getLogger(类名.class);**

## log4j 设置日志输出文件的路径

大家一般都在Windows上开发,部署到linux服务器上通常会出现一些路径报错,因为两个操作系统的文件系统不同,不能用绝对路径来定义log4j的输出日志路径.对于相对路径来说,有以下解决方法.
​	1.日志输出debug级别以上的到stdout(控制台) 和R1(自己随便定义的)

> log4j.rootLogger=debug,stdout,R1  

2.输出到 盘的 根目录下 （不推荐，win和linux 不同）,代码如下:

> og4j.appender.R1.File=/log.log 

3.项目文件中 （不推荐，容易清理掉）

> log4j.appender.R1.File=logs/log.log 

4.tomcat系的容器 这种方法不错，切到别的容器就不行了

> log4j.appender.R.File=${catalina.home}/logs/log.log 

5.在web.xml中添加

```xml
<context-param>
	<param-name>webAppRootKey</param-name>
	<param-value>webApp.root</param-value>  
</context-param>
```

**log4j.properties中**
**log4j.appender.R1.File=${webApp.root}logs/log.log**

（这种方法的好处是不区分系统，不区分容器，缺点是会产生垃圾文件，${webApp.root} 在这个被赋值前有段日志不会在你想要的地方，当然妨碍不大，我推荐只用这种方法）

坦白说,我目前没有一个跨操作系统又能解决路径,还能保存好日志文件的完美解决办法.大家自由选择吧.

作者：_路人_ 
来源：CSDN 
原文：https://blog.csdn.net/lg346426260/article/details/56674290 
版权声明：本文为博主原创文章，转载请附上博文链接！
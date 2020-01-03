博客：Tomcat日志设定 http://blog.csdn.net/lk_cool/article/details/4561306

本文章解决办法： 
1、修改tomcat的日志配置，配置输出日志级别 
2、修改工程的日志配置：输出在控制台的级别

一：改变日志输出级别： 
方法一：一般在部署Tomcat后，运行久了，catalina.out文件会越来越大，对系统的稳定造成了一定的影响。 
可通过修改<font color='red'><conf/logging.properties</font>日志配置文件来屏蔽掉这部分的日志信息。

1catalina.org.apache.juli.FileHandler.level = WARNING 
1catalina.org.apache.juli.FileHandler.directory = ${catalina.base}/logs 
1catalina.org.apache.juli.FileHandler.prefix = catalina.

将level级别设置成WARNING就可以大量减少日志的输出，当然也可以设置成OFF，直接禁用掉。

一般日志的级别有： 
SEVERE (highest value) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)



方法二：进行切割文件 
日志切割：https://www.iyunv.com/forum.php?mod=viewthread&tid=404484&highlight=tomcat%2B

tomcat 日志禁用 ：https://www.iyunv.com/forum.php?mod=viewthread&tid=348904&highlight=tomcat%2B日志

二、解决catalina.out日志超大问题 
接下来考虑catalina.out怎么会这么大呢？查看里面记录了应用的所有日志信息。

查看应用的log4j配置文件，发现输出到控制台的配置，target是System.out

而catalina.out会记录 System.out 与 System.err的信息

删除log4j中的输出控制台的日志配置，catalina.out中不再记录应用的日志。 
日志输出级别：ALL、DEBUG、INFO、WARN、ERROR 
这下它不会涨的那么快了。设置工程项目输出至控制台catalina.out日志的级别：



Log4j具体输出信息级别配置方法：http://blog.csdn.net/hamov/article/details/51673875
--------------------- 
作者：weixin_36586564 
来源：CSDN 
原文：https://blog.csdn.net/weixin_36586564/article/details/78550110 
版权声明：本文为博主原创文章，转载请附上博文链接！
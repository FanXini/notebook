# 关于Mysql的wait_timeout连接超时问题报错解决方案.

bug回顾 ：

  想必大家在用MySQL时都会遇到连接超时的问题，如下图所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
### Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 47,795,922 milliseconds ago.  The last packet sent successfully to the server was 47,795,922 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
; SQL []; The last packet successfully received from the server was 47,795,922 milliseconds ago.  The last packet sent successfully to the server was 47,795,922 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.; nested exception is com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 47,795,922 milliseconds ago.  The last packet sent successfully to the server was 47,795,922 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 大概意思是当前的connection所进行过的最新请求是在52,587秒之前，这个时间是大于服务所配置的wait_timeout时间的。

 

**<font color='red'>原因分析：</font>**

**MySQL连接时，服务器默认的“wait_timeout”是8小时，也就是说一个connection空闲超过8个小时，Mysql将自动断开该connection。connections如果空闲超过8小时，Mysql将其断开，而DBCP连接池并不知道该connection已经失效，如果这时有Client请求connection，DBCP将该失效的Connection提供给Client，将会造成异常。** 

 

mysql分析：

打开MySQL的控制台，运行:show variables like ‘%timeout%’，查看和连接时间有关的MySQL系统变量，得到如下结果： 

![img](https://images2015.cnblogs.com/blog/724399/201701/724399-20170105171319612-2146168205.png)

​    其中wait_timeout就是负责超时控制的变量，其时间为长度为28800s，就是8个小时，那么就是说MySQL的服务会在操作间隔8小时后断开，需要再次重连。也有用户在URL中使用jdbc.url=jdbc:mysql://localhost:3306/nd?autoReconnect=true来使得连接自动恢复，当然了，这是可以的，不过是MySQL4及其以下版本适用。MySQL5中已经无效了，必须调整系统变量来控制了。MySQL5手册中对两个变量有如下的说明： 

​    interactive_timeout：服务器关闭交互式连接前等待活动的秒数。交互式客户端定义为在mysql_real_connect()中使用CLIENT_INTERACTIVE选项的客户端。又见wait_timeout 
​    wait_timeout:服务器关闭非交互连接之前等待活动的秒数。在线程启动时，根据全局wait_timeout值或全局interactive_timeout值初始化会话wait_timeout值，取决于客户端类型(由mysql_real_connect()的连接选项CLIENT_INTERACTIVE定义)，又见interactive_timeout 
​    如此看来，两个变量是共同控制的，那么都必须对他们进行修改了。继续深入这两个变量wait_timeout的取值范围是1-2147483(Windows)，1-31536000(linux)，interactive_time取值随wait_timeout变动，它们的默认值都是28800。 
​    MySQL的系统变量由配置文件控制，当配置文件中不配置时，系统使用默认值，这个28800就是默认值。要修改就只能在配置文件里修改。Windows下在%MySQL HOME%/bin下有mysql.ini配置文件，打开后在如下位置添加两个变量，赋值。（这里修改为388000） 

 

解决方式：

### 1. 增加 MySQL 的 wait_timeout 属性的值 （不推荐）

修改mysql安装目录下的配置文件 my.ini文件（如果没有此文件，复制“my-default.ini”文件，生成“复件 my-default.ini”文件。将“复件 my-default.ini”文件重命名成“my.ini” ），在文件中设置：

```
wait_timeout=31536000  
interactive_timeout=31536000  
```

这两个参数的默认值是8小时(60*60*8=28800)。 注意： 1.wait_timeout的最大值只允许2147483 （24天左右）

也可以使用mysql命令对这两个属性进行修改

 ![img](https://images2015.cnblogs.com/blog/724399/201701/724399-20170105171930487-156198409.png)

 

### 2. 减少连接池内连接的生存周期

减少连接池内连接的生存周期，**使之小于**上一项中所设置的**wait_timeout 的值**。 

修改 c3p0 的配置文件，在 Spring 的配置文件中设置：

```
 <bean id="dataSource"  class="com.mchange.v2.c3p0.ComboPooledDataSource">      
    <property name="maxIdleTime"value="1800"/>  
    <!--other properties -->  
 </bean>
```

 

### 3. 定期使用连接池内的连接

定期使用连接池内的连接，使得它们不会因为闲置超时而被 MySQL 断开。 

修改 c3p0 的配置文件,在 Spring 的配置文件中设置：

```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
    <property name="preferredTestQuery" value="SELECT 1"/>  
    <property name="idleConnectionTestPeriod" value="18000"/>  
    <property name="testConnectionOnCheckout" value="true"/>  
</bean>
```

 

附上dbcp和c3p0的标准配置

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
 <property name="driverClassName" value="com.mysql.jdbc.Driver" />
 <property name="url" value="jdbc:mysql://192.168.40.10:3336/XXX" />
 <property name="username" value="" />
 <property name="password" value="" />
 <property name="maxWait" value="20000"></property>
 <property name="validationQuery" value="SELECT 1"></property>
 <property name="testWhileIdle" value="true"></property>
 <property name="testOnBorrow" value="true"></property>
 <property name="timeBetweenEvictionRunsMillis" value="3600000"></property>
 <property name="numTestsPerEvictionRun" value="50"></property>
 <property name="minEvictableIdleTimeMillis" value="120000"></property>
 <property name="removeAbandoned" value="true"/>
 <property name="removeAbandonedTimeout" value="6000000"/>
</bean>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">   
　　<property name="driverClass"><value>oracle.jdbc.driver.OracleDriver</value></property>   
　　<property name="jdbcUrl"><value>jdbc:oracle:thin:@localhost:1521:Test</value></property>   
　　<property name="user"><value>Kay</value></property>   
　　<property name="password"><value>root</value></property>   
　　<!--连接池中保留的最小连接数。-->   
　　<property name="minPoolSize" value="10" />   
　　<!--连接池中保留的最大连接数。Default: 15 -->   
　　<property name="maxPoolSize" value="100" />   
　　<!--最大空闲时间,1800秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->   
　　<property name="maxIdleTime" value="1800" />   
　　<!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->   
　　<property name="acquireIncrement" value="3" />   
　　<property name="maxStatements" value="1000" />   
　　<property name="initialPoolSize" value="10" />   
　　<!--每60秒检查所有连接池中的空闲连接。Default: 0 -->   
　　<property name="idleConnectionTestPeriod" value="60" />   
　　<!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->   
　　<property name="acquireRetryAttempts" value="30" />   
　　<property name="breakAfterAcquireFailure" value="true" />   
　　<property name="testConnectionOnCheckout" value="false" />   
</bean>   
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

参考：

https://my.oschina.net/guanzhenxing/blog/213364

http://sarin.iteye.com/blog/580311/

http://blog.csdn.net/wangfayinn/article/details/24623575
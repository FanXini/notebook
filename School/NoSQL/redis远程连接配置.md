# Redis服务器开启远程访问

2017年12月29日 02:17:40 [xujian_2001](https://me.csdn.net/xujian_2001) 阅读数：9783



## redis[服务器](https://www.baidu.com/s?wd=%E6%9C%8D%E5%8A%A1%E5%99%A8&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)访问控制

> redis服务器默认是处于保护模式并只能本地访问，打开redis.conf文件可以看到如下配置

### 保护模式控制

![这里写图片描述](https://img-blog.csdn.net/20171229021342019?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveHVqaWFuXzIwMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 本地访问控制

![这里写图片描述](https://img-blog.csdn.net/20171229021558109?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveHVqaWFuXzIwMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以上两个地方修改后，重新启动redis服务，开启防火墙端口，默认是：6379在远程电脑telnet，可以看到已经通了！

远程连接命令：

redis-cli -h 138.138.138.138  -p  6379 




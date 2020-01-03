# [在windows上部署使用Redis](http://www.cnblogs.com/smileyearn/articles/4749746.html)



## 下载Redis

在Redis的官网[下载页](http://redis.io/download)上有[各种各样](https://www.baidu.com/s?wd=%E5%90%84%E7%A7%8D%E5%90%84%E6%A0%B7&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的版本，我这次是在windows上部署的，要去[GitHub](https://github.com/MSOpenTech/redis)上下载。目前的是2.8.12版的，直接解压，在`\bin\release` 目录下有个压缩包，这就是我们需要的：

 

![img](http://img.keenwon.com/2014/07/20140703211533_46794.png)

 

## 启动Redis

直接在上图的目录打开命令窗口，运行：

 

1. redis-server redis.windows.conf

 

![img](http://img.keenwon.com/2014/07/20140704091545_81621.png)

 

结果就悲剧了，提示：`QForkMasterInit: system error caught. error code=0x000005af, message=VirtualAllocEx failed.: unknown error` 。原因是内存分配的问题（如果你的电脑够强悍，可能不会出问题）。解决方法有两个，第一：启动的时候使用`--maxmemory` 命令限制Redis的内存：

 

1. redis-server redis.windows.conf --maxmemory 200m

 

第二种方法就是修改配置文件`redis.windows.conf` ：

 

1. maxmemory 209715200

 

注意单位是字节，改完后如下：

 

![img](http://img.keenwon.com/2014/07/20140703213628_54748.png)

 

之后再运行`redis-server redis.windows.conf` 就可以启动了：

 

![img](http://img.keenwon.com/2014/07/20140704091237_71992.png)

 

但是问题又来了，关闭cmd窗口就会关闭Redis，难道[服务器](https://www.baidu.com/s?wd=%E6%9C%8D%E5%8A%A1%E5%99%A8&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)上要一直开着吗？这显然是不科学的，下面看怎么在服务器上部署。

 

## 部署Redis

其实Redis是可以安装成[windows服务](https://www.baidu.com/s?wd=windows%E6%9C%8D%E5%8A%A1&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的，开机自启动，命令如下：

 

1. redis-server --service-install redis.windows.conf

 

安装完之后，就可看到Redis已经作为windows服务了：

 

![img](http://img.keenwon.com/2014/07/20140704091809_67096.png)

 

![img](http://img.keenwon.com/2014/07/20140704091925_80829.png)

 

但是安装好之后，Redis并没有启动，启动命令如下：

 

1. redis-server --service-start

 

停止命令：

 

1. redis-server --service-stop



启动客户端：redis-cli -p 6379 



还可以安装多个实例

 

1. redis-server --service-install –service-name redisService1 –port 10001
2. redis-server --service-start –service-name redisService1
3. redis-server --service-install –service-name redisService2 –port 10002
4. redis-server --service-start –service-name redisService2
5. redis-server --service-install –service-name redisService3 –port 10003
6. redis-server --service-start –service-name redisService3

 

卸载命令：

 

1. redis-server --service-uninstall

 

最后提示一下：2.8版本的不支持32位系统，32位系统要去下载[2.6版本](https://github.com/MSOpenTech/redis/tree/2.6/bin/release)的。2.6版本的无法像[上面](http://keenwon.com/1275.html)一样方便的部署，它提供一个叫[RedisWatcher](https://github.com/MSOpenTech/redis/tree/2.6/msvs/RedisWatcher)的程序来运行redis server，Redis停止后会自动重启。

 

另外推荐一个Redis可视化管理工具：Redis Desktop Manager，官网的下载地址被墙了，可以在[我的网盘下载](http://pan.baidu.com/s/1dDiWoj3) v0.7.6版，放个截图：

 

![img](http://img.keenwon.com/2014/07/20140704190716_67446.png)



转载自：<http://www.cnblogs.com/smileyearn/articles/4749746.html>
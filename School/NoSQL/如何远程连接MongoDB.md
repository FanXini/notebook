---
typora-copy-images-to: ..\..\img
---

# Ubuntu16.04系统下如何开启远程连接mongodb数据库

1、在linux下（或windows下）打开终端，进入mongodb的安装目录 
2、创建配置文件mongodb.conf(可以在任何文件目录下创建) 
![1546843171431](..\..\img\1546843171431.png)
3、sudo vim mongodb.conf命令进入配置文件并编辑 
![这里写图片描述](https://img-blog.csdn.net/20180908114125120?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x3X3dpc2hlcw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```xml
dbpath=/usr/local/mongodb/mongodb-3.6.5/data/db     #数据文件存放目录
logpath=/usr/local/mongodb/mongodb-3.6.5/data/log/mongodb.log   #日志文件存放目录   
port=27017   #端口号
fork=true    #以守护程序的方式启用,即在后台运行
logappend = true  #日志以追加的形式添加
bind_ip = 0.0.0.0 #可以访问的地址. 127.0.0.1表示自己访问,  0.0.0.0 表示所有人都能访问123456
```

必须使用sudo命令进入，不然添加配置保存退出时权限不够 
4、编辑完之后：wq 保存退出，然后进入mongodb的bin目录，以配置文件启动 mongodb 服务

```
./mongod -config /mongodb.conf
#如果配置过环境变量，则直接输入：
mongod -config /mongodb.conf
```

如果你之前已经开启了mongodb服务，现在是要修改mongodb的配置，则在运行上面的命令之前，要先关闭mongodb服务：

`pkill mongod`或者在mongo shell 中进入admin数据库，使用`db.shutdownServer();`命令

![1546843631355](..\..\img\1546843631355.png)

![1546843612119](..\..\img\1546843612119.png)

5 、在本地电脑上使用Mongo命令远程连接

```java
mongo ip:port
```

![1546843949722](..\..\img\1546843949722.png)
---
typora-copy-images-to: ..\..\img
---

[参考博客](https://www.cnblogs.com/Lovebugs/p/8606000.html)

# Ubuntu 16.04安装MongoDB4.0.5教程及常见安装问题解决方案

MongoDB 提供了 linux 各发行版本 64 位的安装包，你可以在官网下载安装包。

#### 1 .下载地址：<https://www.mongodb.com/download-center#community>

![1546771126490](..\..\img\1546771126490.png)

下载完安装包，并解压 **tgz**（以下演示的是 64 位 Linux上的安装） 。

#### ２．默认下载路径是到用户目录下的Downloads目录，将其解压

```
tar -zxvf mongodb-linux-x86_64-3.2.12.tgz
```

#### ３．将解压后的文件夹移动到/usr/local/的mongodb目录下

```
mv mongodb-linux-x86_64-3.2.12 /usr/local/mongodb
```

#### ４．配置系统文件profile

```
sudo vi /etc/profile
```

插入下列内容：

```
export MONGODB_HOME=/usr/local/mongodb  
export PATH=$PATH:$MONGODB_HOME/bin
```

注意保存后要重启系统配置：

```
source /etc/profile
```

#### ５．创建用于存放数据和日志文件的文件夹，并修改其权限增加读写权限

```
cd /usr/local/mongodb
sudo mkdir data
cd data
sudo mkdir db
cd ..
sudo chmod 777 data
sudo chmod 777 data/db
sudo mkdir logs
cd logs
touch mongodb.log
```

![img](https://images2018.cnblogs.com/blog/1101099/201803/1101099-20180319233418015-953209930.png)

#### ６．mongodb启动配置

进入到bin目录，增加一个配置文件：

```
cd /usr/local/mongodb/bin  
sudo vi mongodb.conf
```

插入下列内容：

```
dbpath = /usr/local/mongodb/data/db #数据文件存放目录  
logpath = /usr/local/mongodb/logs/mongodb.log #日志文件存放目录  
port = 27017  #端口  
fork = true  #以守护程序的方式启用，即在后台运行  
auth=false #不开启认证功能
```

常见配置属性介绍：

![1546847375130](..\..\img\1546847375130.png)

#### ７． 启动mongod数据库服务，以配置文件的方式启动

```
cd /usr/local/mongodb/bin
./mongod -f mongodb.conf  #也可以用-config 代替-f
#如果已经在第四步配置了系统环境变量，则可以直接输入
mongod -config mongodb.conf
```

#### ８．连接mongodb数据库

```
./mongo
#如果已经在第四步配置了系统环境变量，则可以直接输入
mongo
```

![img](https://images2018.cnblogs.com/blog/1101099/201803/1101099-20180319233505037-1320009908.png)

#### ９．设置mongodb.service启动服务，设置开机启动

```
cd /lib/systemd/system  
sudo vi mongodb.service 
```

编辑其内容为：

```
[Unit]  
Description=mongodb  
After=network.target remote-fs.target nss-lookup.target  
  
[Service]  
Type=forking  
ExecStart=/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf  
ExecReload=/bin/kill -s HUP $MAINPID  
ExecStop=/usr/local/mongodb/bin/mongod --shutdown --config /usr/local/mongodb/bin/mongodb.conf  
PrivateTmp=true  
  
[Install]  
WantedBy=multi-user.target
```

#### 10．设置mongodb.service权限

```
chmod 754 mongodb.service
```

#### 11．系统mongodb.service的操作命令如下：

```
#启动服务  
systemctl start mongodb.service  
#关闭服务  
systemctl stop mongodb.service  
#开机启动  
systemctl enable mongodb.service 
```

#### 12．mongodb.service启动测试

![img](https://images2018.cnblogs.com/blog/1101099/201803/1101099-20180319233548242-1156614625.png)

####  13 . 关闭mongodb

1. 在shell下关闭mongodb（必须在admin数据库中使用该命令）

`db.shutdownServer();`

![1546831364518](..\..\img\1546831364518.png)

2. 直接关闭mongo

```
如果是使用mongod 开启的mongodb
使用pkill mongod   //前提是配置了环境变量
```

![1546831428465](..\..\img\1546831428465.png)

## **二、安装过程中遇到的问题**

**1.**在启动./mongod 的时候缺少 libcurl.so 库

```
./mongod: error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directory
```

解决方法：执行下面语句

> apt-get install libcurl4-openssl-dev

2 
`./mongod: error while loading shared libraries: libnetsnmpmibs.so.30: cannot open shared object file: No such file or directory`

解决方法：

> apt-get install snmpd snmp snmp-mibs-downloader

原因缺少 snmpd 相关的东西 。客户端服务端都给它装了。

3．通过配置文件启动服务：mongod -f /etc/mongodb.conf 时报错

```
Error parsing INI config file: unrecognised option 'nohttpinterface' try './
```

这个一开始让我查了好久，后面查到是因为我下载的最新版本的mongodb，而最新的版本貌似不支持以这种配置文件的方式来启动服务，所以无奈我又重新下载安装了3.2.12的版本，然后再次启动服务就正常了。W

4．启动服务时报错：

```
about to fork child process, waiting until server is ready for connections.
forked process: 11335
ERROR: child process failed, exited with error number 1
```

这个错误原因是dbpath文件的权限问题,data和logs目录增加写权限即可，上面提到了。

 

参考链接：

http://blog.csdn.net/a123demi/article/details/70238972
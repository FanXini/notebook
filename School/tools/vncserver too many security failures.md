2018年1月25日

# vncserver too many security failures

在服务器上开了几个虚拟机，装了VNC之后，经常遇到报错too many security failures。查了下相关资料，原来是有人在暴力破解，触发了VNC的黑名单机制。重置黑名单，就能登录了。

```
vncconfig -display :1 -set BlacklistTimeout=0 -set BlacklistThreshold=1000000
```

display ：指定桌面号
BlacklistTimeout ： 设置黑名单的过期时间
BlacklistThreshold ： 允许的失败次数

重新登录之后记得还原黑名单设置，不然就失去了这层防护

```
vncconfig -display :1 -set BlacklistTimeout=600000000000 -set BlacklistThreshold=10
```

默认的过期时间是600秒，这里我设置一个大数，黑名单的IP永远关小黑屋

开启窗口命令

vncserver :1
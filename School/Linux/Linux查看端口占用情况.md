# [linux查看端口占用情况](https://www.cnblogs.com/wangtao1993/p/6144183.html)



今天要使用python写一个端口探测的小程序，以检测一些特定的服务端口有没有被占用，突然发现自己居然不知道在linux中如何查询端口被占用的情况，天呐，赶快学习一下。😁

 

Linux如何查看端口

1、lsof -i:端口号 用于查看某一端口的占用情况，比如查看8000端口使用情况，lsof -i:8000

```
# lsof -i:8000
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
lwfs    22065 root    6u  IPv4 4395053      0t0  TCP *:irdmi (LISTEN)
```

可以看到8000端口已经被轻量级文件系统转发服务lwfs占用

 

2、netstat -tunlp |grep 端口号，用于查看指定的端口号的进程情况，如查看8000端口的情况，netstat -tunlp |grep 8000

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# netstat -tunlp 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      4814/rpcbind        
tcp        0      0 0.0.0.0:5908                0.0.0.0:*                   LISTEN      25492/qemu-kvm      
tcp        0      0 0.0.0.0:6996                0.0.0.0:*                   LISTEN      22065/lwfs          
tcp        0      0 192.168.122.1:53            0.0.0.0:*                   LISTEN      38296/dnsmasq       
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      5278/sshd           
tcp        0      0 127.0.0.1:631               0.0.0.0:*                   LISTEN      5013/cupsd          
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      5962/master         
tcp        0      0 0.0.0.0:8666                0.0.0.0:*                   LISTEN      44868/lwfs          
tcp        0      0 0.0.0.0:8000                0.0.0.0:*                   LISTEN      22065/lwfs        
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# netstat -tunlp | grep 8000
tcp        0      0 0.0.0.0:8000                0.0.0.0:*                   LISTEN      22065/lwfs          
```

 

说明一下几个参数的含义：

​                                

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 -t (tcp) 仅显示tcp相关选项
                                 -u (udp)仅显示udp相关选项
                                 -n 拒绝显示别名，能显示数字的全部转化为数字
                                 -l 仅列出在Listen(监听)的服务状态
                                 -p 显示建立相关链接的程序名
 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

附加一个python端口占用监测的程序，该程序可以监测指定IP的端口是否被占用。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 #!/usr/bin/env python
  2 # -*- coding:utf-8 -*-
  3 
  4 import socket, time, thread
  5 socket.setdefaulttimeout(3) #设置默认超时时间
  6 
  7 def socket_port(ip, port):
  8     """
  9     输入IP和端口号，扫描判断端口是否占用
 10     """
 11     try:
 12         if port >=65535:
 13             print u'端口扫描结束'
 14         s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
 15         result=s.connect_ex((ip, port))
 16         if result==0:
 17             lock.acquire()
 18             print ip,u':',port,u'端口已占用'
 19             lock.release()
 20     except:
 21         print u'端口扫描异常'
 22 
 23 def ip_scan(ip):
 24     """
 25     输入IP，扫描IP的0-65534端口情况
 26     """
 27     try:
 28         print u'开始扫描 %s' % ip
 29         start_time=time.time()
 30         for i in range(0,65534):
 31             thread.start_new_thread(socket_port,(ip, int(i)))
 32         print u'扫描端口完成，总共用时：%.2f' %(time.time()-start_time)
 33 #       raw_input("Press Enter to Exit")
 34     except:
 35         print u'扫描ip出错'
 36 
 37 if __name__=='__main__':
 38     url=raw_input('Input the ip you want to scan: ')
 39     lock=thread.allocate_lock()
 40     ip_scan(url)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该程序执行结果如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# python scan_port.py
Input the ip you want to scan: 20.0.208.112
开始扫描 20.0.208.112
20.0.208.112 : 111 端口已占用
20.0.208.112 : 22 端口已占用
20.0.208.112 : 8000 端口已占用
20.0.208.112 : 15996 端口已占用
20.0.208.112 : 41734 端口已占用
扫描端口完成，总共用时：9.38
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)
# [linux查看进程和线程的命令](https://www.cnblogs.com/panxuejun/p/8759385.html)

[Ubuntu ps命令详解](http://data.digitser.net/ubuntu/zh-CN/ps.html)



1.任务：获得进程信息

：ps命令，或者top命令，它能显示当前运行中进程的相关信息，包括进程的PID。

top -H p pid

ps命令能提供一份当前进程的快照。如果想状态可以自动刷新，可以使用top命令。

2.任务：获得线程信息

：

输入下列命令：

```
# ps -eLf
# ps axms
```
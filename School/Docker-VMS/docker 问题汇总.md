# [docker本地化异常：/bin/sh: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)](https://www.cnblogs.com/hellojesson/p/10831479.html)

docker中经常设置不了 环境变量$LC_ALL, 导致报很多奇怪的编码错误：

/bin/sh: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)

解决方法： 

 sudo localedef -i en_US -f UTF-8 en_US.UTF-8

https://www.cnblogs.com/ifantastic/p/4565822.html


[原文](https://www.cnblogs.com/chenwenbiao/archive/2011/09/12/2174146.html)

linux系统里提供的文件转化编码的命令iconv，使用如下：
 

iconv -t utf-8 -f gb2312 -c my_database.sql > new.sql

-f 原编码
-t 目标编码
-c 忽略无法转换的字符
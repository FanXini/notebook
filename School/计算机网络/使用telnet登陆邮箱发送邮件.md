---
typora-copy-images-to: ..\..\img
---







## 先打开邮箱的smtp服务并获取授权码

请访问：http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256

![1562293777107](..\..\img\1562293777107.png)

## 步骤

注意下面绿色为你要在cmd端输入的内容
`telnet smtp.qq.com 25`

> 220 smtp.qq.com Esmtp QQ Mail Server

`helo  qq.com`

> 与qq服务器握手
> 250 smtp.qq.com

`auth login`

> 334 VXNlcm5hbWU6

`NDcyODQzMzI2QHFxLmNvbQ==`     

> 我的真实qq邮箱472843326@qq.com的base64编码为NDcyODQzMzI2QHFxLmNvbQ== 334 UGFzc3dvcmQ6

YmhmZ2x5bXZ0emtsYmdkYw==

编码的你的授权码，我就不告诉你了，免得你解码

[实现base64编码和解码的在线工具](http://tool.oschina.net/encrypt?type=3)

235 Authentication successful

```
mail from:<992014714@qq.com>
250 Ok
rcpt to:<1274096233@qq.com>;<1375333986@qq.com>
250 Ok
data
354 End data with <CR><LF>.<CR><LF>
from:<992014714@qq.com>
to:<1274096233@qq.com>;<1375333986@qq.com>
subject:hello,you!
//邮件内容：空一行
my name is zhang lian quan
.//“.”结束
250 Ok: queued as220 163.com Anti-spam GT for Coremail System (163com[20111010])
```



作者：zhangyulin54321 
来源：CSDN 
原文：https://blog.csdn.net/zhangyulin54321/article/details/7599403 
版权声明：本文为博主原创文章，转载请附上博文链接！

我的账户信息

```
472843326@qq.com
NDcyODQzMzI2QHFxLmNvbQ==
授权码 ：bhfglymvtzklbgdc 
YmhmZ2x5bXZ0emtsYmdkYw==
```


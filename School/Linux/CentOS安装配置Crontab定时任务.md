# CentOS安装配置Crontab定时任务

yum install crontabs

说明：

service crond start //启动服务
service crond stop //关闭服务
service crond restart //重启服务
service crond reload //重新载入配置
查看crontab服务状态：service crond status
手动启动crontab服务：service crond start

查看crontab服务是否已设置为开机启动，执行命令：ntsysv

加入开机自动启动:

chkconfig crond on



 1、编辑命令

   1)、在命令行输入: crontab -e 然后添加相应的任务，wq存盘退出

   2)、直接编辑/etc/crontab 文件，即vi /etc/crontab，添加相应的任务

2、文件格式

   Minute Hour Day Month DayofWeek CommandPath

 3、参数说明

   Minute：每个小时的第几分钟执行该任务；取值范围0-59

   Hour：每天的第几个小时执行该任务；取值范围0-23

   Day：每月的第几天执行该任务；取值范围1-31

   Month：每年的第几个月执行该任务；取值范围1-12

   DayOfWeek：每周的第几天执行该任务；取值范围0-6，0表示周末

   CommandPath：指定要执行的程序路径

 4、时间格式

   \* ：表示任意的时刻；如小时位 * 则表示每个小时

   n ：表示特定的时刻；如小时位 5 就表示5时

   n,m ：表示特定的几个时刻；如小时位 1,10 就表示1时和10时

   n－m ：表示一个时间段；如小时位 1-5 就表示1到5点

   */n : 表示每隔多少个时间单位执行一次；如小时位 */1 就表示每隔1个小时执行一次命令，也可以写成 1-23/1

 5、调度示例

 \* 1 * * * /opt/script/backup.sh ：从1:0到1:59 每隔1分钟 执行
 15 05 * * * /opt/script/backup.sh ：05:15 执行
*/10 * * * * /opt/script/backup.sh ：每隔10分 执行
0 17 * * 1 /opt/script/backup.sh ：每周一的 17:00 执行
2 8-20/3 * * * /opt/script/backup.sh 8:02,11:02,14:02,17:02,20:02 执行

实际举例

crontab文件的一些例子：

30 21 * * * /etc/init.d/nginx restart      //每晚的21:30重启 nginx。
45 4 1,10,22 * * /etc/init.d/nginx restart    //每月1、 10、22日的4 : 45重启nginx。
10 1 * * 6,0 /etc/init.d/nginx restart      //每周六、周日的1 : 10重启nginx。
0,30 18-23 * * * /etc/init.d/nginx restart    //每天18 : 00至23 : 00之间每隔30分钟重启nginx。
0 23 * * 6 /etc/init.d/nginx restart       //每星期六的11 : 00 pm重启nginx。
\* */1 * * * /etc/init.d/nginx restart      //每一小时重启nginx
\* 23-7/1 * * * /etc/init.d/nginx restart     //晚上11点到早上7点之间，每 隔一小时重启nginx
0 11 4 * mon-wed /etc/init.d/nginx restart    //每月的4号与每周一到周三 的11点重启nginx
0 4 1 jan * /etc/init.d/nginx restart      //一月一号的4点重启nginx
*/30 * * * * /usr/sbin/ntpdate 210.72.145.20   //每半小时同步一下时间

每五分钟执行 */5 * * * *

每小时执行   0 * * * *

每2小时执行   0 */2 * * *

每天执行    0 0 * * *

每周执行    0 0 * * 0

每月执行    0 0 1 * *

每年执行    0 0 1 1 *



### 1. cron服务【Ubuntu环境】

查看cron状态

```html
sudo  service cron status　
```

开启cron

```html
sudo /etc/init.d/cron start
```

关闭cron

```html
sudo /etc/init.d/cron stop
```

重启cron

```html
sudo /etc/init.d/cron restart
```

![img](https://images0.cnblogs.com/blog2015/408927/201505/112150114235275.png)　　

[回到顶部](https://www.cnblogs.com/kaituorensheng/p/4494321.html#_labelTop)

### 2. crontab用法

crontab –e : 修改 crontab 文件，如果文件不存在会自动创建。 
crontab –l : 显示 crontab 文件。 
crontab -r : 删除 crontab 文件。
crontab -ir : 删除 crontab 文件前提醒用户。

 

在crontab文件中写入需要执行的命令和时间，该文件中每行都包括六个域，其中前五个域是指定命令被执行的时间，最后一个域是要被执行的命令。每个域之间使用空格或者制表符分隔。格式如下： 

```html
minute hour day-of-month month-of-year day-of-week commands    
```

合法值为：00-59 00-23 01-31 01-12 0-6 (0 is sunday) 

除了数字还有几个特殊的符号："*"、"/"和"-"、","

- *代表所有的取值范围内的数字
- "/"代表每的意思,"/5"表示每5个单位
- "-"代表从某个数字到某个数字
- ","分开几个离散的数字

**注**：commands 注意以下几点

- 要是存在文件，要写绝对路径
- 即使是打印也不会显示在显示屏，在后台运行，最好重定向日志

[回到顶部](https://www.cnblogs.com/kaituorensheng/p/4494321.html#_labelTop)

### 3. 编辑crontab文件

```html
EDITOR=vi



export EDITOR



crontab -e
```

[回到顶部](https://www.cnblogs.com/kaituorensheng/p/4494321.html#_labelTop)

### **4. 流程举例**

**step1**：写cron脚本文件，命名为crontest.cron。

15,30,45,59 * * * * echo "xgmtest....."   表示，每隔15分钟，执行一次打印命令 

**step2**：添加定时任务。执行命令

```html
crontab /home/del/crontest.cron >~/log
```

**step3**："crontab -l" 查看定时任务是否成功或者检测/var/spool/cron下是否生成对应cron脚本

```html
crontab -l
```

结果程序会每个15分钟往脚本里写一次“xgmtest.....”

[回到顶部](https://www.cnblogs.com/kaituorensheng/p/4494321.html#_labelTop)

### 5. 几个例子

```html
每天早上6点 

0 6 * * * echo "Good morning." >> /tmp/test.txt //注意单纯echo，从屏幕上看不到任何输出，因为cron把任何输出都email到root的信箱了。


每分钟输出一次
*/1 * * * * echo "Good morning." >> /tmp/test.txt //注意单纯echo，从屏幕上看不到任何输出，因为cron把任何输出都email到root的信箱了。

 



每两个小时(第一个为15，指明没两个小时的第15min中执行一次) 



15 */2 * * * echo "Have a break now." >> /tmp/test.txt  



 



晚上11点到早上8点之间每两个小时和早上八点 



0 23-7/2，8 * * * echo "Have a good dream" >> /tmp/test.txt



 



每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点 



0 11 4 * 1-3 command line



 



1月1日早上4点 



0 4 1 1 * command line



 



每小时（第一分钟）执行/etc/cron.hourly内的脚本



01 * * * * root run-parts /etc/cron.hourly



 



每天（凌晨4：02）执行/etc/cron.daily内的脚本



02 4 * * * root run-parts /etc/cron.daily 



 



每星期（周日凌晨4：22）执行/etc/cron.weekly内的脚本



22 4 * * 0 root run-parts /etc/cron.weekly 



 



每月（1号凌晨4：42）去执行/etc/cron.monthly内的脚本 



42 4 1 * * root run-parts /etc/cron.monthly 



 



注意:  "run-parts"这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是文件夹名。 　 



 



每天的下午4点、5点、6点的5 min、15 min、25 min、35 min、45 min、55 min时执行命令。 



5，15，25，35，45，55 16，17，18 * * * command



 



每周一，三，五的下午3：00系统进入维护状态，重新启动系统。



00 15 * *1，3，5 shutdown -r +5



 



每小时的10分，40分执行用户目录下的innd/bbslin这个指令： 



10，40 * * * * innd/bbslink 



 



每小时的1分执行用户目录下的bin/account这个指令： 



1 * * * * bin/account



 



每天早晨三点二十分执行用户目录下如下所示的两个指令（每个指令以;分隔）： 



203 * * * （/bin/rm -f expire.ls logins.bad;bin/expire$#@62;expire.1st）　　



 



每年的一月和四月，4号到9号的3点12分和3点55分执行/bin/rm -f expire.1st这个指令，并把结果添加在mm.txt这个文件之后（mm.txt文件位于用户自己的目录位置）。 



12,553 4-91,4 * /bin/rm -f expire.1st$#@62;$#@62;mm.txt 
参考：``https://www.cnblogs.com/kaituorensheng/p/4494321.html
```
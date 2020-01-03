# 如何将springBoot打包成jar文件运行

1.打开项目，然后右击项目选中‘Open Module Settings’进入project Structure（ 快捷键 Ctrl+Shift+Alt+S或者File->Project Structure ），选中‘Artifacts’，点击中间的绿色+号（Project Settings->Artifacts->JAR->From modules with dependencies ），如下图所示

2.弹出‘Create JAR from Modules’，选择‘Main Class’

然后点击OK，设置完毕

3.开始打包，点击右侧的Maven Projects，打开LIfecycle先点击clean然后点击package，生成target文件夹，里面有一个以项目名命名加版本号的jar文件，至此打包完成。进入jar所在的文件夹运行java -jar demoa-0.0.1-SNAPSHOT.jar，项目就能运行。



附加：

之前一个测试项目按照上面的步骤是能正常生成jar的，但是这个项目一直生成不了，参考了另一篇文章，直接用命令生成jar
cd 项目跟目录（和pom.xml同级）

mvn clean package或者在Terminal框内输入命令

作者：sinat_33201781 
来源：CSDN 
原文：https://blog.csdn.net/sinat_33201781/article/details/80264828 
版权声明：本文为博主原创文章，转载请附上博文链接！
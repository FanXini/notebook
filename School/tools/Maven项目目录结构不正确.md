# eclipse创建maven项目时无src/main/java,src/main/resource,src/test.java文件夹的解决方案

\1. 新建maven项目（选择Maven Project）

![img](https://img-blog.csdn.net/20180511101301813?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
\2. 点击下一步

\3. 在Filter中输入webapp（选择maven-archetype-webapp，然后Next）


![img](https://img-blog.csdn.net/20180511101321570?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
\4. Group Id--主项目名



Artifact Id--本项目名


package--包名可自定义，也暂时不输入，之后再自己创建；

选择finish；


![img](https://img-blog.csdn.net/201805111013424?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
\5. 此时的工程项目结构并不完整，需将将其编程web项目；

（本地安装tomcat6，web module需要设为2.5，本地设为jdk1.6）

右击项目选择properties,再选择Project Facets，

![img](https://img-blog.csdn.net/20180511101403166?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
途中的默认值需要按需求修改，

不过在修改过程中发现，java设为1.6之后，Dynamic Web Module无法设置为2.5？？

解决方案：打开项目的根目录../springmvc/.setting/org.eclipse.wst.common.project.facet.core.xml

打开该文件后找到  <installed facet="jst.web" version="2.3"/>

将version修改为2.5，保存后，在eclipse右击项目选择Refresh进行刷新；即修改完成！

此时的项目结构如图：

![img](https://img-blog.csdn.net/20180511101425927?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这种结构不满足maven项目要求（有的人可能出现这种问题，有的人可能不会，一般情况下是有src/main/java,src/main/resource,src/test.java三个文件夹的）

解决方案：右击项目，选择Properties，点击Java Build Path；切换至Libraries下，

![img](https://img-blog.csdn.net/20180511101441112?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
双击 JRE System Library（javaSE-1.6）；将System library切换至Workspace default JRE（即切换jre为工作空间默认）如图：

![img](https://img-blog.csdn.net/20180511101454774?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
点击finish；在点击finish！

项目结构现在正常了！

![img](https://img-blog.csdn.net/20180511101509145?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
\6. 最后还要将发布文件确定

右击项目选择Properties选择Deployment Assemby；将/src/test/java文件夹remove掉！

![img](https://img-blog.csdn.net/20180511101526764?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1YW5zYW12ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
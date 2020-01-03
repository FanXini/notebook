# javaWeb工程 添加不到tomcat中 解决办法

有时候，我们会遇到javaWeb 工程导入本地eclipse。然而，有可能遇到lib包错误或者tomcat版本不一致或者jar不正确导致导入失败。

在做的过程中，通过百度查找到一些解决办法供大家解惑。

在eclipse中导入之前做个项目，想运行起来看看，发现导入之后没法部署。 

## 方法1 

![img](E:/Users/fanxin/Documents/%E6%9C%89%E9%81%93%E4%BA%91%E7%AC%94%E8%AE%B0/472843326@qq.com/4fc9e1f903df430397aec6504817184b/313173202140.png)

勾选上面三项并选择相应的值后就变成web项目，可以部署在tomcat上了。

## 方法2

如果上面参数值没法修改，修改的时候报错。则可以修改一个文件来达到目的。

修改文件：D:\eclipse工作空间\项目名称\.settings文件夹下  org.eclipse.wst.common.project.facet.core.xml 文件。

比如要导入的工程是采用tomcat8发布的、需要降版本到Tomcat7 则需要修改

```java
<installed facet="jst.web" version="3.0"/>    <!--将3.0改为2.5 -->

<?xml version="1.0" encoding="UTF-8"?>

<faceted-project>

  <fixed facet="wst.jsdt.web"/>

  <fixed facet="java"/>

  <fixed facet="jst.web"/>

  <installed facet="java" version="1.6"/>

  <installed facet="jst.web" version="3.0"/>

  <installed facet="wst.jsdt.web" version="1.0"/>

</faceted-project>
```

此时。我们通过Eclipse 成功把项目添加到tomcat下
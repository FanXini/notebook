## Eclipse/Intellij IDEA查看jar包的源码和注释 原



[Eclipse](https://my.oschina.net/u/1166518?q=Eclipse)[IntelliJ IDEA](https://my.oschina.net/u/1166518?q=IntelliJ IDEA)

[为什么80%的码农都做不了架构师？>>> ](https://my.oschina.net/u/2663968/blog/3051541)![img](https://www.oschina.net/img/hot3.png)

Eclipse/Intellij IDEA看源码还好，itellij idea自带反编译器，eclipse装个插件即可，看注释就麻烦了，总不能去找api文档吧！但是需要注意的是，反编译的代码没有详细注释，而且泛型以及某些注解是没有的（因为在编译打jar包时已经抹去了），所以看源码很有必要。



## 1. 方法一： Maven命令方式



### 1.1 下载sources

```
mvn dependency:sources
```



### 1.2 下载javadocs

```
mvn dependency:resolve -Dclassifier=javadoc
```

运行以上命令，

- 可以在pom.xml所在命令下打开命令行
- 可以在idea中运行，如下图：

![img](https://static.oschina.net/uploads/space/2017/1202/162534_RoVO_1166518.png)

下载后应该就可以查看。

 



## 2. 方法二： IDEA方式

![img](https://static.oschina.net/uploads/space/2017/1202/165731_RWPx_1166518.png)

下载后应该就可以查看。

- 点击File->Setting...->Maven->Importing->勾选Automatically中的Sources和Documentation->OK，可以自动下载。

 



## 3. 方法三： 手动方式



### 3.1 下载

到官网或者maven仓库下载sources和javadocs( 例如：maven mysql 百度一下，肯定会出现仓库地址，找某一个版本下载即可)



### 3.2 Eclipse

- 右击项目->Build Path->Configure Build Path...->选择Java Build Path的Libraries选项卡->找到相应的jar包->Source attachment->Edit->External location->External File...选择对应的源码包即可。
- 鼠标移动到方法上面停留一会儿，便会出现方式注释。



### 3.3 Intellij idea

- 点击工具栏模块设置图标(Project Structure) 【或 File->Project Structure...】-> Libraries -> + -> Java->选择jar源码包或者源码包所在文件夹即可：             
- 查看方法注释，点击进入源码即可，若想和eclipse一样鼠标停留即可出现注释提示，开启方法为：Preferences->Editor->General->Other->Show quick documentation on mouse move 钩上。
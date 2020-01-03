# IDEA中Java错误：Usage of API documented as @since 1.8+ less...

2018年03月05日 20:34:42

 

我是康小小

 

阅读数：1754

更多

个人分类： [Java](https://blog.csdn.net/u010429286/article/category/6452204)



 版权声明：本文为博主康小小原创文章，转载请声明转载出处：http://blog.csdn.net/u010429286 https://blog.csdn.net/u010429286/article/details/79450649

IDEA出现错误

Usage of API documented as @since 1.8+ less… (⌘F1) This inspection finds all usages of methods that have @since tag in their document….

我以为是JDK版本的问题，但是在External Libraries中也能显示出JDK8的系统jar包。解决办法：

File ->Project Structure->Project Settings -> Modules -> Module名称 -> Sources -> Language Level->可以看到当前并不是8，（这里我的是5.0，1.5之前是以1.3，1.4显示）更改一下选择就行。 
图示： 
![第一步](https://img-blog.csdn.net/20180305203232501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQyOTI4Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 
![第二步](https://img-blog.csdn.net/20180305203337193?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQyOTI4Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

[“*Usage* of *API* *documented* as @*since* *1.8+*”报错的解决办法](https://blog.csdn.net/a499477783/article/details/78967586)
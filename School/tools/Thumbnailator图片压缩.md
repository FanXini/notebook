# Java使用google开源工具Thumbnailator实现图片压缩

2017年12月06日 11:48:06 [零度anngle](https://me.csdn.net/zmx729618) 阅读数：9042

[转自](https://blog.csdn.net/zmx729618/article/details/78729049)

## 前言

作为靠谱的java服务端程序员，图片这个事情一直是个头疼的事情。

现在很多网站上，都有上传图片这个功能，而图片对于现在的很多手机来说，拍摄出来的都是高清图片，分辨率也是相当的高，当然占用的存储空间也就大了。问题也就产生了，你每个用户都上传个3M的图片怎么办？

但是显然现在硬盘的存放空间是不值钱的，1T、2T随便来，存放是能用钱解决的问题。

但是网速太值钱了，用户如果天天加载你的网页加载个半天，就是因为图片太大导致的那就不是钱能解决的问题了。

因为用户的网络环境你是不可控制的。所以你只能考虑压缩图片的质量从而保证网站打开的速度。

##  压缩的要求

图片压缩，在我的想法里面有下面几个要求。

1、压缩程度可控制，想压缩成多小就多小。

2、压缩之后图片尽可能的不失真。

3、压缩速度要快。

4、代码简单，依赖较少。

 



## 实现

然后带着这些要求去寻找，找到了Thumbnailator，一个[google](https://www.baidu.com/s?wd=google&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)使用的开源的工具类。

这个工具类满足了上面所说的所有的要求。

同时对于图片的处理还有了别的方法，如旋转，裁切，加水印等等。

在github上面的地址是：<https://github.com/coobird/thumbnailator>

maven的地址

```
<dependency>
   <groupId>net.coobird</groupId>
   <artifactId>thumbnailator</artifactId>
   <version>0.4.8</version>
</dependency>
```

使用起来特别的简单：一行代码就搞定了

Thumbnails.of("原图文件的路径") 
​        .scale(1f) 
​        .outputQuality(0.5f) 
​        .toFile("压缩后文件的路径");

其中的scale是可以指定图片的大小，值在0到1之间，1f就是原图大小，0.5就是原图的一半大小，这里的大小是指图片的[长宽](https://www.baidu.com/s?wd=%E9%95%BF%E5%AE%BD&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)。

而outputQuality是图片的质量，值也是在0到1，越接近于1质量越好，越接近于0质量越差。

对于压缩图片来说上面就已经足够了。

 



## 优点

1、简单容易使用。

2、压缩[图片效果](https://www.baidu.com/s?wd=%E5%9B%BE%E7%89%87%E6%95%88%E6%9E%9C&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)很好。如下：其中100是原图，50就是0.5f

[![image](http://images2017.cnblogs.com/blog/899548/201708/899548-20170822155432855-1068708418.png)](http://images2017.cnblogs.com/blog/899548/201708/899548-20170822155431543-1688462485.png)

3、图片质量不错下面是0.25f和原图的对比

[![image](http://images2017.cnblogs.com/blog/899548/201708/899548-20170822155437339-844342997.png)](http://images2017.cnblogs.com/blog/899548/201708/899548-20170822155435652-289728822.png)

[![image](http://images2017.cnblogs.com/blog/899548/201708/899548-20170822155439824-1968569578.png)](http://images2017.cnblogs.com/blog/899548/201708/899548-20170822155438511-1942715261.png)

上面是压缩过后的，下面是原图、看出来了吗？

 



## 其他功能

最后附上其他功能使用的简单例子

```java
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;

import javax.imageio.ImageIO;

import net.coobird.thumbnailator.Thumbnails;
import net.coobird.thumbnailator.geometry.Positions;

public class ThumbnailatorTest {

    /**
     * 
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        ThumbnailatorTest thumbnailatorTest = new ThumbnailatorTest();
        thumbnailatorTest.test1();
        thumbnailatorTest.test2();
        thumbnailatorTest.test3();
        thumbnailatorTest.test4();
        thumbnailatorTest.test5();
        thumbnailatorTest.test6();
        thumbnailatorTest.test7();
        thumbnailatorTest.test8();
        thumbnailatorTest.test9();
    }

    /**
     * 指定大小进行缩放
     * 
     * @throws IOException
     */
    private void test1() throws IOException {
        /*
         * size(width,height) 若图片横比200小，高比300小，不变
         * 若图片横比200小，高比300大，高缩小到300，图片比例不变 若图片横比200大，高比300小，横缩小到200，图片比例不变
         * 若图片横比200大，高比300大，图片按比例缩小，横为200或高为300
         */
        Thumbnails.of("images/test.jpg").size(200, 300).toFile("C:/image_200x300.jpg");
        Thumbnails.of("images/test.jpg").size(2560, 2048).toFile("C:/image_2560x2048.jpg");
    }

    /**
     * 按照比例进行缩放
     * 
     * @throws IOException
     */
    private void test2() throws IOException {
        /**
         * scale(比例)
         */
        Thumbnails.of("images/test.jpg").scale(0.25f).toFile("C:/image_25%.jpg");
        Thumbnails.of("images/test.jpg").scale(1.10f).toFile("C:/image_110%.jpg");
    }

    /**
     * 不按照比例，指定大小进行缩放
     * 
     * @throws IOException
     */
    private void test3() throws IOException {
        /**
         * keepAspectRatio(false) 默认是按照比例缩放的
         */
        Thumbnails.of("images/test.jpg").size(120, 120).keepAspectRatio(false).toFile("C:/image_120x120.jpg");
    }

    /**
     * 旋转
     * 
     * @throws IOException
     */
    private void test4() throws IOException {
        /**
         * rotate(角度),正数：顺时针 负数：逆时针
         */
        Thumbnails.of("images/test.jpg").size(1280, 1024).rotate(90).toFile("C:/image+90.jpg");
        Thumbnails.of("images/test.jpg").size(1280, 1024).rotate(-90).toFile("C:/iamge-90.jpg");
    }

    /**
     * 水印
     * 
     * @throws IOException
     */
    private void test5() throws IOException {
        /**
         * watermark(位置，水印图，透明度)
         */
        Thumbnails.of("images/test.jpg").size(1280, 1024).watermark(Positions.BOTTOM_RIGHT, ImageIO.read(new File("images/watermark.png")), 0.5f)
                .outputQuality(0.8f).toFile("C:/image_watermark_bottom_right.jpg");
        Thumbnails.of("images/test.jpg").size(1280, 1024).watermark(Positions.CENTER, ImageIO.read(new File("images/watermark.png")), 0.5f)
                .outputQuality(0.8f).toFile("C:/image_watermark_center.jpg");
    }

    /**
     * 裁剪
     * 
     * @throws IOException
     */
    private void test6() throws IOException {
        /**
         * 图片中心400*400的区域
         */
        Thumbnails.of("images/test.jpg").sourceRegion(Positions.CENTER, 400, 400).size(200, 200).keepAspectRatio(false)
                .toFile("C:/image_region_center.jpg");
        /**
         * 图片右下400*400的区域
         */
        Thumbnails.of("images/test.jpg").sourceRegion(Positions.BOTTOM_RIGHT, 400, 400).size(200, 200).keepAspectRatio(false)
                .toFile("C:/image_region_bootom_right.jpg");
        /**
         * 指定坐标
         */
        Thumbnails.of("images/test.jpg").sourceRegion(600, 500, 400, 400).size(200, 200).keepAspectRatio(false).toFile("C:/image_region_coord.jpg");
    }

    /**
     * 转化图像格式
     * 
     * @throws IOException
     */
    private void test7() throws IOException {
        /**
         * outputFormat(图像格式)
         */
        Thumbnails.of("images/test.jpg").size(1280, 1024).outputFormat("png").toFile("C:/image_1280x1024.png");
        Thumbnails.of("images/test.jpg").size(1280, 1024).outputFormat("gif").toFile("C:/image_1280x1024.gif");
    }

    /**
     * 输出到OutputStream
     * 
     * @throws IOException
     */
    private void test8() throws IOException {
        /**
         * toOutputStream(流对象)
         */
        OutputStream os = new FileOutputStream("C:/image_1280x1024_OutputStream.png");
        Thumbnails.of("images/test.jpg").size(1280, 1024).toOutputStream(os);
    }

    /**
     * 输出到BufferedImage
     * 
     * @throws IOException
     */
    private void test9() throws IOException {
        /**
         * asBufferedImage() 返回BufferedImage
         */
        BufferedImage thumbnail = Thumbnails.of("images/test.jpg").size(1280, 1024).asBufferedImage();
        ImageIO.write(thumbnail, "jpg", new File("C:/image_1280x1024_BufferedImage.jpg"));
    }
}
```

其他的具体方法细节可以自己去查看官方的API或者网络上的其他资源。
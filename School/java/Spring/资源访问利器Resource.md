# Spring3.0第四讲:资源访问利器-Resource

[![img](https://s2.51cto.com//wyfs02/M00/85/42/wKiom1eeuyKROzElAAAcTODmtOk620_middle.jpg)](http://blog.51cto.com/zangyanan)

[摘自](http://blog.51cto.com/zangyanan/1842407)



  JDK提供的访问资源的类（如java.net.URL、File等）并不能很好的满足各种底层资源的访问，比如缺少从类路径以及web容器上下文获取资源的操作类，因此，Spring设计了一个Resource接口，他提供了更强访问底层资源的能力，先来看看Resource接口的主要方法：

- exists()：用于判断对应的资源是否真的存在。

- isReadable()：用于判断对应资源的内容是否可读。需要注意的是当其结果为true的时候，其内容未必真的可读，但如果返回false，则其内容必定不可读。

- isOpen()：用于判断当前资源是否代表一个已打开的输入流，如果结果为true，则表示当前资源的输入流不可多次读取，而且在读取以后需要对它进行关闭，以防止内存泄露。该方法主要针对于InputStreamResource，实现类中只有它的返回结果为true，其他都为false。

- getURL()：返回当前资源对应的URL。如果当前资源不能解析为一个URL则会抛出异常。如ByteArrayResource就不能解析为一个URL。

- getFile()：返回当前资源对应的File。如果当前资源不能以绝对路径解析为一个File则会抛出异常。如ByteArrayResource就不能解析为一个File。

- getInputStream()：获取当前资源代表的输入流。除了InputStreamResource以外，其它Resource实现类每次调用getInputStream()方法都将返回一个全新的InputStream。


   Resource在Spring中占有重要作用，Spring用它来读取配置文件、国际化属性文件资源等下面来看一下Resource实现方法：

  [![wKioL1e-XsiQygaLAAA44UQSlGA727.jpg](http://s1.51cto.com/wyfs02/M02/86/69/wKioL1e-XsiQygaLAAA44UQSlGA727.jpg)](http://s1.51cto.com/wyfs02/M02/86/69/wKioL1e-XsiQygaLAAA44UQSlGA727.jpg)

- ByteArrayResource：二进制数组表示的资源，二进制数组源可以在内存中通过程序构造。

- ClassPathResource：类路径下的资源，资源以相对于类路径的方式表示，如：new ClassPathResource("com/baobaotao/beanfactory/bean.xml")。

- FileSystemResource：文件系统资源，资源以文件系统路径的方式表示，如：new FileSystemResource("c:\\beans.xml")。

- InputSteamResource：以输入流返回表示的资源。

- ServletContextResource：为访问Web容器上下文中的资源而设计的类，负责从Web应用根目录中加载资源，它支持以流和Url的方式访问，在WAR解包的情况下，也可以通过File的方式访问，该类还可以直接从JAR包中访问资源。

- UrlResource：Url封装了java.net.URL，它使用户能够访问任何可以通过URL表示的资源，如文件系统的资源、HTTP资源、FTP资源等。

   有了这个抽象的资源类后，我们就可以将Spring的配置信息放置在任何地方（如数据库、LDAP中），只要最终可以通过Resource接口返回配置信息就可以了。

注意：Spring的Resource可以脱离Spring框架下单独使用。

下面我们来看看Resource能给我们带来什么？

**访问文件资源**

   现在我们假设Web项目类路径下有一个文件，那么我们应该怎么去读取呢？

 ①FileSystemResource通过系统的绝对路径去读取。

 ②ClassPathResource以类路径方式进行访问。

 ③ServletContextResource以相对Web应用根目录的方式读取。

相比较JDK提供的访问资源的方法，Resource可以让我们根据需要进行更多的选择，下面我们来看一下2种方法实现的代码（项目名称为Spring在其src下有个conf，里面有个test.txt的文件）：

```java
package test.com.gloryscience.service;

import java.io.IOException;
import java.io.InputStream;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;

public class ResourceTest {
	public static void main(String[] args) {
		try {
		String filepath="F:/guoxiangworkspace/Spring/WebContent/WEB-INF/classes/conf/test.txt";
		//以系统文件路径的方式加载文件
		Resource fileresource=new FileSystemResource(filepath);
		//以class类路径加载文件
		Resource classsource=new ClassPathResource("/conf/test.txt");
		InputStream is2=classsource.getInputStream();
		InputStream is1=fileresource.getInputStream();
		System.out.println("通过FileSystemResource："+fileresource.getFilename());
		System.out.println("通过ClassPathResource："+classsource.getFilename());
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```



​     在获取资源后，您就可以通过 Resource 接口定义的多个方法访问文件的数据和其它的信息：如您可以通过 getFileName() 获取文件名，通过 getFile() 获取资源对应的 File 对象，通过 getInputStream() 直接获取文件的输入流。此外，您还可以通过 createRelative(String relativePath) 在资源相对地址上创建新的资源。

在 Web 应用中，您还可以通过 ServletContextResource 以相对于 Web 应用根目录的方式访问文件资源，如下所示：

```java
 <%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%> 
 <jsp:directive.page import="
    org.springframework.web.context.support.ServletContextResource"/> 
 <jsp:directive.page import="org.springframework.core.io.Resource"/> 
 <% 
    // ① 注意文件资源地址以相对于 Web 应用根路径的方式表示
    Resource res3 = new ServletContextResource(application, 
        "/WEB-INF/classes/conf/file1.txt"); 
    out.print(res3.getFilename()); 
 %>
```



对于位于远程服务器（Web 服务器或 FTP 服务器）的文件资源，您则可以方便地通过 UrlResource 进行访问。

为了方便访问不同类型的资源，您必须使用相应的 Resource 实现类，是否可以在不显式使用 Resource 实现类的情况下，仅根据带特殊前缀的资源地址直接加载文件资源呢？ Spring 提供了一个 ResourceUtils 工具类，它支持“classpath:”和“file:”的地址前缀，它能够从指定的地址加载文件资源，请看下面的例子：

```java
 import org.springframework.util.ResourceUtils; 
 public class ResourceUtilsExample { 
    public static void main(String[] args) throws Throwable{ 
        File clsFile = ResourceUtils.getFile("classpath:conf/file1.txt"); 
        System.out.println(clsFile.isFile()); 

        String httpFilePath = "file:D:/masterSpring/chapter23/src/conf/file1.txt"; 
        File httpFile = ResourceUtils.getFile(httpFilePath); 
        System.out.println(httpFile.isFile());        
    } 
 }
```





**本地化文件资源**

本地化文件资源是一组通过本地化标识名进行特殊命名的文件，Spring 提供的 LocalizedResourceHelper 允许通过文件资源基名和本地化实体获取匹配的本地化文件资源并以 Resource 对象返回。假设在类路径的 i18n 目录下，拥有一组基名为 message 的本地化文件资源，我们通过以下实例演示获取对应中国大陆和美国的本地化文件资源：

```java
import java.util.Locale; 
 import org.springframework.core.io.Resource; 
 import org.springframework.core.io.support.LocalizedResourceHelper; 
 public class LocaleResourceTest { 
    public static void main(String[] args) { 
        LocalizedResourceHelper lrHalper = new LocalizedResourceHelper(); 
        // ① 获取对应美国的本地化文件资源
        Resource msg_us = lrHalper.findLocalizedResource("i18n/message", ".properties", 
        Locale.US); 
        // ② 获取对应中国大陆的本地化文件资源
        Resource msg_cn = lrHalper.findLocalizedResource("i18n/message", ".properties", 
        Locale.CHINA); 
        System.out.println("fileName(us):"+msg_us.getFilename()); 
        System.out.println("fileName(cn):"+msg_cn.getFilename()); 
    } 
 }
```



**文件操作**

在使用各种 Resource 接口的实现类加载文件资源后，经常需要对文件资源进行读取、拷贝、转存等不同类型的操作。您可以通过 Resource 接口所提供了方法完成这些功能，不过在大多数情况下，通过 Spring 为 Resource 所配备的工具类完成文件资源的操作将更加方便。

```java
 import java.io.ByteArrayOutputStream; 
 import java.io.File; 
 import java.io.FileReader; 
 import java.io.OutputStream; 
 import org.springframework.core.io.ClassPathResource; 
 import org.springframework.core.io.Resource; 
 import org.springframework.util.FileCopyUtils; 
 public class FileCopyUtilsExample { 
    public static void main(String[] args) throws Throwable { 
        Resource res = new ClassPathResource("conf/file1.txt"); 
        // ① 将文件内容拷贝到一个 byte[] 中
        byte[] fileData = FileCopyUtils.copyToByteArray(res.getFile()); 
        // ② 将文件内容拷贝到一个 String 中
        String fileStr = FileCopyUtils.copyToString(new FileReader(res.getFile())); 
        // ③ 将文件内容拷贝到另一个目标文件
        FileCopyUtils.copy(res.getFile(), 
        new File(res.getFile().getParent()+ "/file2.txt")); 

        // ④ 将文件内容拷贝到一个输出流中
        OutputStream os = new ByteArrayOutputStream(); 
        FileCopyUtils.copy(res.getInputStream(), os); 
    } 
 }
```



往往我们都通过直接操作 InputStream 读取文件的内容，但是流操作的代码是比较底层的，代码的面向对象性并不强。通过 FileCopyUtils 读取和拷贝文件内容易于操作且相当直观。如在 ① 处，我们通过 FileCopyUtils 的 copyToByteArray(File in) 方法就可以直接将文件内容读到一个 byte[] 中；另一个可用的方法是 copyToByteArray(InputStream in)，它将输入流读取到一个 byte[] 中。

如果是文本文件，您可能希望将文件内容读取到 String 中，此时您可以使用 copyToString(Reader in) 方法，如 ② 所示。使用 FileReader 对 File 进行封装，或使用 InputStreamReader 对 InputStream 进行封装就可以了。

FileCopyUtils 还提供了多个将文件内容拷贝到各种目标对象中的方法，这些方法包括：



​                           方法                                                                  说明



static void copy(byte[] in, File out)                              将 byte[] 拷贝到一个文件中    

static void copy(byte[] in, OutputStream out)             将 byte[] 拷贝到一个输出流中    

static int copy(File in, File out)                                     将文件拷贝到另一个文件中    

static int copy(InputStream in, OutputStream out)    将输入流拷贝到输出流中    

static int copy(Reader in, Writer out)                          将 Reader 读取的内容拷贝到 Writer 指向目标输出中    

static void copy(String in, Writer out)                         将字符串拷贝到一个 Writer 指向的目标中    

在实例中，我们虽然使用 Resource 加载文件资源，但 FileCopyUtils 本身和 Resource 没有任何关系，您完全可以在基于 JDK I/O API 的程序中使用这个工具类。

**属性资源加载**

我们知道可以通过 java.util.Properties 的 load(InputStream inStream) 方法从一个输入流中加载属性资源。Spring 提供的 PropertiesLoaderUtils 允许您直接通过基于类路径的文件地址加载属性资源，请看下面的例子：

```java
 import java.util.Properties; 
 import org.springframework.core.io.support.PropertiesLoaderUtils; 
 public class PropertiesLoaderUtilsExample { 
    public static void main(String[] args) throws Throwable {    
        // ① jdbc.properties 是位于类路径下的文件
        Properties props = PropertiesLoaderUtils.loadAllProperties("jdbc.properties"); 
        System.out.println(props.getProperty("jdbc.driverClassName")); 
    } 
 }
```





一般情况下，应用程序的属性文件都放置在类路径下，所以 PropertiesLoaderUtils 比之于 Properties#load(InputStream inStream) 方法显然具有更强的实用性。此外，PropertiesLoaderUtils 还可以直接从 Resource 对象中加载属性资源：

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| static Properties loadProperties(Resource resource)          | 从 Resource 中加载属性                                       |
| static void fillProperties(Properties props, Resource resource) | 将 Resource 中的属性数据添加到一个已经存在的 Properties 对象中 |

**特殊编码资源**

当您使用 Resource 实现类加载文件资源时，它默认采用操作系统的编码格式。如果文件资源采用了特殊的编码格式（如 UTF-8），则在读取资源内容时必须事先通过 EncodedResource 指定编码格式，否则将会产生中文乱码的问题。

```java
import org.springframework.core.io.ClassPathResource; 
 import org.springframework.core.io.Resource; 
 import org.springframework.core.io.support.EncodedResource; 
 import org.springframework.util.FileCopyUtils; 
 public class EncodedResourceExample { 
        public static void main(String[] args) throws Throwable  { 
            Resource res = new ClassPathResource("conf/file1.txt"); 
            // ① 指定文件资源对应的编码格式（UTF-8）
            EncodedResource encRes = new EncodedResource(res,"UTF-8"); 
            // ② 这样才能正确读取文件的内容，而不会出现乱码
            String content  = FileCopyUtils.copyToString(encRes.getReader()); 
            System.out.println(content);  
    } 
 }
```





EncodedResource 拥有一个 getResource() 方法获取 Resource，但该方法返回的是通过构造函数传入的原 Resource 对象，所以必须通过 EncodedResource#getReader() 获取应用编码后的 Reader 对象，然后再通过该 Reader 读取文件的内容。
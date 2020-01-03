---
typora-copy-images-to: ..\..\img
---

# SpringMVC的高级技术

## 1.处理MuitiPart的数据

该技术用于上传图片、文档等类型的数据。

### 1.1 配置muitipart解析器

从Spring3.1开始，内置了两种解析器供使用者选择：

- CommonsMultipartResolver：使用Jakarta CommonsFileUpload解析multipart请求；
- StandardServletMultipartResolver：依赖于Servlet 3.0对multipart请求的支持（始于Spring 3.1）。

也就是说，在Spring3.1以下，只能使用CommonsMultipartResolver。

这里使用StandardServletMultipartResolver举例。

首先在Spring上下文中声明StandardServletMultipartResolver的bean,本文在RootConfig.java中配置。

```java
package config;

import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.support.StandardServletMultipartResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configurable
@ComponentScan(basePackages = {"entity"},excludeFilters= {@Filter(type=FilterType.ANNOTATION,value=EnableWebMvc.class)})
public class RootConfig {
	
	@Bean
	public MultipartResolver multipartResolver() {
		return new StandardServletMultipartResolver();
	}
}
```

我们发现，构造器是不带参数的，如果我们想限制上传文件的大小并且设置文件的临时存放路径(如果路径不配置的话，则解析器无法工作)，则要在Servlet中配置。

如果我们采用Servlet初始化类的方式来配置DispatcherServlet的话，这个初始化类应该已经实现了
WebApplicationInitializer，那我们可以在Servlet registration上调用setMultipartConfig()方法，传入一个MultipartConfig-Element实例。如下是最基本的DispatcherServlet multipart配置，它将临时路径设置为“/tmp/spittr/uploads”：

![1546683872123](..\..\..\..\img\1546683872123.png)

如果我们配置DispatcherServlet的Servlet初始化类继承了AbstractAnnotationConfigDispatcherServletInitializer类，则只需要重写customizeRegistration()方法即可:

```java
@Override
	protected void customizeRegistration(javax.servlet.ServletRegistration.Dynamic registration) {
		registration.setMultipartConfig(new MultipartConfigElement("F:\\WebProjectWorkspace\\Redis\\src\\main\\webapp\\temp",10485760,20971520,0));
	}

```

在这里，为了把临时缓存目录放入了项目内的文件夹，因此要设置绝对路径，而且设置了额外的三个参数，分别表示：

- 上传文件的最大容量（以字节为单位）。默认是没有限制的。
- 整个multipart请求的最大容量（以字节为单位），不会关心有多少个part以及每个part的大小。默认是没有限制的。
- 在上传的过程中，如果文件大小达到了一个指定最大容量（以字节为单位），将会写入到临时文件路径中。默认值为0，也就是所有上传的文件都会写入到磁盘上。

### 1.2 处理multipart请求

#### 1.2.1 使用< form>元素上传

以JSP作为view为例子

```html
<div class="row">
				<form action="img/test" method="POST"       enctype="multipart/form-data">
					<input type="file" name="img" />
					<input type="submit">
				</form>
</div>
```

&emsp;&emsp;<form>标签现在将enctype属性设置为multipart/formdata，这会告诉浏览器以multipart数据的形式提交表单，而不是以表单数据的形式进行提交。在multipart中，每个输入域都会对应一个part。

看对应的Controller处理方法：

```java
package Controller;

import java.io.File;
import java.io.IOException;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.multipart.MultipartFile;

@Controller
@RequestMapping("img")
public class ImgController {
	
	@RequestMapping("/test")
	public String processImg(MultipartFile img) throws IllegalStateException, IOException {
		//获取文件原始名称
        String originalFileName = img.getOriginalFilename();
        //获取文件后缀
        String imgType = originalFileName.substring(originalFileName.lastIndexOf("."));
        File newFile = new File(originalFileName+imgType);
        // 将内存中的数据写入磁盘
        img.transferTo(newFile);
        return "login"; //跳转到login.jsp
	}
}

```

在上面的processImg方法中使用到了MultipartFile接口，这个接口由Spring提供，用来处理上传的文件。这个改接口的源代码：

```java
public interface MultipartFile
    extends InputStreamSource
{

    public abstract String getName();

    public abstract String getOriginalFilename();

    public abstract String getContentType();

    public abstract boolean isEmpty();

    public abstract long getSize();

    public abstract byte[] getBytes()
        throws IOException;

    public abstract InputStream getInputStream()
        throws IOException;

    public Resource getResource()
    {
        return new MultipartFileResource(this);
    }

    public abstract void transferTo(File file)
        throws IOException, IllegalStateException;

    public void transferTo(Path dest)
        throws IOException, IllegalStateException
    {
        FileCopyUtils.copy(getInputStream(), Files.newOutputStream(dest, new OpenOption[0]));
    }
}
```

#### 1.2.2 使用Ajax上传

已注册一个用户为例子：

```html
<div class="m-t" role="form" action="login.html">
				<div class="form-group">
					<input id="userName" type="text" class="form-control"
						placeholder="用户名(手机号码)" required="">
				</div>

				<div class="form-group">
					<input id="registerPass" type="password" class="form-control"
						placeholder="密码" required="" onblur="chkpwd(this);">
					<div id="chkResult"></div>
				</div>
				<div class="form-group">
					<input id="confirmPass" type="password" class="form-control"
						placeholder="请再次输入密码" required="">
				</div>

				<div class="form-group">
					<input type="file" id="headImage" name="headImage" />
				</div>

				<button id="register" class="btn btn-primary block full-width m-b">注册</button>
```

用户注册有3个属性：username,registerPass,和headImg。

注册<button>对应的js：

```javascript
$("#register").click(function() {
				var username = $("#userName").val();
				var registerPass = $("#registerPass").val();
				var confirmPass = $("#confirmPass").val();
				var image = document.getElementById("headImage").files[0];
				var formData=new FormData(); //创建form文件
				formData.append("image",image);
				formData.append("username",username);
				formData.append("password",registerPass);
				alert(formData)
				/* alert(registerName+" "+registerPass+" "+roleId+" "+siteId) */
				if (username.length == 0) {
					alert("用户名不能为空");
				} else if (username.length == 0) {
					alert("手机号码不能为空");
				} else if (registerPass.length == 0) {
					alert("密码不能为空");
				} else if (confirmPass.length == 0) {
					alert("请输入确认密码");
				} else if (registerPass != confirmPass) {
					alert("密码与确认密码不一致！");
				} else {
					$.ajax({
						type : "POST",
						url : "user/register",
						data : formData,
						processData:false, // jQuery不要去处理发送的数据
					contentType:false,//jQuery不要去设置Content-Type请求头
						async:false,
						dataType : "text",
						success : function(result) {
							if (result == "SUCCESS") {
								alert("注册成功,请耐心等待管理员审核");
							} else if (result == "DUPLICATE") {
								alert("该用户名已被注册");
							} else if (result == "INPUT") {
								alert("请检查注册信息输入是否正确");
							} else {
								alert("注册失败");
							}
						},
						error : function() {
							alert("fail")
						}
					})
				}
			});
```

因为现在要上传的数据不在是字符串类型，而是文件类型，所以不能再像以前那些使用如下的数据传送格式。

![1546685212179](..\..\..\..\img\1546685212179.png)

为了解决这个问题， 这里采用了FormData对象，关于FormData的使用方法这里不做叙述。同时，ajax的属性中

1. 在ajax中加上processData : false,

2. 在ajax中加上contentType : false,

3. 在ajax中加上async:false,

对应的Controller

```java
package Controller;

import java.io.File;
import java.io.IOException;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;

import entity.User;

@Controller
@RequestMapping("user")
public class UserController {

	private static Log log = LogFactory.getLog(UserController.class);

	@RequestMapping("/register")
	@ResponseBody
	public String test(HttpServletRequest request) {
		MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
		List<MultipartFile> filelist = multipartRequest.getFiles("image");
		MultipartFile image = filelist.get(0);
		String username=request.getParameter("username");
		String originalFileName = image.getOriginalFilename();
		// 新的图片名称
		String imgType = originalFileName.substring(originalFileName.lastIndexOf("."));
		// 新的图片
		File newFile = new File(username + imgType);
		// 将内存中的数据写入磁盘
		try {
			image.transferTo(newFile);
		} catch (IllegalStateException | IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return "SUCCESS";

	}
}
```


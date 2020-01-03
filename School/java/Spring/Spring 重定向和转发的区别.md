# SpringMVC中使用forward和redirect进行转发和重定向以及重定向时如何传参详解

[转自](http://blog.51cto.com/983836259/1877188)

[重定向的与请求转发的区别以及什么时候使用](https://blog.csdn.net/hagle_wang/article/details/79039715)

## 重定向与请求转发使用

前后两个页面 有数据传递 用请求转发，没有则用重定向。 
比如servlet查询了数据需要在页面显示，就用请求转发。 
比如servlet做了update操作跳转到其他页面，就用重定向。

## 详解

如题所示，在SpringMVC中可以使用forward和redirect关键字在Controller中对原请求进行转发或重定向到其他的Controller。比如可以这样使用：

```java
package cn.zifangsky.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import cn.zifangsky.model.User;

@Controller
public class TestController {
	
	@RequestMapping("/user/index.html")
	public ModelAndView userIndex() {
		return new ModelAndView("user/index");
	}
	
	@RequestMapping("/testForward.html")
	public ModelAndView testForward(@RequestParam("username") String username){
		ModelAndView mAndView = new ModelAndView("forward:/user/index.html");
		
		User user = new User();
		user.setName(username);
		mAndView.addObject("user", user);
		
		return mAndView;
	}
	
	@RequestMapping("/testRedirect.html")
	public ModelAndView testRedirect(@RequestParam("username") String username){
		ModelAndView mAndView = new ModelAndView("redirect:/user/index.html");
		
		User user = new User();
		user.setName(username);		
		mAndView.addObject("user", user);
		
		return mAndView;
	}
}
```





然后项目启动后，在浏览器中访问：http://localhost:9180/CookieDemo/testForward.html?username=forward

页面显示效果如下：

[![wKioL1g7i7bxN-UVAAAxxLSwXWU933.png](http://s1.51cto.com/wyfs02/M00/8A/BF/wKioL1g7i7bxN-UVAAAxxLSwXWU933.png)](http://s1.51cto.com/wyfs02/M00/8A/BF/wKioL1g7i7bxN-UVAAAxxLSwXWU933.png)

可以看出，在使用forward进行转发时请求的URL链接是不会改变的

接着，在浏览器中访问：http://localhost:9180/CookieDemo/testRedirect.html?username=redirect

页面显示效果如下：

[![wKiom1g7i9OAT7MjAAA36HQ7Kgw414.png](http://s2.51cto.com/wyfs02/M01/8A/C4/wKiom1g7i9OAT7MjAAA36HQ7Kgw414.png)](http://s2.51cto.com/wyfs02/M01/8A/C4/wKiom1g7i9OAT7MjAAA36HQ7Kgw414.png)

可以看出，在使用redirect进行重定向时请求的URL链接发生了改变，并且在controller中放置的参数并没有传递进行。那么，如果想要在重定向时把参数也传递过去应该怎么做呢？

## 方法一：常规做法，重定向之前把参数放进Session中，在重定向之后的controller中把参数从Session中取出并放进ModelAndView

示例代码如下：

```java
	@RequestMapping("/user/index.html")
	public ModelAndView userIndex(HttpServletRequest request) {
		ModelAndView mAndView = new ModelAndView("user/index");
		HttpSession session = request.getSession();
		
		mAndView.addObject("user", session.getAttribute("user"));
		session.removeAttribute("user");
		
		return mAndView;
	}

	@RequestMapping("/testRedirect.html")
	public ModelAndView testRedirect(@RequestParam("username") String username,HttpServletRequest request){
		ModelAndView mAndView = new ModelAndView("redirect:/user/index.html");
		
		User user = new User();
		user.setName(username);
		request.getSession().setAttribute("user", user);
		
		return mAndView;
	}
```



## 方法二：使用RedirectAttributes类

示例代码如下：

```java
	@RequestMapping("/user/index.html")
	public ModelAndView userIndex() {
		return new ModelAndView("user/index");
	}

	@RequestMapping("/testRedirect.html")
	public ModelAndView testRedirect(@RequestParam("username") String username,RedirectAttributes redirectAttributes){
		ModelAndView mAndView = new ModelAndView("redirect:/user/index.html");
		
		User user = new User();
		user.setName(username);
//              redirectAttributes.addAttribute("user", user);  //URL后面拼接参数
		redirectAttributes.addFlashAttribute("user", user);
		
		return mAndView;
	}
```



使用RedirectAttributes这个类来传递参数写法就很简单了，只需要在需要重定向的controller的方法参数中添加RedirectAttributes类型的参数，然后把需要重定向之后也能够获取的参数放进去即可

当然，据说RedirectAttributes本质上也是通过Session来实现的（PS：相关源代码我没看过，只能据说了）。实际上上面的代码是省略的写法，因为重定向之后就直接返回页面视图了，如果经过几次重定向的话估计上面那种写法就获取不到参数了，因此关于获取参数一般还有以下两种方式：

（1）使用@ModelAttribute注解获取参数：

```java
	@RequestMapping("/user/index.html")
	public ModelAndView userIndex(@ModelAttribute("user") User user) {
		ModelAndView mAndView = new ModelAndView("user/index");
		mAndView.addObject("user", user);

		return mAndView;
	}
```



（2）使用RequestContextUtils类来获取：

```java
	@RequestMapping("/user/index.html")
	public ModelAndView userIndex(HttpServletRequest request) {
		ModelAndView mAndView = new ModelAndView("user/index");
		
		Map<String, Object> map = (Map<String, Object>) RequestContextUtils.getInputFlashMap(request);
		User user = (User) map.get("user");
		
		mAndView.addObject("user", user);
		return mAndView;
	}
```



总结，这两种获取方式都可以成功获取到参数。但是还是略显麻烦，如果只是一次重定向之后就返回页面视图的话推荐使用最简单那种写法


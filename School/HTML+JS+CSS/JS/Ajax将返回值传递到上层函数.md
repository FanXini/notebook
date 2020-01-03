# Ajax在跨域情况下async:false失效

## 1. 域内请求

function通过ajax调用获取后台数据，结果返回出来的结果均为空，代码如下：

```javascript
function chart_coinbase_getdata() {
    var test = {postdata:"chart_linear_graph_getdata"}
    var return_data = "";
    jQuery.ajax({
        type: "POST",
        dataType: "json",
        url: "/post/chart_linear_graph_getdata",
        data: test,
        success: function(response_data) {
            // console.log("response_data", response_data)
            //alert(response_data);
            // chart_coinbase(response_data);
            return_data = response_data;
        }
    });
    return return_data;
};
```

修改成如下代码即可：添加async:false参数

```javascript
function chart_coinbase_getdata() {
    var test = {postdata:"chart_linear_graph_getdata"}
    var return_data = "";
    jQuery.ajax({
        type: "POST",
        dataType: "json",
        url: "/post/chart_linear_graph_getdata",
        data: test,
        async:false,
        success: function(response_data) {
            // console.log("response_data", response_data)
            //alert(response_data);
            // chart_coinbase(response_data);
            return_data = response_data;
        }
    });
    return return_data;
};
```

## 2.跨域请求

Ajax在跨域情况下async:false失效

 版权声明：本文为博主原创文章，未经博主允许不得转载。	https://blog.csdn.net/huanghanqian/article/details/52077603

首先看代码：

```javascript
$.ajax({
    url: "http://www.hzhuti.com",    //请求的url地址
    dataType: "jsonp",   //返回格式为jsonp
    async: false, //请求是否异步，默认为true，即异步，这也是ajax重要特性
    data: { "id": "value" },    //参数值
    type: "GET",   //请求方式
    beforeSend: function() {
        //请求前的处理，可不写
    },
    success: function(data) {
        //请求成功时处理
    },
    complete: function() {
        //请求完成的处理，可不写
    },
    error: function() {
        //请求出错处理
    }
});

```

   可以看到上述代码中，dataType为jsonp，可以处理跨域请求；同时async为设置为false，为同步方式，即当这个AJAX执行完毕后才会继续运行其他代码。但是发现在调试时，仍然会执行完之后的代码，再去执行ajax中的代码，即同步失效了，还是异步的方式，为什么呢？

​    上网查阅了一些资料，发现Jquery的API中提到，JSONP格式不支持跨域同步。因为ajax的核心是通过XmlHttpRequest获取非本页内容，而jsonp的核心则是动态添加<script>标签来调用[服务器](https://www.baidu.com/s?wd=%E6%9C%8D%E5%8A%A1%E5%99%A8&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)提供的[js脚本](https://www.baidu.com/s?wd=js%E8%84%9A%E6%9C%AC&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)。jsonp的实现不是ajax，而是script节点,所以对ajax有效的配置未必对jsonp有效。

​    解决方法：将jsonp请求之后的操作放在success回调函数中处理。举例：

```javascript
function getA(allFunc){      
   $.ajax({
       url: "http://www.hzhuti.com",    //请求的url地址
       dataType: "jsonp",   //返回格式为jsonp
       async: false, //请求是否异步，默认为true，即异步，这也是ajax重要特性
       data: { "id": "value" },    //参数值
       type: "GET",   //请求方式
       success: function(data) {
           //请求成功时处理
           if("function"==typeof(allFunc){
                 allFunc();//将ajax请求后需要进行的操作放在回调函数allFunc()中
           }
       },
       error: function() {
           //请求出错处理
       }
   });
}

```
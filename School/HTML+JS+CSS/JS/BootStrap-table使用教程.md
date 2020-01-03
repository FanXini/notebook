# 手把手教你--Bootstrap Table表格插件的使用及数据导出(可导出Excel2003及Exce2007)

2018年04月25日 17:56:04 [小流至江河](https://me.csdn.net/javaYouCome) 阅读数：10922

[其他优质博客](https://www.cnblogs.com/wdlhao/p/6694083.html)

### 1.介绍

Bootstrap Table介绍见官网：<http://bootstrap-table.wenzhixin.net.cn/zh-cn/>

Bootstrap 中文网：<http://www.bootcss.com/>       

Bootstrap Table Demo：<http://issues.wenzhixin.net.cn/bootstrap-table/index.html>

Bootstrap Table API：<http://bootstrap-table.wenzhixin.net.cn/zh-cn/documentation/>

Bootstrap Table源码：<https://github.com/wenzhixin/bootstrap-table>

Boostrap Table 扩展API：<http://bootstrap-table.wenzhixin.net.cn/extensions/>

## 2.页面引用

为了方便bootstrap及bootstrap-table用官方推荐引用方式,想自己下载源码可以自行下载.

导出[Excel](https://www.baidu.com/s?wd=Excel&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)所需额外的4个js可从以下2个地址找到那4个文件,不需要导出的可以无视.

下载地址   <https://github.com/wenzhixin/bootstrap-table/tree/master/src/extensions/export>

​                <https://github.com/hhurz/tableExport.jquery.plugin>

```html
<!-- 引入bootstrap样式 -->
<link href="https://cdn.bootcss.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet"> 
<!-- 引入bootstrap-table样式 -->
<link href="https://cdn.bootcss.com/bootstrap-table/1.11.1/bootstrap-table.min.css" rel="stylesheet"> 
 
<!-- jquery及bootstrapjs -->
<script src="https://cdn.bootcss.com/jquery/2.2.3/jquery.min.js"></script>
<script src="https://cdn.bootcss.com/bootstrap/3.3.6/js/bootstrap.min.js"></script> 
 
<!-- bootstrap-table.min.js -->
<script src="https://cdn.bootcss.com/bootstrap-table/1.11.1/bootstrap-table.min.js"></script> 
<!-- 引入中文语言包 -->
<script src="https://cdn.bootcss.com/bootstrap-table/1.11.1/locale/bootstrap-table-zh-CN.min.js"></script> 
<!-- bootstrap-table-export数据导出---后面两个是Excel2007所需要的js -->
<script src="${basePath }/js/bootstrap-table-export.js"></script>
<script src="${basePath }/js/tableExport.min.js"></script>
<script src="${basePath }/js/xlsx.core.min.js"></script> 
<script src="${basePath }/js/FileSaver.min.js"></script

```

## **3.简单示例**

**3.1在html中定义一个table标签**

```javascript
<table id="table"></table>
```

3.2使用bootstrap-table有两种方式,第一种是:通过data属性的方法--因为不灵活不做过多演示.类似

```javascript
<table data-toggle="table" data-url="data1.json">
    <thead>
        <tr>
            <th data-field="id">Item ID</th>
            <th data-field="name">Item Name</th>
            <th data-field="price">Item Price</th>
        </tr>
    </thead>
</table>

```

3.3使用JavaScript方式,几乎所有使用bootstrap-table的全是以这种方式,学习bootstrap-table,就是学习它的API,介绍基本常用的API,详情请查看官方文档或看这个博主翻译的

<https://blog.csdn.net/rickiyeat/article/details/56483577>

页面引入以下js

```javascript
$("#table").bootstrapTable({ // 对应table标签的id
      url: base_path + "/product/list",   //AJAX获取表格数据的url
      striped: true,                      //是否显示行间隔色(斑马线)
      pagination: true,                   //是否显示分页（*）
      sidePagination: "server",           //分页方式：client客户端分页，server服务端分页（*）
      paginationLoop: false,		  //当前页是边界时是否可以继续按
      queryParams: function (params) {    // 请求服务器数据时发送的参数，可以在这里添加额外的查询参数，返回false则终止请求
			return {
			limit: params.limit, // 每页要显示的数据条数
			offset: params.offset, // 每页显示数据的开始行号
			//sort: params.sort, // 要排序的字段
			//sortOrder: params.order, // 排序规则
			//dataId: $("#dataId").val() // 额外添加的参数
		  }
      },//传递参数（*）
      pageNumber:1,                       //初始化加载第一页，默认第一页
      pageSize: 10,                       //每页的记录行数（*）
      pageList: [10, 25, 50, 100,'all'],  //可供选择的每页的行数（*）
      contentType: "application/x-www-form-urlencoded",//一种编码。在post请求的时候需要用到。这里用的get请求，注释掉这句话也能拿到数据
      //search: true,                     //是否显示表格搜索，此搜索是客户端搜索，不会进服务端，所以，个人感觉意义不大
      strictSearch: false,		  //是否全局匹配,false模糊匹配
      showColumns: true,                  //是否显示所有的列
      showRefresh: true,                  //是否显示刷新按钮
      minimumCountColumns: 2,             //最少允许的列数
      clickToSelect: false,               //是否启用点击选中行
      //height: 500,                      //行高，如果没有设置height属性，表格自动根据记录条数觉得表格高度
      //uniqueId: "id",                   //每一行的唯一标识，一般为主键列
      showToggle:true,                    //是否显示详细视图和列表视图的切换按钮
      cardView: false,                    //是否显示详细视图
      detailView: false,                  //是否显示父子表
      cache: false,                       // 设置为 false 禁用 AJAX 数据缓存， 默认为true
      sortable: true,                     //是否启用排序
      sortOrder: "asc",                   //排序方式
      sortName: 'sn', // 要排序的字段
      columns: [
          {
              field: 'sn', // 返回json数据中的name
              title: '订货号', // 表格表头显示文字
              align: 'center', // 左右居中
              valign: 'middle' // 上下居中
          }, {
              field: 'productname',
              title: '商品名称',
              align: 'center',
              valign: 'middle'
          }, {
              field: 'cost',
              title: '价格(¥)',
              align: 'center',
              valign: 'middle',
              sortable:true
          }, {
              field: 'brankname',
              title: '品牌',
              align: 'center',
              valign: 'middle',
          }, {
              field: 'specificationvalues',
              title: '规格',
              align: 'center',
              valign: 'middle',
          }, {
              field: 'areaname',
              title: '产地',
              align: 'center',
              valign: 'middle',
          }
      ],
      onLoadSuccess: function(){  //加载成功时执行
    	  console.info("加载成功");
      },
      onLoadError: function(){  //加载失败时执行
          console.info("加载数据失败");
      },
      
      //>>>>>>>>>>>>>>导出excel表格设置
      showExport: phoneOrPc(),              //是否显示导出按钮(此方法是自己写的目的是判断终端是电脑还是手机,电脑则返回true,手机返回falsee,手机不显示按钮)
      exportDataType: "basic",              //basic', 'all', 'selected'.
      exportTypes:['excel','xlsx'],	    //导出类型
      //exportButton: $('#btn_export'),     //为按钮btn_export  绑定导出事件  自定义导出按钮(可以不用)
      exportOptions:{  
          //ignoreColumn: [0,0],            //忽略某一列的索引  
          fileName: '数据导出',              //文件名称设置  
          worksheetName: 'Sheet1',          //表格工作区名称  
          tableName: '商品数据表',  
          excelstyles: ['background-color', 'color', 'font-size', 'font-weight'],  
          //onMsoNumberFormat: DoOnMsoNumberFormat  
      }
      //导出excel表格设置<<<<<<<<<<<<<<<<
 
});

```

```javascript
/*判断终端是手机还是电脑--用于判断文件是否导出(电脑需要导出)*/
function phoneOrPc(){
	var sUserAgent = navigator.userAgent.toLowerCase();  
	var bIsIpad = sUserAgent.match(/ipad/i) == "ipad";  
	var bIsIphoneOs = sUserAgent.match(/iphone os/i) == "iphone os";  
	var bIsMidp = sUserAgent.match(/midp/i) == "midp";  
	var bIsUc7 = sUserAgent.match(/rv:1.2.3.4/i) == "rv:1.2.3.4";  
	var bIsUc = sUserAgent.match(/ucweb/i) == "ucweb";  
	var bIsAndroid = sUserAgent.match(/android/i) == "android";  
	var bIsCE = sUserAgent.match(/windows ce/i) == "windows ce";  
	var bIsWM = sUserAgent.match(/windows mobile/i) == "windows mobile";  
	if (bIsIpad || bIsIphoneOs || bIsMidp || bIsUc7 || bIsUc || bIsAndroid || bIsCE || bIsWM) {  
		return false;  
	} else {  
	    return true; 
	}  
}

```



需要注意的项

1.测试时分页先选择'client',因为分页是客户端的话,导出数据会方便可以随意定义'basic', 'all', 'selected'.如果分页是服务端的话即使选择'all导出的也是当前页('basic'),如果想导出全部话,可以先将页面显示条目数为全部,再导出当前页即就是全部数据了.

2.分页如果是服务端的话返回的json串必须包含2个数据,一个是"total"即所有数据条数.另一个"rows"即"当前页"显示的内容.(见下图json串格式)

![img](https://img-blog.csdn.net/20180425173637765)

后台参考代码(以服务端分页为例,如果客户端分页先把方法参数去掉,再把for循环改成循环100次,后直接返回list即可)

```java
@Controller
@RequestMapping("/product")
public class TestController {
 
	@RequestMapping("/list")
	@ResponseBody
	public Map<String,Object> listProduct(@RequestParam(value = "limit", required = false)Integer limit, @RequestParam(value = "offset", required = false)Integer offset) {
		Map<String,Object> map = new HashMap<>();
		
		List<SkuExt> list = new ArrayList<SkuExt>();//此处应该是从数据库查出来的数据,为了测试方便写个循环
		for (int i = offset; i < limit+offset; i++) {
			SkuExt skuExt = new SkuExt();
			skuExt.setSn(i+"");
			skuExt.setProductname("商品名称"+i);
			skuExt.setCost(new BigDecimal(i*100));
			skuExt.setBrankname("品牌名称"+i);
			skuExt.setSpecificationvalues("规格值是"+i);
			skuExt.setAreaname("产地"+i);
			list.add(skuExt);
		}
		map.put("total", 100);//假设共有100条数据
		map.put("rows", list);
        return  map;
    }

```

--------------------------------------以上就是代码,效果如下------------------------------------------

![img](https://img-blog.csdn.net/20180425175404980)

showToggle:true这个属性的效果如下:

![img](https://img-blog.csdn.net/20180425175419576)

导出按钮效果如下:导出的按钮和下拉提示应该不是这样,是我自己改了里面内容文字,很好实现.

![img](https://img-blog.csdn.net/20180425175441654)

 

以上,纯手写,大神勿喷.
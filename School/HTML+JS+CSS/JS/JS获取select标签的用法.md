# [js获取select标签选中的值](https://www.cnblogs.com/qq3245792286/p/6390504.html)

## 原生js

```javascript
var obj = document.getElementByIdx_x(”testSelect”); //定位id

var index = obj.selectedIndex; // 选中索引

var text = obj.options[index].text; // 选中文本

var value = obj.options[index].value; // 选中值
```

## [jQuery](http://lib.csdn.net/base/22)中获得选中select值

### 第一种方式

```javascript
$('#testSelect option:selected').text();//选中的文本

$('#testSelect option:selected') .val();//选中的值

$("#testSelect ").get(0).selectedIndex;//索引 
```

### 第二种方式

```javascript
$("#tesetSelect").find("option:selected").text();//选中的文本
…….val();
…….get(0).selectedIndex;
```

#### 如果select标签是有id属性的，如

```html
<select id=xx>...
则用下述方法获取当前选项的值：
var v = xx.value;
或
var v = document.getElementById("xx").value;   //此方法兼容性好
```



#### 如果select标签是有name属性的，如

```html
<form name=form1>
<select name=xx>...
则用下述方法获取当前选项的值：
var v = form1.xx.value;
或
var v = document.getElementsByName("xx")[0].value;
如果同一页面含有多个name属性相同的标签，则上述[0]中的数字要改为相应的物理顺序号（从0起算）
```

#### 如果select标签不含有任何可供定位的属性，如

```html
 <select>...
则用下述方法获取当前选项的值：
var v = document.getElementsByTagName("select")[0].value;
如果同一页面含有多个select标签，则上述[0]中的数字要改为相应的物理顺序号（从0起算）
```

#### 对于以下select标签，获取当前选择的值得方式如下：

```html
<select id="test" name="">
<option value="1">text1</option>
<option value="2">text2</option>
</select>
```

code:
一：javascript原生的方法
1:拿到select对象： var myselect=document.getElementById("test");
2：拿到选中项的索引：var index=myselect.selectedIndex ; // selectedIndex代表的是你所选中项的index
3:拿到选中项options的value： myselect.options[index].value;
4:拿到选中项options的text： myselect.options[index].text;


二：jquery方法（前提是已经加载了jquery库）

1:var options=$("#test option:selected");  //获取选中的项

2:alert(options.val());   //拿到选中项的值

3:alert(options.text());   //拿到选中项的文本
## FormData 使用详解

## 一 概述

FormData类型其实是在XMLHttpRequest 2级定义的，它是为序列化表以及创建与表单格式相同的数据（当然是用于XHR传输）提供便利。

FormData的详细介绍及使用请[点击此处](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)

FormData 对象： 不支持ie8，ie9
![formdata](https://sfault-image.b0.upaiyun.com/203/933/2039338818-5a28f425ab032_articlex)

## 二 创建一个formData对象实例的方式

### 2.1. 创建一个空对象

```javascript
var formData = new FormData();//通过append方法添加数据
```

### 2.2. 使用已有表单来初始化对象

```javascript
//表单示例
<form id="myForm" action="" method="post">
    <input type="text" name="name">名字
    <input type="password" name="psw">密码
    <input type="submit" value="提交">
</form>

//方法示例
// 获取页面已有的一个form表单
var form = document.getElementById("myForm");
// 用表单来初始化
var formData = new FormData(form);
// 我们可以根据name来访问表单中的字段
var name = formData.get("name"); // 获取名字
var psw = formData.get("psw"); // 获取密码
// 当然也可以在此基础上，添加其他数据
formData.append("token","kshdfiwi3rh");
```

 

## 三. 操作方法

formData里面存储的数据是以健值对的形式存在的，key是唯一的，一个key可能对应多个value。 
如果是使用表单初始化，每一个表单字段对应一条数据，它们的HTML name属性即为key值，它们value属性对应value值。 

### 3.1.获取值

```javascript
//通过get(key)/getAll(key)来获取对应的value
formData.get("name"); // 获取key为name的第一个值
formData.get("name"); // 返回一个数组，获取key为name的所有值
```

###  3.2 添加数据

```javascript
//通过append(key, value)来添加数据，如果指定的key不存在则会新增一条数据，如果key存在，则添加到数据的末尾
formData.append("k1", "v1");
formData.append("k1", "v2");
formData.append("k1", "v3");
```

获取值时方式及结果如下

```javascript
formData.get("k1"); // "v1"
formData.getAll("k1"); // ["v1","v2","v3"]
```

### 3.3设置修改数据

```javascript
//set(key, value)来设置修改数据，如果指定的key不存在则会新增一条，如果存在，则会修改对应的value值
formData.append("k1", "v1");
formData.set("k1", "1");
formData.getAll("k1"); // ["1"]
```

### 3.4.判断是否存在对应数据

```javascript
//has(key)来判断是否对应的key值
formData.append("k1", "v1");
formData.append("k2",null);

formData.has("k1"); // true
formData.has("k2"); // true
formData.has("k3"); // false
```

### 3.5.删除数据

```javascript
//delete(key)删除数据
formData.append("k1", "v1");
formData.append("k1", "v2");
formData.append("k1", "v1");
formData.delete("k1");

formData.getAll("k1"); // []
```

### 3.6 遍历

我们可以通过entries()来获取一个迭代器，然后遍历所有的数据，

```javascript
formData.append("k1", "v1");
formData.append("k1", "v2");
formData.append("k2", "v1");

var i = formData.entries();

i.next(); // {done:false, value:["k1", "v1"]}
i.next(); // {done:fase, value:["k1", "v2"]}
i.next(); // {done:fase, value:["k2", "v1"]}
i.next(); // {done:true, value:undefined}
```

可以看到返回迭代器的规则

1. 每调用一次next()返回一条数据，数据的顺序由添加的顺序决定
2. 返回的是一个对象，当其done属性为true时，说明已经遍历完所有的数据，这个也可以作为判断的依据
3. 返回的对象的value属性以数组形式存储了一对key/value，数组下标0为key，下标1为value，如果一个key值对应多个value，会变成多对key/value返回

我们也可以通过values()方法只获取value值



```javascript
formData.append("k1", "v1");
formData.append("k1", "v2");
formData.append("k2", "v1");

var i = formData.entries();

i.next(); // {done:false, value:"v1"}
i.next(); // {done:fase, value:"v2"}
i.next(); // {done:fase, value:"v1"}
i.next(); // {done:true, value:undefined}
```

3.7  发送数据

```javascript
var xhr = new XMLHttpRequest();
xhr.open("post","login");
xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
xhr.send(formData);
```



## 四.JQuery实例

```javascript
//添加数据方式见上二。
//processData: false, contentType: false,多用来处理异步上传二进制文件。
 $.ajax({
    url: 'xxx',
    type: 'POST',
    data: formData,                    // 上传formdata封装的数据
    dataType: 'JSON',
    cache: false,                      // 不缓存
    processData: false,                // jQuery不要去处理发送的数据
    contentType: false,                // jQuery不要去设置Content-Type请求头
    success:function (data) {           //成功回调
        console.log(data);
    }
});
```

附：

```javascript
/**
 * 将以base64的图片url数据转换为Blob文件格式
 * @param urlData 用url方式表示的base64图片
 */
function convertBase64UrlToBlob(urlData) {
    var bytes = window.atob(urlData.split(',')[1]); //去掉url的头，并转换为byte
    //处理异常,将ascii码小于0的转换为大于0
    var ab = new ArrayBuffer(bytes.length);
    var ia = new Uint8Array(ab);
    for(var i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i);
    }
    return new Blob([ab], {
        type: 'image/png'
    });
}
```
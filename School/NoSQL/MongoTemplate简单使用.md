# MongoTemplate 简单使用

Spring整合Mongo需要的三个jar包

- spring-data-commons
- spring-data-mongodb
- mongo-java-driver

[配置(包括分页插件)](https://blog.csdn.net/tototuzuoquan/article/details/57155827)

## 1.简介

查了好多关于MongoDB的文字，大部分基于数据库的，很少有Template的，这里总结下

2.实现代码
2.1插入对象：
MongoTemplate mongos = MongoInstance.getMongo(); 获得模板对象

在项目中用：

@Autowired
​ private MongoTemplate mongoTemplate;

对象代码如下：

```java

package com.jf.cloud.monitor;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.data.mongodb.core.MongoTemplate;
 
public class MongoInstance {
 
	public static MongoTemplate getMongo(){
		return new ClassPathXmlApplicationContext("applicationContext.xml").getBean("mongoTemplate",MongoTemplate.class);
	}
	
}

```




MongoDB引入到ssm项目可以看我上篇文章；

当然你也可以这么写：

```java
//        Mongo mongo = new Mongo("localhost",27017);
//        
//        DB db = mongo.getDB("test");
//        
//        DBCollection co = db.getCollection("dayCollection");
```



获得collection进行操作；

## 2.调用模板的insert方法

mogo.insert(插入的对象，要插入的集合)；

 ```java
static MongoTemplate mongo = MongoInstance.getMongo();
 Query query=new Query(Criteria.where("operationCode").is("b1"));
	    List<Document> collection = mongo.find(query, Document.class,logName);
	    int i=0;
	    int jvmTotal=0;
	    int jvmUsed = 0;
	    for(Document c:collection){
	    //	System.out.println(c.get("jvmTotal").toString());
	    	 jvmTotal =+ Integer.valueOf(c.get("jvmTotal").toString());
	    	i++;
	    	 jvmUsed =+Integer.valueOf(c.get("jvmUsed").toString());
	    }
	    Document doc = new Document();
	    doc.put("projectCode",ServerMonitorFormatter.proCode);
	    doc.put("operationCode", ServerMonitorFormatter.serviceCode.b1.toString());
	    doc.put("controlCode", ServerMonitorFormatter.controlCode);
	    doc.put("logLevel", ServerMonitorFormatter.logLevel.INFO.toString());  		
	    doc.put("LogTime", logName);
	    doc.put("jvmUsed", jvmUsed/i);
	    doc.put("jvmTotal", jvmTotal/i);
	    mongo.insert(doc, DAY_DATA_COLLECTION);	
 ```



## 3查询初级操作

这是查询key：operationCode 是c1，recordTime时间范围在startDate和endDate之间的操作。

//"$gt" 、"$gte"、 "$lt"、 "$lte"(分别对应">"、 ">=" 、"<" 、"<=")

```java
Query query=new Query(Criteria.where("operationCode").is("c1").and("recordTime").gte(startDate).lte(endDate));
List<Document> collection =
mongos.find(query, Document.class,"dayCollection");
```

**mongos.find（查询条件，返回的对象类型，在哪个collection中查询）**

当然你还可以加条件。

排序：

```java
 Query query=new Query(Criteria.where("operationCode").is("b1"));
 query.with(new Sort(new Order(Direction.DESC, "recordTime")));//时间倒序
 query.limit(1);//选取最新的一条
 List<Document> collection = mongoTemplate.find(query,Document.class,daStr);
```

 

## 4高级查询操作

Aggregation的应用，很版本有很多关系。但是功能更强大。还有一种使用DBObject方法的查询

 

查询如上；

 ```java
Query query=new Query(Criteria.where("operationCode").is("b4"));
		List<Document> collection = mongo.find(query, Document.class,logName);
		int count = Integer.valueOf(collection.get(0).get("diskCount").toString());
		 
		 
		Criteria c = Criteria.where("operationCode").is("b4");
		
		Aggregation aggregation = Aggregation.newAggregation(Aggregation.match(c),Aggregation.limit(count));
		AggregationResults<Document> collection2 =  mongo.aggregate(aggregation, logName ,Document.class);
		
	    	
	    mongo.insert(collection2, DAY_DATA_COLLECTION);

 ```




聚合查询；

 

感谢：https://blog.csdn.net/vbirdbest/article/details/77102999

---------------------
作者：zzqtty 
来源：CSDN 
原文：https://blog.csdn.net/zzqtty/article/details/80846036 
版权声明：本文为博主原创文章，转载请附上博文链接！
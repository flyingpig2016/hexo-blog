---
title: mongodb常用命令学习
date: 2017-07-02 16:23:25
tags: MongoDB
categories:
toc : true     
comments: true  
---
## mongoDB常用命令
###  **1. 展示数据库：**`show dbs`；
### **2. 显示当前数据库对象或集合：**`db`;
### **3. 连接到一个指定的数据库:**  `use`;
<!--more-->
### **4. 关系型数据库和mongodb的术语：**
![这里写图片描述](http://img.blog.csdn.net/20170610182030359?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmx5aW5ncGlnMjAxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### **5. 集合就是 MongoDB 文档组，**类似于 RDBMS中的表格，例如：
```
{"site":"www.baidu.com"}
{"site":"www.google.com","name":"Google"}
{"site":"www.runoob.com","name":"菜鸟教程","num":5}
```
&emsp;&emsp;集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。
### **6.创建myblog数据库：
**`use myblog`,此时还不能用`show dbs`来查看，因为没有数据。当我们插入数据的时候才能看到数据列表：`db.myblog.insert({name:'bangbang'})` ;

### **7.删除base02数据库：**
	首先，我们先创建一个base02数据库,并插入一条数据，**注意：要想显示数据库，需要至少插入一个文档进去。**
```
use base02;
db.base02.insert({name:'qiqi'}); //插入一条数据
db.dropDatabase();		//删除数据库
```
###  **8.创建和删除集合：**
创建集合：
```
use base02;
db.createCollection('mycollection'); 
show collections //使用show collections来检查创建的集合
db.createCollection("mycol", { capped : true, autoIndexID : true, size : 6142800, max : 10000 } )//根据集合的规格创建
```
其实在MongoDB中并不需要创建集合。 当插入一些文档 MongoDB 会自动创建集合。
```
//创建集合mycol1
db.mycol1.insert({name:'bangbang'})
//删除集合mycol1
db.mycol1.drop()
```
###  **9.插入文档**
```
db.mycol1.insert({name:'bangbang'})
```
要以单个查询插入多个文档，可以通过文档 insert() 命令的数组方式
```
var datas = [
	{title: 'MongoDB Overview',     description: 'MongoDB is no sql database',    by_user: 'runoob.com',    url: 'http:
//www.runoob.com',    tags: ['mongodb', 'database', 'NoSQL'],    likes: 100 }, 
	{title: 'NoSQL Overview',     description: 'No sql
database is very fast',    by_user: 'runoob.com',    url: 'http://www.runoob.com',    tags: ['mongodb', 'database', 'NoSQL'],    likes
: 10 }, 
	{title: 'Neo4j Overview',     description: 'Neo4j is no sql database',    by_user: 'Neo4j',    url: 'http://www.neo4j.com'
,    tags: ['neo4j', 'database', 'NoSQL'],    likes: 750 }];
db.mycol1.insert(datas);
```
### **10.查询文档**
```
//格式化显示
db.mycol1.find().pretty();
//非格式化显示
db.mycol1.find();
//按照条件查找
db.mycol1.find({"title":"Neo4j Overview"})
//并列查找相当于and
db.mycol1.find({"title":"Neo4j Overview","likes":750})
//或查找，相当于or
db.mycol1.find({
		$or:[{likes:10},{likes:750}]
	})
//联合查询
db.mycol1.find({likes:10,$or:[{likes:10},{likes:750}]})
```
**11.更新文档**
&emsp;&emsp;update()方法将现有的文档中的值更新，而save()方法使用传递到save()方法的文档替换现有的文档。
```
//用update更新选中的文档
db.mycol1.update({title:'MongoDB Overview'},{$set:{title:'bangbang',job:'program'}})
//用save更新，也就是全部替换了_id为593df22dfafc15a3d4d9bfc4的文档
db.mycol1.save({_id:ObjectId("593df22dfafc15a3d4d9bfc4"),title:'qiqi',description:'I am qiqi'}
```
###  **12.删除文档**
```
//删除所有标题为MongoDB Overview的文件
db.mycol1.remove({
'title':'MongoDB Overview'})
//只删除一个
db.mycoll1.remove({'title':'MongoDB Overview'},1)
//如果没有指定删除条件，则MongoDB将从集合中删除整个文件。这相当于SQL的 truncate 命令。
db.mycol1.remove()
```
### **13.MongoDB投影**
&emsp;&emsp;mongodb投影意义是只选择需要的数据，而不是选择整个一个文档的数据。如果一个文档有5个字段，只需要显示3个，只从中选择3个字段。MongoDB的find()方法，解释了MongoDB中查询文档接收的第二个可选的参数是要检索的字段列表。在MongoDB中，当执行find()方法，那么它会显示一个文档的所有字段。要限制这一点，需要设置字段列表值为1或0。1是用来显示字段，而0被用来隐藏字段.
```
//这里只显示title和description字段，不显示_id字段
db.mycol1.find({},{title:1,description:1,_id:0})
```
### **14.限制文档**
&emsp;&ems;要在MongoDB中限制记录，需要使用limit()方法。 limit() 方法接受一个数字类型的参数，这是要显示的文档数量
```
//只显示两条数据
db.mycol1.find().limit(2);
//只显示一条数据
db.mycol1.find({},{title:1,description:1,_id:0}).limit(1)
```
除了 limit()方法还有一个方法 skip()也接受数字类型参数并用于跳过文件数
```
//跳过第一条显示全部
db.mycol1.find().skip(1);
//跳过第一条显示两条数据
db.mycol1.find().limit(2).skip(1);
```
### **15.文档排序**
&emsp;&emsp;要排序MongoDB中的文档，需要使用 sort()方法。 sort() 方法接受一个包含字段列表以及排序顺序的文档。 要使用1和-1指定排序顺序。1用于升序，而-1是用于降序。
```
db.mycol1.find().sort({likes:1})
```
### **16.MongoDB索引**
索引是特殊的数据结构，存储在一个易于设置遍历形式的数据的一小部分。索引存储在索引中指定特定字段的值或一组字段，并排序字段的值。要创建索引，需要使用MongoDB的ensureIndex()方法。
```
db.mycol2.ensureIndex({title:1})
```
### **17.MongoDB 聚合**
&emsp;&emsp;聚合操作处理数据记录并返回计算结果。从多个文档聚合分组操作数值，并可以执行多种对分组数据业务返回一个结果。
### **18.复制**
&emsp;&emsp;复制是同步在多个服务器上的数据过程。复制提供了冗余和数据在不同的数据库服务器上的多个副本提高数据的可用性，复制防止在单个服务器上丢失数据库。 复制也可以从硬件故障和服务中断中恢复。带有数据的其他副本，可以选择其中一个灾难恢复，报告或备份。
[主从复制](http://www.cnblogs.com/huangxincheng/archive/2012/03/04/2379755.html)
---
title: 'MongoDB (3):查询分析与索引基础'
date: 2017-08-21 15:54:26
toc: true
tags:
 - MongoDB
 - NoSQL
---

   这篇文章主要是写简单的写MongoDB的索引入门，索引是什么，有什么作用这种废话就不要介绍了，但是正如我在第一篇文章中所说，你需要注意MongoDB和传统许多的关系型数据库一样基于B-Tree索引。

   MongoDB提供了多种索引类型让用户在不同的情况下选择，在介绍这些之前，我们必须先了解一个函数explain,它用于做查询分析，在3.0版本之后，explain函数受到了新的更新，它的输出大为不同，如果你尝试搜索关于MongoDB的explain博文就会发现很多都是基于3.0版本之前介绍的，由于我用的3.4.7版本的MongoDB,所以我在此就基于该版本来写

> [官方文档](https://docs.mongodb.com/manual/reference/operator/meta/explain/#behavior)上说3.2开始可以使用$explain来得到2.6版本的那种返回结果，但是我尝试了好像没用...

<!--more-->

   要将索引首先要有足够多的数据，有MongoDB shell是一个js shell，我们可以直接以js语法写一个循环来插入20万条数据：

```javascript
for(var i=0;i<200000;i++){
    db.num.insert({order:i})
}

> db.num.count()
200000
```

## 查询分析

  现在我们有了20万条数据，你插入时或许会观察到这20万数据花费了你十几秒到几十秒不等，效率很慢不是么？如果做一个互联网应用，每秒写入数据量几百万条，虽然说服务器性能远远大于我们的机器，但是我们应该去思考这个点，鉴于我们现在讨论的是索引，实际上处理索引在插入数据时又要花费一些时间，但是它会给我们的查询带来很多的好处，现在我们利用explain来看看,没有索引的情况下，我们查询一组数据需要花费多少时间吧：

```powershell
> db.num.find({order:{$gt:185000,$lt:185005}}).explain({})
{
        "queryPlanner" : {						//包含了查询优化器选择查询计划的信息
                "plannerVersion" : 1,			//计划版本
                "namespace" : "saul.num",		//指定查询的命名空间
                "indexFilterSet" : false,		// 是否使用索引过滤
                "parsedQuery" : {			   //查询语句
                        "$and" : [
                                {
                                        "order" : {
                                                "$lt" : 185005
                                        }
                                },
                                {
                                        "order" : {
                                                "$gt" : 185000
                                        }
                                }
                        ]
                },
                "winningPlan" : {			//查询优化器选择查询计划的详情
                        "stage" : "COLLSCAN",	//阶段描述操作符
                        "filter" : {
                                "$and" : [
                                        {
                                                "order" : {
                                                        "$lt" : 185005
                                                }
                                        },
                                        {
                                                "order" : {
                                                        "$gt" : 185000
                                                }
                                        }
                                ]
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]	//被查询优化器拒绝的候选查询计划
        },
         "executionStats" : {			//描述最终采取的查询计划的完成执行情况
         "executionSuccess" : true,		//是否成功
         "nReturned" : 4,			   //有多少符合查询条件的文档
         "executionTimeMillis" : 132,	//查询计划选择和执行所花费的总时间
         "totalKeysExamined" : 0,	    //扫描索引的总数
         "totalDocsExamined" : 200000,	//扫描文档的总数
         "executionStages" : {		   //以阶段的树形方式展示最终执行计划的详情
                 "stage" : "COLLSCAN",  //阶段描述操作符
                 "filter" : {		   //过滤条件
                         "$and" : [
                                 {
                                         "order" : {
                                                 "$lt" : 185005
                                         }
                                 },
                                 {
                                         "order" : {
                                                 "$gt" : 185000
                                         }
                                 }
                         ]
                 },
                 "nReturned" : 4,		//符合查询条件的文档数
                 "executionTimeMillisEstimate" : 133,
                 "works" : 200002,		//执行查询计划阶段所要的工作单位
                 "advanced" : 4,	   //中间或优先返回给父阶段的结果
                 "needTime" : 199997,  //没有提前返回中间结果给母阶段的工作周期数
                 "needYield" : 0,	   //存储层要求查询系统层加锁的时间数
                 "saveState" : 1565,
                 "restoreState" : 1565,
                 "isEOF" : 1,			//查询周期是否检索到了流末
                 "invalidates" : 0,
                 "direction" : "forward",
                 "docsExamined" : 200000	//该查询执行阶段扫描的文档数
         },
         "allPlansExecution" : [ ]
         },
        "serverInfo" : {		//MongoDB实例信息
                "host" : "DESKTOP-RA8F8EV",
                "port" : 27017,
                "version" : "3.4.7",
                "gitVersion" : "cf38c1b8a0a8dca4a11737581beafef4fe120bcd"
        },
        "ok" : 1
}
```

这里有一个要说明的是stage（阶段）有四种值，它们分别代表：

**COLLSCAN  集合扫描**

**IXSCAN  索引键值扫描**

**FETCH 检索文档**

**SHARD_MERGE 从碎片中合并结果**

  返回的一大段信息中，我们可以注意到executionStats中的executionTimeMillis和totalDocsExamined，可以看到的是我们总共花费了132毫秒来选择和执行我们的查询计划，这个计划扫描了20万个文档最终查询到了我们所需要的4个文档，这是极其费时的！那么添加索引在MongoDB中就是非常有必要的了。

## 索引优化

  实际上在集合一开始创建的时候，MongoDB会默认添加上一个基于_id的唯一性索引,该索引**无法被删除**，它可以**预防客户端同时插入两个相同\_id文档的情况**,利用getIndexes()方法我们可以看到这个索引的存在：

```powershell
> db.num.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "saul.num"
        }
]
```

###createIndex()函数
> **ensureIndex和createIndex的区别**

  我们需要加入新的索引来对查询进行优化，在**3.0版本之前**我们都是通过**ensureIndex()**来添加索引的，但是在**3.0版本之后**推荐使用**createIndex()**来添加索引，但是为了**兼容同时保留了ensureIndex()作为createIndex()方法的别名。**在我的这一系列文章中，我都会使用createIndex()方法来创建索引。



`db.collection.createIndex(keys,options)`

**keys**

keys文档包括了被定义为索引键的字段以及用于描述索引类型的值，比如

`db.num.createIndex({order:1})`标识以order为索引键，按升序建立索引

**options**

options文档表示了一系列在创建索引时的控制参数,不同类型的索引也有不同的有效参数,下面是一些常用公用参数，还有很多参数可以以参考[官方文档](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#options)。

| 参数                 | 类型      | 描述                                       |
| ------------------ | ------- | ---------------------------------------- |
| background         | Boolean | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background"    可选参数。  "background" 默认值为**false**。 |
| unique             | Boolean | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**。   |
| name               | String  | 索引名称，不指定则系统用键名拼凑。                        |
| sparse             | Boolean | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | integer | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。         |

### 索引优化后查询效率

现在我们来为num集合创建一个索引，它建立在order上，且是升序的：

```powershell
> db.num.createIndex({order:1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}

//现在我们可以查看到我们的新索引
> db.num.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "saul.num"
        },
        {
                "v" : 2,
                "key" : {
                        "order" : 1
                },
                "name" : "order_1",
                "ns" : "saul.num"
        }
]

//现在我们重新执行查询操作来看看这次发生了什么
> db.num.find({order:{$gt:185000,$lt:185005}}).explain("executionStats")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "saul.num",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "$and" : [
                                {
                                        "order" : {
                                                "$lt" : 185005
                                        }
                                },
                                {
                                        "order" : {
                                                "$gt" : 185000
                                        }
                                }
                        ]
                },
                "winningPlan" : {
                        "stage" : "FETCH",   //可以看到我们现在的母阶段类型，变成了FETCH
                        "inputStage" : {
                                "stage" : "IXSCAN",  //子阶段类型为IXSCAN按索引检索
                                "keyPattern" : {
                                        "order" : 1
                                },
                                "indexName" : "order_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "order" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "order" : [
                                                "(185000.0, 185005.0)"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 4,
                "executionTimeMillis" : 0,//总耗时也几乎为0，过去是132毫秒
                "totalKeysExamined" : 4,//总共扫描了4个索引
                "totalDocsExamined" : 4,//总共扫描了4个文档，过去是200000个
                "executionStages" : {
                        "stage" : "FETCH",
                        "nReturned" : 4,
                        "executionTimeMillisEstimate" : 0,//几乎为0毫秒的估计搜索时间
                        "works" : 5,//花费了4个工作周期
                        "advanced" : 4,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 0,
                        "restoreState" : 0,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 4,//只扫描了4个文档
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 4,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 5,
                                "advanced" : 4,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 0,
                                "restoreState" : 0,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "order" : 1
                                },
                                "indexName" : "order_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "order" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "order" : [
                                                "(185000.0, 185005.0)"
                                        ]
                                },
                                "keysExamined" : 4,
                                "seeks" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0,
                                "seenInvalidated" : 0
                        }
                }
        },
        "serverInfo" : {
                "host" : "DESKTOP-RA8F8EV",
                "port" : 27017,
                "version" : "3.4.7",
                "gitVersion" : "cf38c1b8a0a8dca4a11737581beafef4fe120bcd"
        },
        "ok" : 1
}
```

### 删除索引

如何删除一个索引呢？

db.colloction.dropIndexes()删除所有索引（除了_id上的默认索引外）

db.colloction.dropIndex(索引名)，删除某个索引或某个字段上的索引。

```
//删除order_1可以使用如下方式
db.num.dropIndex("order_1")
db.num.dropIndex({order:1})
```














---
title: 'MongoDB (2):MongoDB shell'
date: 2017-08-19 16:37:31
toc: true
tags:
 - MongoDB
 - NoSQL
---

##  在windows上配置MongoDB

mongodb安装完成之后并不会自动给你建立data/db文件夹，你需要手动建立它，不然你会发现你无法执行mongod命令，来启动js shell操作mongodb

```powershell
2017-08-19T00:58:21.465-0700 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory C:\data\db\ not found., terminating
```




但是一般情况下，你可不想为mongod进程单开一个控制台让他阻塞在那里来监听我们的客户端操作，windows情况下我们一般把它当做服务来操作

```powershell
//执行以下命令

mongod.exe 

--install

--bind_ip 127.0.0.1 		

--logpath "C:\data\dbConf\mongodb.log" //这里是日志文件路径,你必须创建这个文件否则报错

--logappend 			//这里是以追加形式写日志

--dbpath "C:/data/db" 		//这里修改则默认文件夹就不是data/db了，你必须先创建这个文件夹

--port 27017 			//端口 默认就是27017

--serviceName "mongodb"		//服务名

```

<!--more-->


现在服务安装完成了，在cmd下以mongo启动客户端程序就可以连接到mongodb来执行操作了

这里我先切换到一个名叫saul的数据库，方便操作，但是实际上我预先并没有建立这个数据库，在mongodb中创建数据库并不是必须的操作，**第一次插入文档时数据库与集合才会被创建**，因为mongodb并不需要预先定义文档的结构。

```powershell
//切换数据库
> use saul
switched to db saul

//以db查看我们当前的数据库
> db
saul

//show dbs查看所有数据库，此时saul数据库并没有被建立，因为我们并没有进行文档操作
> show dbs
admin  0.000GB
local  0.000GB
test   0.000GB
```

## CRUD操作

### 插入操作

现在我们可以尝试插入数据了,因为mongodb shell其实是一个**javascript shell**，所以我们的语法其实就是**js语法**，我们插入的数据以j**son格式**来表示，但是实际存储的时候是以**bson**来存储这一点我已经在上篇文章说明了。现在我们插入一条数据，它非常简单：

```powershell
> db.users.insert({username:"saul",password:"123456"})
WriteResult({ "nInserted" : 1 })

//插入数据后，我们可以看到saul已经被建立
> show dbs
admin  0.000GB
local  0.000GB
saul   0.000GB
test   0.000GB
```

插入这条数据时你会感到卡了一下的样子，实际上正如我前面所描述----**"第一次插入文档时数据库与集合才会被创建"**,在这个时间内做了三件事：

**1.创建数据库saul**

**2.创建users集合**

然后还有：

**3.插入我们的{username:"saul",password:"123456"}**

在3.2版本之后，mongodb引入了两个新的插入操作，分别是insertOne和insertMany它们分别用于插入一条和多条数据，而且返回值和insert不同，更加清晰一点点。

```powershell
//使用insertOne插入
> db.users.insertOne({"username":"Mary","password":"papap"})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("599906f9ca2fae229a87be0f")
}

//调用pretty可以使返回结果在shell中显示的更加直观
> db.users.find().pretty()
{
        "_id" : ObjectId("5998ff4cca2fae229a87be0e"),
        "username" : "tom",
        "password" : "123456"
}
{
        "_id" : ObjectId("5997fa4e034161a3af28e14e"),
        "username" : "saul",
        "password" : "123"
}
{
        "_id" : ObjectId("599906f9ca2fae229a87be0f"),
        "username" : "Mary",
        "password" : "papap"
}

//使用insertMany插入
> db.users.insertMany([{"username":"Jack","password":"papap"},{"username":"Fule","password":"papap"}])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("599907cbca2fae229a87be10"),
                ObjectId("599907cbca2fae229a87be11")
        ]
}
```



### 查询操作

```javascript
db.collection.find(query, projection)
//query是查询选择器
//projection是投影操作符，它可以指定要返回的键，如果省略它则就是返回所有的键值
```

现在我们可以用find操作来查询到我们刚刚插入的数据：

```powershell
> db.users.find()
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "username" : "saul", "password" : "123456" }
```

我们可以观察shell的响应看到我们插入的saul用户多了一个**_id属性**，它是文档的主键，所有的mongodb文档都要求有一个_id，在我们创建文档时如果没有提供这个字段，则系统自动生成一个ObjectId对象作为主键添加到文档里，既然是主键你可以料到在集合里\_id的值是唯一的。

现在我们以save方法来插入一个值，然后尝试给find方法传入参数来查询到我们的新值：

```powershell
//以save插入值，如果插入的数据中存在主键，则会做更新操作
> db.users.save({username:"tom",password:"123456"})
WriteResult({ "nInserted" : 1 })

//传入选择查询器参数会搜索出符合条件的文档，选择查询器本身也是一个文档
> db.users.find({username:"tom"})
{ "_id" : ObjectId("5998039e034161a3af28e14f"), "username" : "tom", "password" : "123456" }
> db.users.find({username:"saul"})
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "username" : "saul", "password" : "123456" }

//以save方法修改saul的password
> db.users.save({_id: ObjectId("5997fa4e034161a3af28e14e"),username:"saul",password:"123"})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

//可以看到saul的密码已经被改变
> db.users.find({username:"saul"})
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "username" : "saul", "password" : "123" }
```

#### in,or以及and

现在我们做一些新的改变，我们来尝试使用几个新的操作分别是in和or以及and

```powershell
> db.users.find({password:{$in:["papap","123"]}})
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "username" : "saul", "password" : "123" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "username" : "Mary", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Fule", "password" : "papap" }

> db.users.find({$or:[{password:{$in:["papap"]}},{username:"saul"}]})
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "username" : "saul", "password" : "123" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "username" : "Mary", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Fule", "password" : "papap" }

> db.users.find({password:{$in:["123"]},username:"saul"})
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "username" : "saul", "password" : "123" }
```

以上三个命令类似于sql里的：

```sql
select * from users
where password in ("papap","123")

select * from users
where password in ("papap") or username="saul"

select * from users
where password in ("123") and username="saul"
```

### 更新操作

下面我们来看看如何更新一条已有的数据和如何为已有的数据文档插入新的属性

```embeddedjs
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

首先为2个必选参数:

**第一个参数为查询选择器**
**第二个则为要添加的元素**

还有三个可选参数：

**upsert参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。**

**multi参数多项更新，因为MongoDB默认更新操作只会应用于查询选择器匹配到的第一个文档，而如果你想更新所有匹配到的文档，就必须显示的把多项更新设置为true**

**writeConcern参数为抛出异常的级别**

下面我们来看看更新操作：

```powershell
> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "123456" }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "username" : "Mary", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Fule", "password" : "papap" }

//以update更新
> db.users.update( {password:"papap"} ,{password:"789"})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

/*
可以看到更新操作好像与你想象中的不同，首先你可以观察到这种更新是对整条文档进行更新，第二是我们看到它只更新了第一条查找到的文档
*/
> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "123456" }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "papap" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Fule", "password" : "papap" }
```

#### set和unset操作

以上方法下所做的操作肯定不是你想要的，实际上我们必须采用**$set操作**来更新一条记录，而不是全部文档，第二我们需要制定mutip参数为true,如果你对js熟悉，会发现这有好几种方法~

```powershell
//利用$set操作更新，按顺序分别制定了upsert和multi
> db.users.update( {password:"papap"} ,{$set:{password:"789"}},false,true)
WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })

> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "123456" }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Fule", "password" : "789" }

//js函数也可以直接指定参数
> db.users.update( {password:"123456"} ,{$set:{password:"654321"}},{multi:true})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "654321" }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Fule", "password" : "789" }
```

实际上我们还可以用$set操作来添加文档以前没有的属性，如果不想要这个属性就可以用unset删除

```powershell
//插入新的属性favoriteLanguage,可以看到mongodb中同一集合的文档结构不需要相同
> db.users.update({password:"789"},{$set:{favoriteLanguage:["c++","scala"]}},true,true)
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "654321" }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789", "favoriteLanguage" : [ "c++", "scala" ] }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789", "favoriteLanguage" : [ "c++", "scala" ] }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Flue", "password" : "789", "favoriteLanguage" : [ "c++", "scala" ] }

//现在我们用unset操作去除新的属性favoriteLanguage
> db.users.update({password:"789"},{$unset:{favoriteLanguage:1}},true,true)
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "654321" }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Flue", "password" : "789" }
>
```

#### inc操作

现在我们来看一个新的操作，$inc(increase)，它在键不存在的时候创建这个键，但是如果键已经存在就往键值上加

```powershell
//在age不存在的情况下mongodb为tom增加了一个字段age，值就是我们设定的22
> db.users.update({username:"tom"},{$inc:{age:22}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "654321", "age" : 22 }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Flue", "password" : "789" }

//可看到在age已经存在的情况下,它减少了1,即我们指定的-1
> db.users.update({username:"tom"},{$inc:{age:-1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find()
{ "_id" : ObjectId("5998ff4cca2fae229a87be0e"), "username" : "tom", "password" : "654321", "age" : 21 }
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Flue", "password" : "789" }
```

#### rename操作

有时候我们需要对键名进行更改操作，这时候我们可以用$rename

```powershell
//可以用rename来更改键的名称
> db.users.update({username:"tom"},{$rename:{age:"myage"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

//可以看到的age键，已经被更改
> db.users.find({username:"tom"}).pretty()
{
        "_id" : ObjectId("5998ff4cca2fae229a87be0e"),
        "username" : "tom",
        "password" : "654321",
        "myage" : 21
}
```

#### setOnInsert操作

这个方法可以从字面上来理解，只有在Insert操作时发生Set操作，发生的是update操作则不作任何操作！说起来有点抽象，我们来看下面这个例子吧：

```powershell
//可以看到，如果文档本身就存在，那么setOnInsert毫无作用，因为此时根本不会触发Insert操作
> db.users.update({username:"tom"},{$setOnInsert:{myage:14}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
> db.users.find({username:"tom"}).pretty()
{
        "_id" : ObjectId("5998ff4cca2fae229a87be0e"),
        "username" : "tom",
        "password" : "654321",
        "myage" : 21
}
//尝试更新一个不存在的键值
> db.users.update({username:"tom"},{$setOnInsert:{age:14}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
> db.users.find({username:"tom"}).pretty()
{
        "_id" : ObjectId("5998ff4cca2fae229a87be0e"),
        "username" : "tom",
        "password" : "654321",
        "myage" : 21
}

//现在我们来尝试一个插入操作，我们此时把upsert设置为true，这代表了我们如果找不到这个文档就尝试插入操作！
> db.users.update({username:"Jerry"},{$setOnInsert:{age:14}},true)
WriteResult({
        "nMatched" : 0,
        "nUpserted" : 1,
        "nModified" : 0,
        "_id" : ObjectId("599948b17f378622835add1e")
})
//我们可以看到的是，setOnInsert成功的执行了
> db.users.find({username:"Jerry"}).pretty()
{
        "_id" : ObjectId("599948b17f378622835add1e"),
        "username" : "Jerry",
        "age" : 14
}
```

#### 数组更新操作

  除了一般的单值字段以外，我们可能会遇到一个数组字段的情况，比如一张唱片里包含的歌曲，一篇文章的tags等等！所以mongodb当然也提供了专门对数组的操作。

现在我们切换一下集合到articles,可以看到这里面有一篇文章，该文章包括了一个tags字段。

```powershell
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995117ca2fae229a87be13"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs"
        ]
}
```

我们如何对tags来做一些update操作呢？首先我们先接触一下两个操作，他们分别是：

**$pull**

**$push**

字面意义上你应该已经知道他们是做什么的了，但是它们都只能对**单个值做操作**

```powershell
//对于tags我们给它插入两个值吧，对于push我们只能一个一个为tags增加值
> db.articles.update({},{$push:{tags:"News"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.update({},{$push:{tags:"America"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

//我们可以看到我们增加了两个值tags
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995117ca2fae229a87be13"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs",
                "News",
                "America"
        ]
}


//而对于pull我们也只能一个个将tag移除tags
> db.articles.update({},{$pull:{tags:"America"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.update({},{$pull:{tags:"News"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995117ca2fae229a87be13"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs"
        ]
}
```

对于pull和push，你也许会感到很无奈，学数据库的时候你就已经学过了，尽量减少插入语句执行的次数，因为这是极其低效的，**实际上我们如果push/pull一个数组，mongodb会把这个数组当做是一个值来对待**，比如：

```powershell
//我想这一定不是你想看到的结果
> db.articles.update({},{$push:{tags:["News","America"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.articles.find().pretty()
{
        "_id" : ObjectId("59995117ca2fae229a87be13"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs",
                [
                        "News",
                        "America"
                ]
        ]
}
```

实际上如果push一个数组，则我们可以用两个新的操作：

**$pushAll**

**$pullAll**

```powershell
//我们先把前面插入的数组数据给删掉
> db.articles.update({},{$pull:{tags:["News","America"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

//现在我们执行pushAll操作，可以看到我们一次性插入了多个值到tags中
> db.articles.update({},{$pushAll:{tags:["News","America"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995117ca2fae229a87be13"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs",
                "News",
                "America"
        ]
}

//相同原理我们可以一次性删除多个值
> db.articles.update({},{$pullAll:{tags:["News","America"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995117ca2fae229a87be13"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs"
        ]
}
```

对于push和pushAll操作，其实它们都不会避免重复的值，我们还要介绍一个**addToSet操作**

**1.它用来做插入操作且会避免重复的值**

**2.它可以与$each操作结合来插入一个数组（同pushAll）**

**3.对于与$each结合的数组，它会分别判断每一个值是否重复**

这是什么意思呢？请看下面的操作：

```powershell
//现在我们插入了往tags插入了一个重复值
> db.articles.update({},{$push:{tags:"political"}})

//可以看到重复值居然被插入了
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995ba4ca2fae229a87be14"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "political",
                "commentary",
                "affairs",
                "political"
        ]
}

> db.articles.update({},{$pull:{tags:"political"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995ba4ca2fae229a87be14"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "commentary",
                "affairs"
        ]
}

//从数组中移除political
> db.articles.update({},{$pull:{tags:"political"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

//现在以addToSet增加值
> db.articles.update({},{$addToSet:{tags:"political"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

//第二次操作，可以看到nModified操作为0，即没有插入数据
> db.articles.update({},{$addToSet:{tags:"political"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })

//当然我们可以插入多条值！，需要用$each操作符搭配
> db.articles.update({},{$addToSet:{tags:{$each:["News","America"]}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.articles.find().pretty()
{
        "_id" : ObjectId("59995ba4ca2fae229a87be14"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "commentary",
                "affairs",
                "political",
                "News",
                "America"
        ]
}

//依然会判读重复值
> db.articles.update({},{$addToSet:{tags:{$each:["News","America"]}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })

//但是对于数组里的新值它依然会插入
> db.articles.update({},{$addToSet:{tags:{$each:["News","White House"]}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.articles.find().pretty()
{
        "_id" : ObjectId("59995ba4ca2fae229a87be14"),
        "title" : "Is Trump a Good President?",
        "tags" : [
                "commentary",
                "affairs",
                "political",
                "News",
                "America",
                "White House"
        ]
}
```

### 删除操作

在3.2版本之前MongoDB都是用remove来做删除操作

```embeddedjs
db.collection.remove(
   <query>,						//选择过滤器
   {
     justOne: <boolean>,		//如果未true，只删除第一个匹配到的文档
     writeConcern: <document>,	 //抛出异常的级别
     collation: <document>		//3.4版本增加，用于指定排序规则
   }
)
```

在3.2版本之后增加了两个操作符，等是把remove以justOne选项的值拆分为了两个操作

**deleteOne**		//只删除一个文档

**deleteMany**		//删除多条文档

```powershell
//以remove来做删除操作，不指定justOne
> db.users.remove({$or:[{age:{$lt:15}},{myage:{$lt:22}}]})
WriteResult({ "nRemoved" : 2 })

> db.users.find()
{ "_id" : ObjectId("5997fa4e034161a3af28e14e"), "password" : "456" }
{ "_id" : ObjectId("599906f9ca2fae229a87be0f"), "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be10"), "username" : "Jack", "password" : "789" }
{ "_id" : ObjectId("599907cbca2fae229a87be11"), "username" : "Flue", "password" : "789" }

//指定justOne,可以看到只移除一条记录
> db.users.remove({password:"789"},true)
WriteResult({ "nRemoved" : 1 })

//3.4版本后才有得deleteOne和deleteMany操作
> db.users.deleteOne({password:"789"})
{ "acknowledged" : true, "deletedCount" : 1 }

> db.users.deleteMany({password:"789"})
{ "acknowledged" : true, "deletedCount" : 1 }

```






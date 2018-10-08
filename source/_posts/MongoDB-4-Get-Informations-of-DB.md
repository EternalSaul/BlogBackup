---
title: 'MongoDB (4):Get Information of DB'
date: 2017-08-23 11:08:20
toc: true
tags:
 - MongoDB
 - NoSQL
---

##  简单的回顾

   如果你过oracle数据库，我们知道可以用一些系统视图来查询到当前数据库的状态云云，数据库状态在实际运用中是调整数据库性能的重要依据。作为一款成熟的数据库产品，MongoDB当然也提供了这种功能，这篇文章就是介绍如何用MongoDB shell去查询我们的数据库状态，前面我们已经看到了一些简单的查询，这里我们先回顾一下这些命令：

```powershell
//使用db命令显示当前使用的数据库
> db
test

//改变数据库
> use saul
switched to db saul

//先我们可以看到，当前数据库已经变成了db
> db
saul

//显示数据库名
> show dbs
admin  0.000GB
local  0.000GB
saul   0.004GB
test   0.000GB

//显示当前数据库的集合名
> show collections
articles
num
users
```

<!--more-->

## 详情查询

###   数据库详情

  以上都是简单的信息，如果我们想知道一个数据库或者一个集合比较详细的信息，比如你想知道索引具体占了多少空间，可以使用以下的命令：

```powershell
/*
db.stats(scale),scale用来表示显示数据大小的单位，默认单位是字节
若传入scale，则是scale bytes,比如传入1024，表示输出数据的文档大小是1K字节
*/

//访问一个数据库的信息
> db.stats()
{
        "db" : "saul",				//数据库名
        "collections" : 3,			//集合总数
        "views" : 0,				//3.4版本之后，数据库中只读的视图总数
        "objects" : 200002,			//整个数据库包含的文档数
        "avgObjSize" : 37.000664993350064,//平均文档大小，单位是byte（这个字段不会被scale所影响）
        "dataSize" : 7400207,		//该数据库中未压缩数据的总大小，单位byte
        "storageSize" : 2670592,	//为存储文档，分配给集合的总数据大小，单位byte
        "numExtents" : 0,		   //所有集合的扩展数
        "indexes" : 3,			  //所有集合的索引数
        "indexSize" : 1921024,     //索引的总大小，单位byte
        "ok" : 1				
}
```

>views 请参考，[官方文档](https://docs.mongodb.com/manual/core/views/index.html)

### 集合详情

如果是集合：

```powershell
> db.num.stats()
{
        "ns" : "saul.num",    //当前集合的命名空间
        "size" : 7400000,     //当前集合所有记录在内存中所占的大小
        "count" : 200000,     //集合中的总文档（对象）数
        "avgObjSize" : 37,    //平均每个文档（对象）所占的大小，单位是byte（这个字段不会被scale所影响）
        "storageSize" : 2596864,//为存储文档，分配给集合的总数据大小
        "capped" : false,//集合是否受限，见下面的注释capped collection
        "wiredTiger" : {//WiredTiger引擎信息，3.0之后才有
                "metadata" : {
                        "formatVersion" : 1
                },
                "creationString" : "access_pattern_hint=none,allocation_size=4KB,app_metadata=(formatVersion=1),block_allocation=best,block_compressor=snappy,cache_resident=false,checksum=on,colgroups=,collator=,columns=,dictionary=0,enc
ryption=(keyid=,name=),exclusive=false,extractor=,format=btree,huffman_key=,huffman_value=,ignore_in_memory_cache_size=false,immutable=false,internal_item_max=0,internal_key_max=0,internal_key_truncate=true,internal_page_max=4KB,key_form
at=q,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=32KB,leaf_value_max=64MB,log=(enabled=true),lsm=(auto_throttle=true,bloom=true,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=false,chunk_count_limit=0,chunk_
max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=10m,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=false,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=90,type=file,
value_format=u",
                "type" : "file",
                "uri" : "statistics:table:collection-6-3691317590485950066",
                "LSM" : {
                        "bloom filter false positives" : 0,
                        "bloom filter hits" : 0,
                        "bloom filter misses" : 0,
                        "bloom filter pages evicted from cache" : 0,
                        "bloom filter pages read into cache" : 0,
                        "bloom filters in the LSM tree" : 0,
                        "chunks in the LSM tree" : 0,
                        "highest merge generation in the LSM tree" : 0,
                        "queries that could have benefited from a Bloom filter that did not exist" : 0,
                        "sleep for LSM checkpoint throttle" : 0,
                        "sleep for LSM merge throttle" : 0,
                        "total size of bloom filters" : 0
                },
                "block-manager" : {
                        "allocations requiring file extension" : 0,
                        "blocks allocated" : 0,
                        "blocks freed" : 0,
                        "checkpoint size" : 2547712,
                        "file allocation unit size" : 4096,
                        "file bytes available for reuse" : 32768,
                        "file magic number" : 120897,
                        "file major version number" : 1,
                        "file size in bytes" : 2596864,
                        "minor version number" : 0
                },
                "btree" : {
                        "btree checkpoint generation" : 3,
                        "column-store fixed-size leaf pages" : 0,
                        "column-store internal pages" : 0,
                        "column-store variable-size RLE encoded values" : 0,
                        "column-store variable-size deleted values" : 0,
                        "column-store variable-size leaf pages" : 0,
                        "fixed-record size" : 0,
                        "maximum internal page key size" : 368,
                        "maximum internal page size" : 4096,
                        "maximum leaf page key size" : 2867,
                        "maximum leaf page size" : 32768,
                        "maximum leaf page value size" : 67108864,
                        "maximum tree depth" : 0,
                        "number of key/value pairs" : 0,
                        "overflow pages" : 0,
                        "pages rewritten by compaction" : 0,
                        "row-store internal pages" : 0,
                        "row-store leaf pages" : 0
                },
                "cache" : {
                        "bytes currently in the cache" : 45446,
                        "bytes read into cache" : 28200,
                        "bytes written from cache" : 0,
                        "checkpoint blocked page eviction" : 0,
                        "data source pages selected for eviction unable to be evicted" : 0,
                        "hazard pointer blocked page eviction" : 0,
                        "in-memory page passed criteria to be split" : 0,
                        "in-memory page splits" : 0,
                        "internal pages evicted" : 0,
                        "internal pages split during eviction" : 0,
                        "leaf pages split during eviction" : 0,
                        "modified pages evicted" : 0,
                        "overflow pages read into cache" : 0,
                        "overflow values cached in memory" : 0,
                        "page split during eviction deepened the tree" : 0,
                        "page written requiring lookaside records" : 0,
                        "pages read into cache" : 3,
                        "pages read into cache requiring lookaside entries" : 0,
                        "pages requested from the cache" : 2,
                        "pages written from cache" : 0,
                        "pages written requiring in-memory restoration" : 0,
                        "tracked dirty bytes in the cache" : 0,
                        "unmodified pages evicted" : 0
                },
                "cache_walk" : {
                        "Average difference between current eviction generation when the page was last considered" : 0,
                        "Average on-disk page image size seen" : 0,
                        "Clean pages currently in cache" : 0,
                        "Current eviction generation" : 0,
                        "Dirty pages currently in cache" : 0,
                        "Entries in the root page" : 0,
                        "Internal pages currently in cache" : 0,
                        "Leaf pages currently in cache" : 0,
                        "Maximum difference between current eviction generation when the page was last considered" : 0,
                        "Maximum page size seen" : 0,
                        "Minimum on-disk page image size seen" : 0,
                        "On-disk page image sizes smaller than a single allocation unit" : 0,
                        "Pages created in memory and never written" : 0,
                        "Pages currently queued for eviction" : 0,
                        "Pages that could not be queued for eviction" : 0,
                        "Refs skipped during cache traversal" : 0,
                        "Size of the root page" : 0,
                        "Total number of pages currently in cache" : 0
                },
                "compression" : {
                        "compressed pages read" : 1,
                        "compressed pages written" : 0,
                        "page written failed to compress" : 0,
                        "page written was too small to compress" : 0,
                        "raw compression call failed, additional data available" : 0,
                        "raw compression call failed, no additional data available" : 0,
                        "raw compression call succeeded" : 0
                },
                "cursor" : {
                        "bulk-loaded cursor-insert calls" : 0,
                        "create calls" : 1,
                        "cursor-insert key and value bytes inserted" : 0,
                        "cursor-remove key bytes removed" : 0,
                        "cursor-update value bytes updated" : 0,
                        "insert calls" : 0,
                        "next calls" : 0,
                        "prev calls" : 1,
                        "remove calls" : 0,
                        "reset calls" : 1,
                        "restarted searches" : 0,
                        "search calls" : 0,
                        "search near calls" : 0,
                        "truncate calls" : 0,
                        "update calls" : 0
                },
                "reconciliation" : {
                        "dictionary matches" : 0,
                        "fast-path pages deleted" : 0,
                        "internal page key bytes discarded using suffix compression" : 0,
                        "internal page multi-block writes" : 0,
                        "internal-page overflow keys" : 0,
                        "leaf page key bytes discarded using prefix compression" : 0,
                        "leaf page multi-block writes" : 0,
                        "leaf-page overflow keys" : 0,
                        "maximum blocks required for a page" : 0,
                        "overflow values written" : 0,
                        "page checksum matches" : 0,
                        "page reconciliation calls" : 0,
                        "page reconciliation calls for eviction" : 0,
                        "pages deleted" : 0
                },
                "session" : {
                        "object compaction" : 0,
                        "open cursor count" : 1
                },
                "transaction" : {
                        "update conflicts" : 0
                }
        },
        "nindexes" : 1,	   //该集合的索引个数，所有集合都至少有默认的_id索引
        "totalIndexSize" : 1859584,    //索引总的索引大小
        "indexSizes" : {    //指明了集合中存在的所有索引的索引键和其所占大小
                "_id_" : 1859584
        },
        "ok" : 1
}
```

>Capped Collection   一个有最大存储上限的集合，当数据达到写满集合时，再写数据就会自动覆盖最老的数据，请见[官方文档](https://docs.mongodb.com/manual/core/capped-collections/)

## 这些命令如何工作

  你如果接触过oracle的话会知道orcle实例本身维护了一些表，我们所查询的信息实际上是对它的视图来进行一个查询，那么相应的MongoDB也不能无中生有给你临时计算出这些数据，实际上我们的stats()方法是对一个名为$cmd的特殊虚集合做查询操作。

  我前面提到过在3.0版本之后ensureIndex其实是createIndex方法的别名，实际上对于stats，它也是一个辅助的方法，你可以把它也看做一个别名,实时是这样的，db.stats()等同于：

```powershell
//实际上stats封装了该方法
> db.runCommand({dbstats:1})
{
        "db" : "saul",
        "collections" : 3,
        "views" : 0,
        "objects" : 200002,
        "avgObjSize" : 37.000664993350064,
        "dataSize" : 7400207,
        "storageSize" : 2670592,
        "numExtents" : 0,
        "indexes" : 3,
        "indexSize" : 1921024,
        "ok" : 1
}

//传入scale参数
> db.runCommand({dbstats:1,scale:1024})
{
        "db" : "saul",
        "collections" : 3,
        "views" : 0,
        "objects" : 200002,
        "avgObjSize" : 37.000664993350064,
        "dataSize" : 7226.7646484375,
        "storageSize" : 2608,
        "numExtents" : 0,
        "indexes" : 3,
        "indexSize" : 1876,
        "ok" : 1
}
```

对于db.collection.stats操作也是一样的

```powershell
//db.num.stats(1024) 等同于：
db.runCommand({collstats:"num",scale:1024})
```

实际上这些都不是我们要讨论和值得思索的，正在值得我们探讨的应该是runCommand函数是怎么工作的，由于js的优良特性，函数就是一个文本，所以在mongodb shell中我们可以看到这个文本，只要我们不加上()就可以直接显示这个函数的源码:

```powershell
> db.runCommand
function (obj, extra, queryOptions) {
        var mergedObj = (typeof(obj) === "string") ? this._mergeCommandOptions(obj, extra) : obj;
        // if options were passed (i.e. because they were overridden on a collection), use them.
        // Otherwise use getQueryOptions.
        var options =
            (typeof(queryOptions) !== "undefined") ? queryOptions : this.getQueryOptions();
        var res;
        try {
            res = this._runCommandImpl(this._name, mergedObj, options);
        } catch (ex) {
            // When runCommand flowed through query, a connection error resulted in the message
            // "error doing query: failed". Even though this message is arguably incorrect
            // for a command failing due to a connection failure, we preserve it for backwards
            // compatibility. See SERVER-18334 for details.
            if (ex.message.indexOf("network error") >= 0) {
                throw new Error("error doing query: failed: " + ex.message);
            }
            throw ex;
        }
        return res;
    }

//可以看到runCommand实际上调用了_runCommandImpl
> db._runCommandImpl
function (name, obj, options) {
        return this.getMongo().runCommand(name, obj, options);
    }

/*
在新版的mongodb shell中我们并不能直接看到我们查询了$cmd集合，查看db.getMongo().runCommand源码，可观察到这段源码是编译后的c++代码，但是我们可以用反推方式这里实际就是执行了一个对$cmd的查询
*/
> db.getMongo().runCommand
function runCommand() {
    [native code]
}

//实际上这里的机器码应该是执行了，一个查询操作，因为我们如果对$cmd集合直接使用查询操作我们也能得到类似的结果
> db.$cmd.findOne({dbStats:1})
{
        "db" : "saul",
        "collections" : 3,
        "views" : 0,
        "objects" : 200002,
        "avgObjSize" : 37.000664993350064,
        "dataSize" : 7400207,
        "storageSize" : 2670592,
        "numExtents" : 0,
        "indexes" : 3,
        "indexSize" : 1921024,
        "ok" : 1
}

//同理对集合查询可以修改为这样标识，直接查询$cmd集合
> db.$cmd.findOne({collStats:"num"})
```


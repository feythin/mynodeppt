title: mongodb分析
speaker: 刘飞宇(feythin.lau)
url: http://www.liufeiyu.cn 
transition: horizontal3d
files: /js/demo.js,/css/moon.css,/js/zoom.js
theme: dark

[slide style="background-image:url('/img/blackops.jpg');background-repeat:no-repeat;background-position: center center;"]

# 线上mongodb分析
## 刘飞宇(feythin.lau)
<small>Weibo: <a href="http://weibo.com/539167012">@Feythin</a></small><br>
<small>Email: <a href="mailto:liufeyu@55tuan.com">liufeiyu@55tuan.com</a></small><br>
<small>Blog: <a href="http://liufeiyu.cn">ww.liufeiyu.cn</a></small><br>
<small><i class="fa fa-github"></i>github:  <a href="https://github.com/feythin">@feythin</a></small><br>

[slide data-transition="horizontal3d"]
## 窝窝mongodb现状分析
----
* 线上mongodb架构 {:&.moveIn}
* 线上mongodb现状
* 线上mongodb分析
* 建议 

[slide]

## 线上mongodb架构
----
#### 线上mongodb架构图
![replication](/img/replication.jpg "replication")

[slide]

## 线上mongodb现状
----
#### mongodb-172 mongodb-173 架构图
![replication](/img/mongo-online.jpg "replication")

[slide]

## 分析：
----
* 多个mongo数据的primary在一台机器上 {:&.moveIn}

  如果读写操作平凡的话，很容易造成这台都是主的服务器磁盘io／内存等压力。

[slide]

## 解决办法：
----
* 通过设置优先级，把每个mongod进程的主分散放到多台机器 

<pre><code>
/* 优先级配置方法： */
    cfg = rs.conf();
    cfg.members[0].priority = 1;
    cfg.members[1].priority = 2;
    rs.reconfig(cfg);
</code></pre>

[slide]

## 线上mongo99，goodscenter库 慢查询分析
----

```json
{ 
 "op" : "remove",
 "ns" : "goodscenter.StatOldBillScoreTask", 
 "query" : { "city_id" : 22 },
 "ndeleted" : 7715, 
 "keyUpdates" : 0, 
 "numYield" : 5, 
 "lockStats" : { "timeLockedMicros" : { "r" : NumberLong(0), "w" : NumberLong(1086113) }, 
              "timeAcquiringMicros" : { "r" : NumberLong(0), "w" : NumberLong(43406) } },
 "millis" : 551, "execStats" : {  }, "ts" : ISODate("2015-06-07T16:42:03.824Z"),
 "client" : "172.16.52.23", "allUsers" : [ ], "user" : ""
}
```
* 从日志来看，这条是先通过查询city_id来删除相关文档的，耗时551毫秒。删除了7715条。 {:&.moveIn}

[slide]
## 线上mongo99，goodscenter库 慢查询分析
----
* 问题: 

  * 删除数量大 --7715条。 {:&.moveIn}
  * 延迟大 --551ms
  * 大量的删除操作造成空间的极度浪费 --(mongodb的删除操作后是不回收磁盘空间的)
  * 查询city_id的时候没有看到索引(如删除goodscenter.StatTotalScoreTaskB文档)。 
  * 索引有重复

[slide]
## 线上mongo99，goodscenter库 慢查询分析
----
<pre><code>
goodscenter:PRIMARY> db.StatOldBillScoreTask.getIndexKeys()
[
  {
    "_id" : 1
  },
  {
    "city_id" : 1,
    "score" : -1
  },
  {
    "city_id" : 1,
    "channel_id" : 1,
    "score" : -1
  }
]
</code></pre>

[slide]
## 索引过多，占用过大

<pre><code>
usercomment:PRIMARY> db.stats()
{
  "db" : "usercomment",
  "collections" : 15,
  "objects" : 53685684,
  "avgObjSize" : 1053.6167006459302,
  "dataSize" : 56564133248,
  "storageSize" : 58151329424,
  "numExtents" : 144,
  "indexes" : 42,  #索引数量
  "indexSize" : 56188505296,  #byte 大概52G
  "fileSize" : 133012914176,
  "nsSizeMB" : 16,
  "dataFileVersion" : {
    "major" : 4,
    "minor" : 5
  },
  "extentFreeList" : {
    "num" : 0,
    "totalSize" : 0
  },
  "ok" : 1
}
</code></pre>

[slide]

## mongodb BTree树索引
---
![replication](/img/b-index.png "replication")

[slide]
### 索引例子
数据集

```json
{ 
  "_id" : ObjectId("556fea643c37bf43d763cd71"), 
  "enable" : 1, 
  "hid" : 55, 
  "name" : "abc", 
  "date" : 20150512,
  "position" : "china",
  "price" : 6793, 
  "writeTime" : ISODate("2015-06-04T14:04:18.955Z") 
}
```
[slide]
## 索引例子

什么索引都没有的时候我们查询（默认的有_id索引）：
```
db.test.find({"hid":23, "date":{"$gte":20150513, "$lte":20150515}, "enable":1}).sort({"price":1}).limit(10).explain()
{ "cursor" : "BasicCursor",
  "n" : 10,
  "nscannedObjects" : 100000,
  "nscanned" : 100000,   ##全表扫描
  "nscannedObjectsAllPlans" : 100000,
  "nscannedAllPlans" : 100000,
  "scanAndOrder" : true,  ##内存排序
  "nYields" : 781,
  "nChunkSkips" : 0,
  "millis" : 147,
  "server" : "mongodbtest2.55tuan.me:27021",
  "filterSet" : false
}
```
这个正常，没有用到索引全表扫描，最后得出10条结果，其中scanAndOrder表示我们进行了扫描并排序的操作，这是非常消耗cpu和内存的。

[slide]
## 索引例子
我们创建一个hid的索引。
```
rs1:PRIMARY> db.test.ensureIndex({hid:1})

{
  "cursor" : "BtreeCursor hid_1",
  "n" : 10,
  "nscannedObjects" : 993,
  "nscanned" : 993, ##扫描文档
  "nscannedObjectsAllPlans" : 993,
  "nscannedAllPlans" : 993,
  "scanAndOrder" : true,  ##内存排序
  "indexOnly" : false,
  "nYields" : 7,
  "millis" : 8,
}
```
我们看到从全表10万扫描，到993个扫描，使用了刚才创建的hid索引。同时查询时间也下降了。

[slide]
## 索引例子
我们再建立hid＋date的联合索引：
```
rs1:PRIMARY> db.test.ensureIndex({hid:1,date:1})
{
  "cursor" : "hid_1_date_1",
  "n" : 10,
  "nscannedObjects" : 174,
  "nscanned" : 174,  ##扫描文档
  "nscannedObjectsAllPlans" : 446,
  "nscannedAllPlans" : 447,
  "scanAndOrder" : true, ##内存排序
  "nYields" : 4,
  "nChunkSkips" : 0,
  "millis" : 5,
}
```
看到使用了hid＋date的联合索引，扫描文档数也从993个下降到174个。

[slide]
## 索引例子
再建立一个hid＋date＋enable的联合索引：
```
rs1:PRIMARY> db.test.ensureIndex({hid:1,date:1,enable:1})

{  
  "cursor" : "BtreeCursor hid_1_date_1_enable_1",
  "n" : 10,
  "nscannedObjects" : 166,
  "nscanned" : 170,  ##扫描文档
  "nscannedObjectsAllPlans" : 604,
  "nscannedAllPlans" : 609,
  "scanAndOrder" : true, ##内存排序
  "nYields" : 6,
  "nChunkSkips" : 0,
  "millis" : 8,
}
```
我们看到使用了hid＋date＋enable联合索引，文档扫描次数也小了，但是不是很明显。

[slide]
## 索引例子

再创建hid＋date＋enable＋price的联合索引：
```
rs1:PRIMARY> db.test.ensureIndex({hid:1,date:1,enable:1,price:1})

{
  "cursor" : "BtreeCursor hid_1_date_1_enable_1",
  "n" : 10,
  "nscannedObjects" : 166,
  "nscanned" : 170,
  "nscannedObjectsAllPlans" : 770,
  "nscannedAllPlans" : 779,
  "scanAndOrder" : true,
  "nYields" : 8,
  "nChunkSkips" : 0,
  "millis" : 6,
}
```
看到优化不明显，还是使用hid＋date＋enable的索引，似乎我们的索引已经没有优化的了。

[slide]
## 索引例子

我们在看看调整一下索引顺序，使用hid＋price＋date＋enable看看：
```
rs1:PRIMARY> db.test.ensureIndex({hid:1,price:1,date:1,enable:1})
{
  "cursor" : "BtreeCursor hid_1_price_1_date_1_enable_1",
  "isMultiKey" : false,
  "n" : 10,
  "nscannedObjects" : 10,
  "nscanned" : 126,  ## 扫描文档数也少了
  "nscannedObjectsAllPlans" : 50,
  "nscannedAllPlans" : 170,
  "scanAndOrder" : false,  ##已经不是内存索引了
  "indexOnly" : false,
  "nYields" : 0,
  "nChunkSkips" : 0,
  "millis" : 3,
}
```
看到nscannedObjects已经是和返回的文档数一样的了，使用了hid＋price＋date＋enable的联合索引.我们从索引中查找到126条记录放到内存里，其中，我们只查找其中的10条就返回记录了。同时 scanAndOrder这个字段也成为了false，表示我们没有做在内存里的扫描和排序操作，将会降低cpu和内存的消耗，似乎结果很满意。

[slide]

## 索引例子

enable的值只有两个要么是1，要么是0，如果我们在这种差别不是很大的字段建索引是浪费索引空间和内存的，对与我们定位记录是没有太大帮助的，而且也影响插入性能，我们去掉这个试试。

```
rs1:PRIMARY> db.test.dropIndex("hid_1_price_1_date_1_enable_1")
rs1:PRIMARY> db.test.ensureIndex({hid:1,price:1,date:1})
rs1:PRIMARY> db.test.find({"hid":23, "date":{"$gte":20150513, "$lte":20150515}, "enable":1}).sort({"price":1}).limit(10).explain()
{
  "cursor" : "BtreeCursor hid_1_price_1_date_1",
  "isMultiKey" : false,
  "n" : 10,
  "nscannedObjects" : 10,
  "nscanned" : 126,
  "nscannedObjectsAllPlans" : 50,
  "nscannedAllPlans" : 170,
  "scanAndOrder" : false,
  "indexOnly" : false,
  "nYields" : 0,
  "nChunkSkips" : 0,
  "millis" : 2,
}
```
看到还是从索引中查找到10条记录直接返回了。
所以更好的方案，根据查询，创建一个包含有低选择性字段的混合索引

[slide]

## 索引例子

我们还可以在查询的时候强制使用某种索引：
```
rs1:PRIMARY> db.test.find({hid:{$gt:55},date:20150512}).sort({enable:1}).hint({hid:1,date:1,enable:1}).explain()
```
这里我们强制使用hid＋date＋enable索引。


[slide]
## mongodb 日志文件过大，没有定时拆分，删除

```
[feiyu.liu@Mongodb-99 ~]$ ls -alh /data/mongodb/log/mongodb.log
-rw-rw-r-- 1 mongodb mongodb 21G Jun 10 17:29 /data/mongodb/log/mongodb.log
```

[slide]
## 总结
* 1.多个mongodb的primary在一台机器上，可以分散，如让一台机器一个primary，一个secondary等，降低对一台服务器磁盘压力（有的磁盘io还好)  {:&.moveIn}
* 2.查询优化，开慢查询后，我们报告需要优化的只是开发写的慢查询，没有对源头来审查这种。如果可以，需要对开发体检数据库操作的每条语句，DBA需审查语句的合理规范等，这样可以：
             1）、避免性能太差的sql进入生产系统,导致整体性能降低. 2）、检查开发设计的索引是否合理,是否需要添加索引
* 3.当磁盘达到70%以上之前，我们应该有后续的处理办法来扩展或者升级或者客户端修改之类的。
* 4.有些mongo的日志文件太大20多G，如果日志对我们没有用的话，可以考虑删除，或者拆分。
* 5.有必要整理一些数据库操作的常用规范：如哪些操作是要避免的，哪些操作是可以优化的等。

[slide style="background-image:url('/img/blackops.jpg');background-repeat:no-repeat;background-position: center center;"]
# Q&A






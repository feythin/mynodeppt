title: mongodb测试
speaker: 刘飞宇(feythin.lau)
url: http://www.liufeiyu.cn 
transition: horizontal3d
files: /js/demo.js,/css/moon.css,/js/zoom.js
theme: dark

[slide style="background-image:url('/img/blackops.jpg');background-repeat:no-repeat;background-position: center center;"]

# mongodb性能测试
## 刘飞宇(feythin.lau)
<small>Weibo: <a href="http://weibo.com/539167012">@Feythin</a></small><br>
<small>Email: <a href="mailto:liufeyu@55tuan.com">liufeiyu@55tuan.com</a></small><br>
<small>Blog: <a href="http://liufeiyu.cn">ww.liufeiyu.cn</a></small><br>
<small><i class="fa fa-github"></i>github:  <a href="https://github.com/feythin">@feythin</a></small><br>

[slide data-transition="horizontal3d"]
## mongodb测试简介
----
* mongodb测试环境 {:&.moveIn}
* mongodb测试架构 
* mongodb测试点 
* mongodb测试数据集 
* mongodb测试结果 

[slide]

## mongodb测试环境
----
### 软件环境
* OS: CentOS release 6.5  kern：2.6.32-431.3.1.el6.x86_64 {:&.rollIn}
* FileSystem: EXT4
* Mongodb: mongodb-linux-x86_64-rhel62-3.0.2 mongodb2.6.1

[slide]

## mongodb测试环境
----
### 硬件环境
* Server*3 虚拟机 {:&.rollIn}
* cpu: 1c
* Memory: 2G RAM
* Disk: 20GB

[slide]

## mongodb测试架构
----
* MongoDB replication architecture

[slide]

## mongodb测试架构
----
![replication](/img/replications.png "replication")
<pre><code>
<span class="blue">Server 1： mongod  primary</span><br>
<span class="blue">Server 2： mongod  secondary</span><br>
<span class="blue">Server 3： mongod  arbiter</span>
</code></pre>

[slide]

## mongodb测试数据集
----
<pre><code class="javascript">
{
        "_id" : "0",
        "date" : ISODate("2015-05-06T01:24:09.829Z"),
        "text" : "My first blog post!",
        "tags" : [
                "mongodb",
                "python",
                "pymongo"
        ],
        "author" : "0Mike"
}
</pre></code>

[slide]

## MongoDB副本集模式下不同引擎的磁盘占用情况
----
![replication disk](/img/replication-disk.png "replication disk")

----
* 在副本集模式下的, 插入100w数据的磁盘占比情况: Wiretiger占用 0.027GB, Mmap占用 0.453GB 使用新引擎wiretiger压缩明显

[slide]

## MongoDB副本集模式下不同引擎数据插入时间
----
![replication insert](/img/replication-insert.png "replication insert")

----
* 在副本集集群下的，单线程插入100w数据的不同引擎下的插入时间对比情况：使用Wiretiger花费时间1235.00s，使用MMAPv1花费时间1355.35s。使用WireTiger引擎在数据插入有一定优势。

[slide]

## MongoDB副本集模式下不同引擎随机查询时间
----
![replication query](/img/replication-query.png "replication query")

----
* 在副本集集群下，使用单线程随机读取100w数据，使用WireTiger引擎查询占用时间24.16s，使用MMAPv1引擎查询，占用时间24.18s。
由此可见，WireTiger引擎下查询的效率和MMAPv1引擎差不多。

[slide]

## MongoDB副本集模式下不同引擎删除100w数据时间
----
![replication delete](/img/replication-delete.png "replication delete")

----
* 在副本集集群模式下,使用单线程随机删除100w数据，发现使用wiretiger删除的时间要短。MMAP引擎，删除时间：75.56s，wiretiger引擎删除时间：65.07s


[slide]

## 不同版本，不同引擎，副本集模式下的mongodb测试

[slide]

## 磁盘占用情况
----
![replication disk](/img/multi-disk.png "replication disk")

----
* 上图展示了不同的mongodb版本，replica集群模式下，不同引擎下的，插入100w数据磁盘占用情况。可以看到版本不一样时，MMAPv1引擎下占用磁盘空间差不多，使用WireTiger引擎下，占用磁盘空间要小很多，这是新引擎下的一大优势。

[slide]

## 插入100w数据花费时间情况
----
![replication insert](/img/multi-insert.png "replication insert")

----
* replica集群模式下，不同引擎下的，单线程插入100w数据插入时间的占比情况。可以看出，同种MMAPv1引擎下，新版本插入性能要好些。同种版本下，WireTiger引擎插入的性能要好些。

[slide]

## 100w数据随机查询情况
----
![replication query](/img/multi-query.png "replication query")

----
* replica集群模式下，不同引擎下的，单线程随机查询100w数据查询时间的占比情况。可以看出，新老版本，不同引擎，查询性能差不多，没有明显提高。

[slide]

## 删除100w数据时间
----
![replication delete](/img/multi-delete.png "replication delete")

----
* replica集群模式下，不同引擎下的，单线程删除100w数据删除时间的占比情况。可以看出，同种MMAPv1引擎下，新版本删除性能要好些。同种版本下，WireTiger引擎删除的性能要好些。这个跟插入的性能呈现一样的趋势。

[slide]

## mognod shard + replication 架构
----
![shard](/img/shard1.png "shard")

[slide]

## mognod测试架构  shard + replication 
----
![shard-replica](/img/shard.png "shard-replica")
----

[slide]

## mognod测试数据集
----
<pre><code class="javascript">
{
        "_id" : "0",
        "date" : ISODate("2015-05-04T02:43:17.738Z"),
        "text" : "My first blog post!",
        "tags" : [
                "mongodb",
                "python",
                "pymongo"
        ],
        "author" : "0Mike"
}
</pre></code>

[slide]
## 对数据库做shard，数据打到不同shard
<pre><code>
shards:
 {"_id":"rs1","host":"rs1/mongodbtest1.55tuan.me:27021,
                          mongodbtest2.55tuan.me:27021,
                          mongodbtest3.55tuan.me:27021"}
 {"_id":"rs2","host":"rs2/mongodbtest1.55tuan.me:27022,
                          mongodbtest2.55tuan.me:27022,
                          mongodbtest3.55tuan.me:27022"}
</code>
</pre>

[slide]
## 分片后的数据
<pre><code>
databases:
  {  "_id" : "admin",  "partitioned" : false,  "primary" : "config" }
  {  "_id" : "wowotest",  "partitioned" : true,  "primary" : "rs1" }
    wowotest.userinfo
    shard key: { "_id" : 1 }
      chunks:
        rs1     7
        rs2     6
        { "_id" : { "$minKey" : 1 } } -->> { "_id" : "1" } on : rs2 Timestamp(2, 0) 
        { "_id" : "1" } -->> { "_id" : "153541" } on : rs2 Timestamp(3, 0) 
        { "_id" : "153541" } -->> { "_id" : "206708" } on : rs2 Timestamp(4, 0) 
        { "_id" : "206708" } -->> { "_id" : "259877" } on : rs2 Timestamp(5, 0) 
        { "_id" : "259877" } -->> { "_id" : "313041" } on : rs2 Timestamp(6, 0) 
        { "_id" : "313041" } -->> { "_id" : "40717" } on : rs1 Timestamp(7, 1) 
        { "_id" : "40717" } -->> { "_id" : "49437" } on : rs1 Timestamp(4, 4) 
        { "_id" : "49437" } -->> { "_id" : "547536" } on : rs1 Timestamp(5, 2) 
        { "_id" : "547536" } -->> { "_id" : "600702" } on : rs1 Timestamp(5, 3) 
        { "_id" : "600702" } -->> { "_id" : "7" } on : rs1 Timestamp(5, 4) 
        { "_id" : "7" } -->> { "_id" : "806331" } on : rs1 Timestamp(6, 2) 
        { "_id" : "806331" } -->> { "_id" : "99999" } on : rs1 Timestamp(6, 3) 
        { "_id" : "99999" } -->> { "_id":{ "$maxKey":1 } } on :rs2 Timestamp(7, 0)

</code>
</pre>

[slide]

## MongoDB副本集分片模式下不同引擎的磁盘占用情况
----
![replication disk](/img/shard-disk.png "replication disk")

----
* 在副本集分片模式下的，插入100w数据的磁盘占比情况：从图中可以看出，Wiretiger占用 0.027GB， Mmap占用0.656GB，磁盘占用情况跟没分片的差不多。

[slide]

## MongoDB副本集分片模式下插入100w数据时间情况
----
![replication insert](/img/shard-insert.png "replication insert")

----
* 在副本集分片模式下的，插入100w数据的不同引擎下的插入时间对比情况：
从图中可以看出，wiretiger引擎下，插入效率要高。Wiretiger花费时间2129.93s，MMAPv1花费时间2511.93s。


[slide]

## MongoDB副本集分片模式下随机查询100w数据
----
![replication query](/img/shard-query.png "replication query")
----
* 在副本集分片模式下，使用单线程随机读取100w数据，使用WireTiger引擎查询占用时间24.68s，使用MMAPv1引擎查询，占用时间24.80s。
由此可见，WireTiger引擎下查询的效率比MMAPv1引擎提高不是很明显。


[slide]

## MongoDB副本集分片模式下随机删除100w数据
----
![replication delete](/img/shard-delete.png "replication delete")
----
* 在副本集分片模式下,使用单线程随机删除100w数据，发现使用mmap删除的时间要短，跟理论有差距。MMAP引擎，删除时间：130.17s，wiretiger引擎删除时间：145.53s。


[slide]

## MongoDB多线程下测试
----
![replication delete](/img/shard-delete.png "replication delete")
----
* 在副本集分片模式下,使用单线程随机删除100w数据，发现使用mmap删除的时间要短，跟理论有差距。MMAP引擎，删除时间：130.17s，wiretiger引擎删除时间：145.53s。


[slide]

## 基于yahoo的ycsb测试
### mongodb 2.6.1 MMAPv1引擎 副本集集群 读写比例95:5 
----
![replication mmap](/img/mongo-261-mmap-95.jpg "replication mmap")
----
* case：使用MMAPv1引擎，副本级集群。测试100w数据在查询和插入比例为95:5的情况下的查询，插入性能。查询和插入使用同一台mongodb {:&.rollIn}
* collections为单一id索引

[slide]

### mongodb 2.6.1 MMAPv1引擎 副本集集群 读写比例95:5 
----
![replication mmap](/img/mongo-261-mmap-95.jpg "replication mmap")
----
* 使用yahoo的ycsb测试程序对数据库做大约95%查询和5%的插入同时处理。测试不同线程下的查询和插入平均延迟情况。 {:&.bounceIn}
**说明：中间20个进程出现插入很高的情况可能是当时服务器IO延迟大。**
* 从中可以看出：
  * 随着线程的增加，查询和插入的延迟时间也相应增加。 {:&.rollIn}
  * 随着线程的增加，lockdb有出现100%情况（少量，使用mongodbstat查看，上面图表没有反应），qr和qw出现次数增多，ar和aw次数也相应增多）

[slide]

### mongodb 2.6.1 MMAPv1引擎 副本集集群 读写比例90:10 
----
![replication mmap](/img/mongo-261-mmapv1-90.jpg "replication mmap")
----
* Case：使用MMAPv1引擎，副本级集群。测试100W数据在查询和插入比例为90:10的情况下的查询和插入性能。 {:&.rollIn}
* Collections使用id做索引
* 使用yahoo的ycsb测试。
* 从中可以看出：
  * 随着线程增加，查询和插入性能的延迟也相应增加。

[slide]
####mongodb 2.6.1 **MMAPv1引擎** 副本集集群 读写比例90:10/95:5 对比图
----
![replication mmap](/img/mongo-261-mmap-90-95.jpg "replication mmap")
----
* Case：Mongodb2.6.1 MMAPv1引擎 副本集模式。测试100w数据在查询和插入比例90:10和比例为95:5的情况下的对比。 {:&.bounceIn}
* 从图中可以看出:
  * 查询和插入出现不规律性，跟测试的时候宿主的cpu，磁盘使用情况有关，总的可以看出，比例为95:5的插入和读取情况要比比例为90:10的时间要小。

[slide]
####mongodb 3.0.2 **MMAPv1引擎** 副本集集群 读写比例95:5
----
![replication mmap](/img/mongo-302-mmap-95.jpg "replication mmap")
----

* Case：Mongodb3.0.2 MMAPv1引擎副本集模式，读写比例为95:5。测试100w数据在查询和插入比例为95:5比例情况下的对比.  {:&.rollIn}
* 从图中可以看出:
  * 随着线程数的增加，查询时的时间是越来越长的。插入时间在达到50线程后出现明显的提高。

[slide]
####mongodb 3.0.2 **Wiretiger引擎** 副本集集群 读写比例95:5
----
![replication wire](/img/mongo-302-wire-95.jpg "replication wire")
----

* Case：mongodb3.0.2 **wiretiger引擎** 副本集模式，读写比例95:5。测试100w数据在查询和插入的比例为95:5比例下的对比。 {:&.bounceIn}
* 从图中可以看出：
  * 读写时间也是随着线程的增加而相应增加的。线程大了之后，插入时间有明显的提高。

[slide]
####mongodb 3.0.2 **Wiretiger引擎** 和 **MMAPv1引擎** 对比
----
![replication wire](/img/mongo-302-wire-mmap-95.jpg "replication wire")
----

* Mongodb3.0.2版本，读写比例95:5，两种引擎下的对比. {:&.rollIn}
* 从图中可以看出:
  * 两种引擎下的读性能差别不是很大，线程大了后，wiretiger引擎表现的读时间少些，表现的性能要好些，但是提高的不明显。 {:&.rollIn}
  * 两种引擎下的写性能有较明显的差别，wiretiger引擎不论在线程数多少的情况下，都比mmapv1引擎要表现的好些，尤其是在线程数明显增大的情况下，写性能优势更明显。

[slide]
#### **mongodb2.6.1** 和 **mongodb3.0.2** 在 **MMAPv1引擎** 副本集集群 读写比例95:5 对比
----
![replication wire](/img/mongo-261-302-95.jpg "replication wire")
----
* mongodb2.6.1，mongodb3.0.2 版本在MMAPv1引擎下，读写比例为95:5的性能测试对比. {:&.rollIn}
* 从图中可以看出:
  * 同在MMAPv1引擎下，高版本的mongodb写性能差别大些，写性能差别较小，但是都比低版本的mongodb性能要好些。


[slide]
#### **mongodb2.6.1** 和 **mongodb3.0.2** 在 **wiretiger引擎** 副本集集群 读写比例95:5 对比
----
![replication wire](/img/mongo-261-mmap-302-wire-95.jpg "replication wire")
----
* mongodb2.6.1，MMAPv1引擎与mongodb3.0.2 wiretiger引擎在读写比例为95:5下的性能测试对比. {:&.rollIn}
* 从图中可以看出:
  * 新版本的mongodb在wiretiger引擎下的写性能优势比低版本的MMAPv1引擎高很多。 {:&.rollIn}
  * 读性能在线程数增加时表现的优势也越来越明显些。

[slide]
#### **mongodb2.6.1** 和 **mongodb3.0.2** 吞吐率比较
----
![replication wire](/img/throughout.png "replication wire")
----
* 副本集模式下，两个版本引擎在95％读取和5%的更新场景。显示wiretiger引擎有差不多两倍的吞吐量，这个比官方测试的4倍多的吞吐量有差别。 {:&.rollIn}
* 在MongoDB2.6中，并发控制是在数据库级别，而且写会阻塞读操作，从而降低总体的并发量。通过这次测试，我们看到MongoDB 3.0更加细粒的并发性控制明显提高了总并发量。

[slide]
## **结论**
* 启用wiretiger引擎后，磁盘占用大概比mmap引擎减少了75%-85% {:&.bounceIn}
* 副本集模式下，新版本mmap/wiretiger引擎的插入和删除要好。
* 副本集模式下，新版本的查询效率差不多（测试的时候没有测试secondary read模式）
* 多线程模式下，在查询插入比例为95:5时，新版本表现性能的比老版本好。使用wiretiger引擎插入性能有明显的提高。
* Shard+replica模式（提前建索引或者没有提前建索引）下，插入和删除效率没有replica模式下好。
## **建议**  {:&.rollIn}
* 在出现很大性能问题时，可以先把测试的版本升级到3.0.2后，看运行情况，再升级到wiretiger引擎。 
## **注意**  {:&.rollIn}
* 本测试没有覆盖数据的备份，恢复。在shard+replica模式下插入遇到两次连接中断情况。

[slide style="background-image:url('/img/blackops.jpg');background-repeat:no-repeat;background-position: center center;"]
# Q&A
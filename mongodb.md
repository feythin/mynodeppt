title: mongodb测试
speaker: 刘飞宇(feythin.lau)
url: http://www.liufeiyu.cn 
transition: horizontal3d
files: /js/demo.js,/css/moon.css,/js/zoom.js
theme: dark

[slide style="background-image:url('/img/blackops.jpg')"]

# mongodb性能测试
<small>Created by <a href="http://liufeiyu.cn">Feythin</a></small><br>
<small>Weibo: <a href="http://weibo.com/539167012">@Feythin</a></small><br>
<small>Blog: <a href="http://liufeiyu.cn">http://liufeiyu.cn</a></small>

[slide data-transition="horizontal3d"]
## mongodb测试简介
----
* mongodb测试环境 {:&.moveIn}
* mongodb测试架构 
* mongodb测试点 
* mongodb测试数据集 
* mongodb测试结果 

[slide]

## mognod测试环境
----
### 软件环境
* OS: CentOS release 6.5  kern：2.6.32-431.3.1.el6.x86_64 {:&.rollIn}
* FileSystem: EXT4
* Mongodb: mongodb-linux-x86_64-rhel62-3.0.2 mongodb2.6.1

[slide]

## mognod测试环境
----
### 硬件环境
* Server*3 虚拟机 {:&.rollIn}
* cpu: 1c
* Memory: 2G RAM
* Disk: 20GB

[slide]

## mognod测试架构
----
* MongoDB replication architecture

[slide]

## mognod测试架构
----
![replication](/img/replications.png "replication")
<pre><code>
<span class="blue">Server 1： mongod  primary</span><br>
<span class="blue">Server 2： mongod  secondary</span><br>
<span class="blue">Server 3： mongod  arbiter</span>
</code></pre>

[slide]

## mognod测试数据集
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
  {  "_id" : "hawaii",  "partitioned" : true,  "primary" : "rs1" }
    hawaii.userinfo
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

## 基本语法指南 {:&.flexbox.vleft}
----

<pre><code class="markdown">/* 先写总的配置 */
title: 这是title，网页名称
speaker: 演讲者名称
url: https://github.com/ksky521/nodePPT
transition: 全局转场动画
files: 引入的js和css文件，多个以半角逗号隔开
hightStyle: 代码高亮样式，默认monokai_sublime

/* 以&#91;slide&#93; 隔开每个ppt页面 */
&#91;slide&#93;
## 二级标题
这里写内容即可

&#91;slide&#93;
...
</code>
</pre>



[slide style="background-image:url('/img/bg1.png')"]

# 支持单页添加背景图片 {:&.flexbox.vleft}
## 使用方法：&#91;slide style="background-image:url('/img/bg1.png')"&#93;

[slide]
## 支持.class/#id/自定义属性样式
----

```html
使用：.class{:.class}
使用：#id{:#id}
组合使用：{:.class.class2 width="200px"}
父元素样式使用&：{:&.class}
```

[slide]

## 主页面样式
### ----是上下分界线
----

nodeppt是基于nodejs写的支持 **Markdown!** 语法的网页PPT

nodeppt：https://github.com/ksky521/nodePPT

[slide]

[subslide]
## 这是一个列表
---
* 上下左右方向键翻页
    * 列表支持渐显动画 {:&.moveIn}
    * 支持多级列表
    * 这个动画是moveIn
* 完全基于markdown语法哦
============
## 这是一个数字类型列表，这是一个subslide页面
---
1. 数字列表 {:&.rollIn}
2. 数字列表
3. 数字列表，这是一个subslide页面
[/subslide]

[slide]
## 列表渐显动画：fadeIn
----
* 列表支持渐显动画哦 {:&.fadeIn}
    * 使用方法
    * markdown列表第一条加上：{:&.动画类型}
* 动画类型
    * fadeIn
    * rollIn
    * bounceIn
    * moveIn
    * zoomIn

[slide]
## 列表渐显动画：zoomIn
----
* 列表支持渐显动画哦 {:&.zoomIn}
* 动画类型
    * fadeIn
    * rollIn
    * bounceIn
    * moveIn
    * zoomIn

[slide]
## 列表渐显动画：bounceIn
----
* 列表支持渐显动画哦 {:&.bounceIn}
* 动画类型
    * fadeIn
    * rollIn
    * bounceIn
    * moveIn
    * zoomIn



[slide]
## 表格示例
### 市面上主要的css预处理器：Less\Sass\Stylus
---
| Less | Sass | Stylus
:-------|:------:|-------:|--------
环境 |js/nodejs | Ruby(这列右对齐) | nodejs(高亮) {:.highlight}
扩展名 | .less | .scss/.sass | .styl
特点 | 老牌，用户多，支持js解析 | 功能全，有成型框架，发展快 | 语法多样，小众
案例/框架 | [Bootstrap](http://getbootstrap.com/) | [Compass](http://beta.compass-style.org) [Bootstrap](http://getbootstrap.com/css/#sass) [Foundation](http://foundation.zurb.com/) [Bourbon](http://bourbon.io) [Base.Sass](https://github.com/jsw0528/base.sass) |


[slide]
## text
-----

<span class="text-danger">.text-danger</span> <span class="text-success">.text-sucess</span><span class="text-primary">.text-primary</span>

<span class="text-warning">.text-warning</span><span class="text-info">.text-info</span><span class="text-white">.text-white</span><span class="text-dark">.text-dark</span>


<span class="blue">.blue</span><span class="blue2">.blue2</span><span class="blue3">.blue3</span><span class="gray">.gray</span><span class="gray2">.gray2</span><span class="gray3">.gray3</span>

<span class="red">.red</span><span class="red2">.red2</span><span class="red3">.red3</span>

<span class="yellow">.yellow</span><span class="yellow2">.yellow2</span><span class="yellow3">.yellow3</span><span class="green">.green</span><span class="green2">.green2</span><span class="green3">.green3</span>

[slide]
## label and link
<span class="label label-primary">.label.label-primary</span><span class="label label-warning">.label.label-warning</span><span class="label label-danger">.label.label-danger</span><span class="label label-default">.label.label-default</span><span class="label label-success">.label.label-success</span><span class="label label-info">.label.label-info</span>

<a href="#">link style</a> <mark>mark</mark>

[slide]
## blockquote
----

> nodeppt可能是迄今为止最好用的web presentation <small>三水清</small>


下面是另外一种样式

> 这是一个class是：pull-right的blockquote <small>small一下</small> {:&.pull-right}

[slide]
## buttons
----

<button class="btn btn-default">.btn .btn-default</button>  <button class="btn btn-primary">.btn.btn-lg.btn-primary</button> <button class="btn btn-warning">.btn.btn-waring</button> <button class="btn btn-success">.btn.btn-success</button> <button class="btn btn-danger">.btn.btn-danger</button>



<button class="btn btn-lg btn-default">.btn.btn-lg.btn-default</button> <button class="btn btn-xs btn-success">.btn.btn-xs.btn-success</button> <button class="btn btn-sm btn-primary">.btn.btn-sm.btn-primary</button> <button class="btn btn-rounded btn-warning">.btn.btn-rounded.btn-waring</button>  <button class="btn btn-danger" disabled="disabled">disabled.btn.btn-danger</button>


<button class="btn btn-success"><i class="fa fa-share mr5"></i></button>

[slide]
## icons: Font Awesome
------

<i class="fa fa-apple"></i>
<i class="fa fa-android"></i>
<i class="fa fa-github"></i>
<i class="fa fa-google"></i>
<i class="fa fa-linux"></i>
<i class="fa fa-css3"></i>
<i class="fa fa-html5"></i>
<i class="fa fa-usd"></i>
<i class="fa fa-pie-chart"></i>
<i class="fa fa-file-video-o"></i>
<i class="fa fa-cog"></i>


[slide]

## 代码格式化
### 使用 `highlightjs` 进行语法高亮
----
<div class="columns-2">
    <pre><code class="javascript">(function(window, document){
    var a = 1;
    var test = function(){
        var b = 1;
        alert(b);
    };
    //泛数组转换为数组
    function toArray(arrayLike) {
        return [].slice.call(arrayLike);
    }
}(window, document));
    </code></pre>
    <pre><code class="javascript">(function(window, document){
    var a = 1;
    var test = function(){
        var b = 1;
        alert(b);
    };
    //泛数组转换为数组
    function toArray(arrayLike) {
        return [].slice.call(arrayLike);
    }
}(window, document));
    </code></pre>
</div>



[slide]
## 支持多种皮肤
----

[colors](/)-[moon](?theme=moon)-[blue](?theme=blue)-[dark](?theme=dark)-[green](?theme=green)-[light](?theme=light)


[slide data-incallback="testScriptTag"]
## 支持 HTML 和 markdown 语法混编
----

<div class="file-setting">
    <p>这是html</p>
</div>
<p id="css-demo">这是css样式</p>
<p>将html代码直接混编到**markdown**文件中即可</p>

我是js控制的颜色 black {:#testScriptTag}

<script>
    function testScriptTag(){
        document.getElementById('testScriptTag').style.color = 'black';
    }

</script>
<style>
#css-demo{
    color: red;
}
</style>


[slide]
## iframe效果
----
<iframe data-src="http://www.baidu.com" src="about:blank;"></iframe>

[slide]
## 动画样式强调
----

这段话里面的**加粗**和*em*字体会动画哦~

按下【H】键查看效果


[slide]
## 支持zoom.js
----

增加了zoom.js的支持，在演示过程中使用`alt`+鼠标点击，则点击的地方就开始放大，再次`alt+click`则回复原装

[slide]

## 图片，点击全屏
----

![小萝莉](/girl.jpg "小萝莉")


[slide]
[note]
##这里是note

使用n键，才能显示
[/note]
## 使用note笔记
### note笔记是多窗口，或者自己做一些笔记用的
---
按下键盘【N】键测试下note，

markdown语法如下：
```markdown
[note]
这里是note，{ 要换成中括号啊！！
{/note]
```
[slide]

## 使用画笔
### 使用画笔做标记哦~你也可以随便作画啊！
---
按下键盘【P】键。按下鼠标左键，在此处乱花下看看效果。

按下键盘【C】键。清空画板

[slide]

## 宽度不够？？
---
按下键盘【W】键，切换到更宽的页面看效果，第二次按键返回

 |less| sass | stylus
:-------|:------:|-------:|--------
环境 |js/nodejs | Ruby(这列右对齐) | nodejs(高亮) {:.highlight}
扩展名 | .less | .sass/.scss | .styl
特点 | 老牌，用户多，支持js解析 | 功能全，有成型框架，发展快 | 语法多样，小众
案例/框架 | [Bootstrap](http://getbootstrap.com/) | [compass](http://compass-style.org) [bourbon](http://bourbon.io) |

[slide]
## 使用overview模式
---
按下键盘【O】键。看下效果。

在overview模式下，方向键下一页，【enter】键进入选中页

或者按下键盘【O】键，退出overview模式

[slide]

## 多窗口演示
## 双屏演示不out！
---
本页面网址改成 [url?_multiscreen=1](?_multiscreen=1)，支持多屏演示哦！

跟powderpoint一样的双屏功能，带有备注信息。

[slide]
## 20种转场动画随心换
----
 * <a href="?transition=slide">slide</a>/<a href="?transition=slide2">slide2</a>/<a href="?transition=slide3">slide3</a>
 * [newspaper](?transition=newspaper)
 * [glue](?transition=glue)
 * [kontext](?transition=kontext)/[vkontext](?transition=vkontext)
 * [move](?transition=move)/[circle](?transition=circle)
 * [horizontal](?transition=horizontal)/[horizontal3d](?transition=horizontal3d)
 * [vertical3d](?transition=vertical3d)
 * [zoomin](?transition=zoomin)/[zoomout](?transition=zoomout)
 * [cards](?transition=cards)
 * [earthquake](?transition=earthquake)/[pulse](?transition=pulse)/[stick](?transition=stick)...


[slide data-transition="glue"]

## 这是一个glue的动画
----
使用方法（全局设置） 1：

> transition: glue


[slide data-transition="glue"]

## 这是一个glue的动画
----
使用方法 2：

&#91;slide data-transition="glue"&#93;

[slide data-transition="zoomin"]

## 这是一个zoomin的动画
----
使用方法：

&#91;slide data-transition="zoomin"&#93;

[slide data-transition="vertical3d"]

## 这是一个vertical3d的动画
----
使用方法：

&#91;slide data-transition="vertical3d"&#93;

[slide data-outcallback="outcallback" data-incallback="incallback" ]
## 使用回调
----

* &#91;slide data-outcallback="fnName"&#93;
    * 进入执行回调incallback函数
* &#91;slide data-incallback="fnName"&#93;
    * 退出执行outcallback函数

亦可以组合写：

> &#91;slide data-outcallback="foo" data-incallback="bar"&#93;


<p id="incallback"></p>
<p id="outcallback"></p>

[slide]
## 远程执行函数
----
在多屏和远程模式中，可以使用`proxyFn`来远程执行函数。

```html
<script>
function globalFunc(){
}
</script>
<button onclick="Slide.proxyFn('globalFunc')" class="btn btn-default">远程执行函数</button>
```

<button onclick="Slide.proxyFn('globalFunc','args')" class="btn btn-default">测试远程执行函数</button>
<a href="?_multiscreen=1#33">在多屏中测试远程执行</a>
<script>
    function globalFunc(a){
        alert('proxyFn success: '+a);
    }
</script>


[slide]

## 更多玩法
---
https://github.com/ksky521/nodePPT

什么？这些功能还不够用？

socket远程控制可以在手机上摇一摇换页哦~

查看项目目录ppts获取更多帮助信息

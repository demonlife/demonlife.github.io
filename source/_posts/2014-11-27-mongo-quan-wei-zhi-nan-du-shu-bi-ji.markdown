---
layout: post
title: "Mongo 权威指南读书笔记"
date: 2014-11-27 15:03:05 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 基本概念</a></li>
<li><a href="#sec-2">2 基础知识</a>
<ul>
<li><a href="#sec-2-1">2.1 文档</a></li>
<li><a href="#sec-2-2">2.2 集合</a>
<ul>
<li><a href="#sec-2-2-1">2.2.1 命名</a></li>
<li><a href="#sec-2-2-2">2.2.2 子集合</a></li>
</ul>
</li>
<li><a href="#sec-2-3">2.3 数据库</a></li>
<li><a href="#sec-2-4">2.4 启动mongodb</a>
<ul>
<li><a href="#sec-2-4-1">2.4.1 mongodb的版本管理</a></li>
<li><a href="#sec-2-4-2">2.4.2 启动</a></li>
</ul>
</li>
<li><a href="#sec-2-5">2.5 mongodb shell</a></li>
<li><a href="#sec-2-6">2.6 mongodb 客户端命令</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">基本概念</h2>
<div class="outline-text-2" id="text-1">

<ul>
<li>纵向扩展
    使用计算能力更强的机器
</li>
<li>横向扩展
    通过分区将数据分散到更多的机器上
</li>
</ul>

</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">基础知识</h2>
<div class="outline-text-2" id="text-2">


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">文档</h3>
<div class="outline-text-3" id="text-2-1">

<p>   文档是键值对的一个有序集合，非常类似于关系型数据库中的行。
   每一个文档都有一个特殊的键"<sub>id</sub>", 这个键在文档所属的集合中是唯一的。
   文档的键是字符串，并且区分大小写，除少数例外的情况，键可以使用任意UTF-8字符，例外情况如下：
</p>


<pre class="example">1. 键不能含有\0(空字符)，该值用于表示键的结尾
2. .(点)和$都有特殊意义，只能在特定环境中使用
</pre>

<p>
   文档中不能有重复的键，并且键/值对是有序的，通常字段顺序不重要，无须让数据库模式依赖特定的
   字段顺序，但某些情况下，字段顺序变得很重要。
</p></div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">集合</h3>
<div class="outline-text-3" id="text-2-2">

<p>   可以看做是一个拥有动态模式的表
   由一组文档组成，相当于关系数据库中的一张表
</p>
</div>

<div id="outline-container-2-2-1" class="outline-4">
<h4 id="sec-2-2-1">命名</h4>
<div class="outline-text-4" id="text-2-2-1">

<p>    集合名可以是满足以下条件的任意UTF-8字符串:
</p>


<pre class="example">集合名不能是空字符串("")
集合名不能包含\0字符(空字符), 该字符表示集合名的结束
集合名不能以"system."开头，这是为系统集合保留的前缀。
用户创建的集合不能在集合名中包含保留字符"$"。因为某些系统生成的集合中包含$, 很多驱动程序确实
支持在集合名里包含该字符。除非你要访问这种系统创建的集合，否则不应该在集合中包含$.
</pre>

</div>

</div>

<div id="outline-container-2-2-2" class="outline-4">
<h4 id="sec-2-2-2">子集合</h4>
<div class="outline-text-4" id="text-2-2-2">

<p>    使用"."分割不同命名空间的子集和。如： 一个博客功能可能包含两个集合，分别是blog.posts, blog.authors
    这是为了使组织结构更清晰，这里的blog集合(这个集合甚至不需要存在)跟它的子集合没有任何关系。
    在mongodb中，使用子集合来组织数据非常高效，值得推荐。
</p></div>
</div>

</div>

<div id="outline-container-2-3" class="outline-3">
<h3 id="sec-2-3">数据库</h3>
<div class="outline-text-3" id="text-2-3">

<p>   多个集合就可以组成数据库。每个数据库都有独立的权限，不同的数据库放置在不同的文件中。
   数据库名有如下的限制:
</p>


<pre class="example">不能是空字符串("")
不得含有/,\ , . ", *, &lt;, &gt;, :, |, ?, $， 一个空格， \0(空字符)。基本上只能使用ASCII中的字母和数字
数据库名区分大小写，即便是在不区分大小写的文件系统中也是如此。简单起见，数据库名应全部小写
数据库名最多为64字节
</pre>

<p>
   数据库最终会变成文件系统里的文件， 数据库名就是相应的文件名。
   特殊的保留数据名有：
</p>


<pre class="example">admin: 从身份验证的角度将，这是"root"数据库。将某个用户添加到admin数据库中，该用户将自动获得所有
数据库的权限。某些特定的服务器端命令也只能从admin数据库运行。
local: 该数据库永远不可以复制，并且一台服务器上的所有本地集合都可以存储在这个数据库中
config: mongodb用于分片设置时，分片信息会存储在config数据库中。
</pre>

</div>

</div>

<div id="outline-container-2-4" class="outline-3">
<h3 id="sec-2-4">启动mongodb</h3>
<div class="outline-text-3" id="text-2-4">

<p>   通常mongodb作为网络服务器来运行，客户端可以连接到该服务器并执行操作。
</p>
</div>

<div id="outline-container-2-4-1" class="outline-4">
<h4 id="sec-2-4-1">mongodb的版本管理</h4>
<div class="outline-text-4" id="text-2-4-1">

<p>    mongodb的版本中，偶数号为稳定版本，奇数号为开发版。例如：2.4就是稳定版本，2.5就是开发版
    mongodb的漏洞追踪系统: <a href="https://jira.mongodb.org/secure/Dashboard.jspa">https://jira.mongodb.org/secure/Dashboard.jspa</a> 存在着核心服务器路线
    图，查看该路线图，可以得知下一个稳定版本的发布时间。
</p></div>

</div>

<div id="outline-container-2-4-2" class="outline-4">
<h4 id="sec-2-4-2">启动</h4>
<div class="outline-text-4" id="text-2-4-2">

<p>    必须创建一个目录以便数据库写入文件。数据库默认使用/data/db目录，也可以指定其他目录。
    如果建立了默认目录，需要确保有正确的写权限。
</p><ul>
<li>启动mongodb
      &lt;mongodb-install-path&gt;/mongod &ndash;dbpath /data/path， 启动mongodb，并指定默认的数据存储目录
      有关常用的选项，可以运行 mongod &ndash;help来查看
</li>
</ul>

<p>    启动时，服务器会打印版本和系统信息，然后等待连接。默认情况下mongodb监听27017端口。
    如果需要使用web rest接口，需要加上参数&ndash;rest, 2.6.1版本中可以使用&ndash;httpinterface
</p>
</div>
</div>

</div>

<div id="outline-container-2-5" class="outline-3">
<h3 id="sec-2-5">mongodb shell</h3>
<div class="outline-text-3" id="text-2-5">

<p>   mongodb自带javascript shell，可以在shell中使用命令行与mongodb实例交流
   mongo shell是一个完备的javascript解释器， 可以运行任意javascript程序。也可以充分利用javascript的标准库
   也可以定义和调用javascript函数。
   shell会检测输入的javascript语句是否完整， 在某行连续3次按下回车键可取消未输入完成的命令，并退出回到&gt;-提示符
   shell还包含一些非javascript语法的扩展。
</p></div>

</div>

<div id="outline-container-2-6" class="outline-3">
<h3 id="sec-2-6">mongodb 客户端命令</h3>
<div class="outline-text-3" id="text-2-6">

<p>   db: 查看当前指向哪个数据库
   use dbname: 切换到数据库dbname
{% codeblock lang:javascript %}
   post = {"title":"my blog", "content":"my content"} //定义一个合法的文档
   db.blog.insert(post) // 插入到集合中
   db.blog.find() // 查找插入的内容， 返回的结果中，会发现有一个额外的"_id"键， 
   //shell会自动显示最多20个匹配文档，可以获取更多
   db.blog.findOne() // 查看一条结果
   // find(), findOne()可以接受一个查询文档作为限定条件。
{% endcodeblock %}
</p>
</div>
</div>
</div>

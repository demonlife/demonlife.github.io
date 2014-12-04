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
<li><a href="#sec-2-5">2.5 mongodb shell</a>
<ul>
<li><a href="#sec-2-5-1">2.5.1 使用脚本将变量注入到shell</a></li>
</ul>
</li>
<li><a href="#sec-2-6">2.6 mongodb 客户端命令</a></li>
<li><a href="#sec-2-7">2.7 数据类型</a></li>
<li><a href="#sec-2-8">2.8 创建.mongorc.js文件</a>
<ul>
<li><a href="#sec-2-8-1">2.8.1 定制shell提示</a></li>
<li><a href="#sec-2-8-2">2.8.2 编辑符合变量</a></li>
</ul></li>
</ul>
</li>
<li><a href="#sec-3">3 创建/更新/删除文档</a>
<ul>
<li><a href="#sec-3-1">3.1 插入</a></li>
<li><a href="#sec-3-2">3.2 删除文档</a></li>
<li><a href="#sec-3-3">3.3 更新</a>
<ul>
<li><a href="#sec-3-3-1">3.3.1 使用修改器</a></li>
<li><a href="#sec-3-3-2">3.3.2 findAndModify</a></li>
</ul>
</li>
<li><a href="#sec-3-4">3.4 写入安全</a></li>
</ul>
</li>
<li><a href="#sec-4">4 查询</a>
<ul>
<li><a href="#sec-4-1">4.1 查询条件</a></li>
<li><a href="#sec-4-2">4.2 or查询</a></li>
<li><a href="#sec-4-3">4.3 $not</a></li>
<li><a href="#sec-4-4">4.4 条件语义</a></li>
<li><a href="#sec-4-5">4.5 特定类型的查询</a></li>
<li><a href="#sec-4-6">4.6 正则表达式</a></li>
<li><a href="#sec-4-7">4.7 查询数组</a></li>
<li><a href="#sec-4-8">4.8 返回一个匹配的数组元素</a></li>
<li><a href="#sec-4-9">4.9 查询内嵌文档</a></li>
<li><a href="#sec-4-10">4.10 $where查询</a></li>
<li><a href="#sec-4-11">4.11 游标</a></li>
<li><a href="#sec-4-12">4.12 高级查询选项</a>
<ul>
<li><a href="#sec-4-12-1">4.12.1 获取一致的结果</a></li>
<li><a href="#sec-4-12-2">4.12.2 游标生命周期</a></li>
</ul>
</li>
<li><a href="#sec-4-13">4.13 数据库命令</a>
<ul>
<li><a href="#sec-4-13-1">4.13.1 数据库命令工作原理</a></li>
</ul></li>
</ul>
</li>
<li><a href="#sec-5">5 索引</a>
<ul>
<li><a href="#sec-5-1">5.1 使用覆盖索引</a></li>
<li><a href="#sec-5-2">5.2 隐式索引</a></li>
<li><a href="#sec-5-3">5.3 索引的使用</a></li>
<li><a href="#sec-5-4">5.4 索引对象和数组(P96)</a>
<ul>
<li><a href="#sec-5-4-1">5.4.1 索引数组</a></li>
</ul>
</li>
<li><a href="#sec-5-5">5.5 explain 命令返回值的解释</a></li>
<li><a href="#sec-5-6">5.6 索引类型</a></li>
</ul>
</li>
<li><a href="#sec-6">6 特殊的索引和集合</a>
<ul>
<li><a href="#sec-6-1">6.1 固定集合</a></li>
<li><a href="#sec-6-2">6.2 TTL 索引</a></li>
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

<p>
    当某些集合名是mongo shell中的函数时，此时可以使用db.getCollection("name")来访问相应的集合
    如果集合名中包含无效的javascript属性名称，也可以使用这个函数。
    也可以使用数组访问方法来访问无效属性名称的集合。例如:
{% codeblock lang:javascript %}
    var collections = ["posts", "comments", "authors"];
    for (var i in collections) {
        print(db.blog[collections[i]]);
    }
{% endcodeblock %}
    使用数组方法访问怪异的集合:
{% codeblock lang:javascript %}
    var name = "@#&!";
    db[name].find();
{% endcodeblock %}
</p></div>

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
</p></div>
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
   mongo： 会自动连接到本地默认端口上的mongodb
   mongo some-host:port/dbname: 连接到指定host:port上的dbname数据库
   mongo &ndash;nodb: 启动时不连接任何数据库，之后使用如下代码可以连接到指定的数据库
{% codeblock lang:javascript %}
   conn = new Mongo("host:port")
   db = conn.getDB("dbname")
{% endcodeblock %}
   执行完上述命令后，就可以像正常使用db了，任何时候都可以使用这些命令来连接到不同的数据库或者服务器上。
   在shell中输入help可以查看帮助文档
   db.help()： 查看数据库级别的帮助文档
   db.foo.help(): 查看集合级别的帮助文档
   如果想知道一个函数的作用，可以直接输入函数名(函数名不用带())就可以查看该函数的实现代码。
   mongo file.js: 执行js文件
   mongo &ndash;quiet host:port/dbname file1.js file2.js: 执行指定主机端口上的mongod运行脚本
</p>


<pre class="example">可以在脚本中使用print()函数，将内容输出到标准输出，这样就可以在shell中使用管道命令。
如果将shell脚本的输出管道给另一个使用--quiet选项的命令，就可以让shell不打印"MongoDB shell version..."提示
</pre>

<p>
   也可以使用load()函数，从交互式shell中运行脚本，load("file.js")
   在脚本中可以访问db变量，但是shell的辅助函数(如use db, show dbs)不可以在文件中使用。这些辅助函数有对应的
   js函数，如下所示：
</p>


<pre class="example">use foo =&gt; db.getSisterDB("foo")
show dbs =&gt; db.getMongo().getDBs()
show collections =&gt; db.getCollectionNames()
</pre>

<p>
   在服务器运行mongod时，通过指定&ndash;noscripting可以关闭js的执行
   在shell中关闭mongodb：
</p>


<pre class="example">use admin
db.shutdownServer()
</pre>


</div>

<div id="outline-container-2-5-1" class="outline-4">
<h4 id="sec-2-5-1">使用脚本将变量注入到shell</h4>
<div class="outline-text-4" id="text-2-5-1">


{% codeblock lang:javascript %}
    // 连接到指定的数据库, 保存到文件defineConnectTo.js
    var connectTo = function(port, dbname) {
        if (!port) {
            port = 27017;
        }
        if (!dbname) {
            dbname = "test";
        }
        db = connect("localhost:"+port+"/"+dbname);
        return db;
    };
    
{% endcodeblock %}
<p>
    之后就可以在mongo shell中使用如下语句加载函数
    load('defineConnectTo.js')
    shell会在运行是所处的目录中查找脚本，可以使用run('pwd')命令查看启动mongo shell的路径
    load()也可以传递一个绝对路径的js文件，但是无法解析'~'符号
    run()可以执行命令行程序，例如:
</p>


<pre class="example">run("ls", "-l", "/home/demon")
使用这种方式局限性很大，输出格式也很奇怪，且不支持管道。
</pre>

</div>
</div>

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

   post.comments = []
   // update接受至少两个参数，第一个是限定条件(用于匹配待更新的文档), 第二个是新的文档。
   db.blog.update({"title":"my blog"}, post)

   // remove方法可将文档从数据库中永久删除
   db.blog.remove({"title": "my blog"})
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-2-7" class="outline-3">
<h3 id="sec-2-7">数据类型</h3>
<div class="outline-text-3" id="text-2-7">

<ul>
<li>null
     用于表示空值或者不存在的字段
</li>
<li>布尔型
     两个取值： true/false
</li>
<li>数值
     shell默认使用64位浮点型数值
     对于整型值，可以使用NumberInt类(4字节带符号整数)/NumberLong类(8字节带符号整数)
</li>
<li>字符串
     UTF-8字符串都可以表示为字符串类型的数据
</li>
<li>日期
     日期被存储为自新纪元以来经过的毫秒数，不存储时区， {"x": new Date()}
     创建日期对象时，应该使用new Date(&hellip;),而非Date(&hellip;),
     原因是：Date()返回的是日期的字符串表示，而非日期对象， 可以使用typeof(&hellip;)查看数据类型
</li>
<li>正则表达式
     查询时， 使用正则表达式作为限定条件，语法与js的正则表达式语法相同
</li>
<li>数组
     {"x": ["a", "b", "c"]}
     数组可以包含不同数据类型的元素
     mongodb能"理解"其结构， 因此能使用数组内容对数组进行查询和构建索引了。
     mongodb可以使用原子更新对数组内容进行修改。
</li>
<li>内嵌文档
     {"x": {"foo":"bar"}}
     mongodb也能对内嵌文档建立索引。
</li>
<li>对象id
     是一个12个字节的ID，是文档的唯一标示。是一个由24个16进制数字组成的字符串，这个长长的
     ObjectId是实际存储数据的两倍长
     ObjectId的12字节按照如下方式生成:
     0-3：时间戳
     4-6：机器
     7-8：PID(进程标识符)
     9-11：计数器
     绝大多数驱动程序会提供一个方法，用于从ObjectId获取这些信息
     一秒中最多允许每个进程拥有2563(16777216)个不同的ObjectId
</li>
<li>二进制数据
     不能直接在shell中使用，如果要将非UTF-8字符保存到数据库中，二进制数据是唯一的方式。
</li>
<li>代码
     查询和文档中可以包括任意js代码
</li>
</ul>


</div>

</div>

<div id="outline-container-2-8" class="outline-3">
<h3 id="sec-2-8">创建.mongorc.js文件</h3>
<div class="outline-text-3" id="text-2-8">

<p>   频繁加载的命令/脚本可以添加到.mongorc.js文件中, 该文件会在启动shell时自动运行。
   例如，打印欢迎信息:
{% codeblock lang:javascript %}
   var info =["attractive", "intelligent", "like Batman"];
   var index = Math.floor(Math.random()*3)
   print("Hello, "+info[index]);
{% endcodeblock %}
   .mongorc.js最常见用途之一就是移除那些比较"危险"的辅助函数，例如:
{% codeblock lang:javascript %}
   var no = function() {
       print("Not on my watch.");
   }

   // 禁止删除数据库
   db.dropDatabase = DB.prototype.dropDatabase = no;
{% endcodeblock %}
   mongo &ndash;norc: 启动时禁止加载.mongorc.js
</p>
</div>

<div id="outline-container-2-8-1" class="outline-4">
<h4 id="sec-2-8-1">定制shell提示</h4>
<div class="outline-text-4" id="text-2-8-1">

<p>    设置prompt:
    prompt = function() {
        return ((new Date()))+"&gt; ";
    };
</p></div>

</div>

<div id="outline-container-2-8-2" class="outline-4">
<h4 id="sec-2-8-2">编辑符合变量</h4>
<div class="outline-text-4" id="text-2-8-2">

<p>    首先需要设置EDITOR环境变量， EDITOR="/editor/path"
    之后在mongo shell中可以使用如下命令编辑变量: edit var-name
    可以在.mongorc.js文件中添加一行EDITOR="/editor/path"即可
</p>
</div>
</div>
</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">创建/更新/删除文档</h2>
<div class="outline-text-2" id="text-3">


</div>

<div id="outline-container-3-1" class="outline-3">
<h3 id="sec-3-1">插入</h3>
<div class="outline-text-3" id="text-3-1">

<ul>
<li>批量插入
     在shell中可以使用batchInsert(), 接受一个文档数组作为参数。
     如果在执行批量插入的过程中有一个文档插入失败，那么在这个文档之前的所有
     文档都会成功插入，之后的文档不会插入。如果希望批量插入忽略错误，可以使用
     continueOnError选项， shell并不支持该选项。所有驱动程序都支持
</li>
<li>插入校验
     所有文档都必须小于16M(这个值是MongoDB设计者认为设定的, 未来可能会增大).
     Object.bsonsize(doc): 查看doc的大小。
</li>
</ul>

</div>

</div>

<div id="outline-container-3-2" class="outline-3">
<h3 id="sec-3-2">删除文档</h3>
<div class="outline-text-3" id="text-3-2">

<p>   db.foo.remove(): 删除foo集合中的所有信息，但不会删除集合本身，也不会
   删除集合的元信息。
   如果要清空整个集合，使用drop直接删除集合会更快。
</p></div>

</div>

<div id="outline-container-3-3" class="outline-3">
<h3 id="sec-3-3">更新</h3>
<div class="outline-text-3" id="text-3-3">

<p>   update()有两个参数，一个是查询文档，另一个是修改器。
   最好确保更新是总是指定一个唯一文档。
   如果要更新所有匹配的文档，可以将update的第4个参数设置为true，
   update的行为可能会发生变化，建议每次都显示指定是否要更新多个文档
   想要知道更新了多少个文档，可以使用如下命令:
</p>


<pre class="example">db.runCommand({getLastError:1}) // 输出的n就是被更新文档的数量
</pre>


</div>

<div id="outline-container-3-3-1" class="outline-4">
<h4 id="sec-3-3-1">使用修改器</h4>
<div class="outline-text-4" id="text-3-3-1">

<ul>
<li>$set 修改器
</li>
</ul>


{% codeblock lang:javascript %}
      db.col_name.update({"_id":"xxx"}, {"$set":{"key":"value"}});
      // $set还可以改变键的类型, 例如：
      db.col_name.update({"_id":"xxx"}, {"$set":{"key":["value", "value2"]}});
      // $unset可以删除整个键
      db.users.update({"name": "joe"}, {"$unset": {"key": 1}})
      // 可以使用set修改内嵌文档, 修改author内嵌的name属性
      db.blog.posts.update({"author":"joe"}, {"$set": {"author.name":"xxx"}})
{% endcodeblock %}
<ul>
<li>$inc
      用来增加已有键的值，或者该键不存在就创建一个。该方法只能用于整型，长整型或
      双精度浮点型的值。
</li>
<li>$push
      向已有的数组末尾加入一个元素， 要是没有就创建一个数组。
      使用$each子操作，可以一次添加很多个值:
      db.stock.ticker.update({"<sub>id</sub>":"xx"}, {"$push": {"hourly": {"$each":[5, 6, 7]}}});
</li>
<li>$slice
      如果希望数组的最大长度是固定的， 可以将"$slice"和"$push"组合在一起。
</li>
</ul>


{% codeblock lang:javascript %}
      // 限制数组只包含最后加入的10个元素
      db.movies.update({"genere":"h"}, {"$push":{"top10": {"$each":["n", "m"], "$slice":-10}}});
{% endcodeblock %}
<p>
      不能只将"$slice"或者"$sort"与"$push"配合使用，且必须使用"$each"，
{% codeblock lang:javascript %}
      // 对xxx进行排序
      db.movies.update({"genere":"h"}, {"$push":{"top10": {"$each":["n", "m"], "$slice":-10, "$sort":{"xxx":-1}}}});
{% endcodeblock %}
</p><ul>
<li>将数组作为数据集使用
      "$ne": 如果不在数组中，就添加进去
      db.papers.update({"authors cited":{"$ne": "Richie"}}, {$push: {"authors cited": "Richie"}});
      也可以使用$addToSet来实现，某些情况下addToSet更合适。
      db.users.update({"<sub>id</sub>":"xx"}, {"$addToSet":{"emails":"xx"}});
      可以将$addToSet与$each组合起来：
      db.users.update({"<sub>id</sub>":"xx"}, {"$addToSet": {"emails":{"$each":["x1", "x2"]}}});
      {$pop:{"key":1}}：从数组的末尾删除一个元素
      {$pop:{"key":-1}}: 从数组头部删除
      $pull： 会将所有匹配的文档删除，而不是只删除一个。
      db.list.update({}, {"$pull":{"todo":"value"}});
</li>
<li>基于位置的数组修改
      将comments的第一个元素的votest增加1
      db.blog.update({"<sub>id</sub>":"xx"}, {"\(inc":{"comments.0.votest":1}})
      mongodb提供了定位操作符"\)", 用来定位查询文档已经匹配的数组元素，并进行更新，例如：
</li>
</ul>


{% codeblock lang:javascript %}
      // blog的一个文档内容如下
      /*
        {"_id":xx,
         "content":"...",
         "comments": [
             {
                 "comment":"..",
                 "author":"john",
                 "votes":0
             },
             {
                 "comment":"..",
                 "author":"hhh",
                 "votes":0
             }
         ]}
      */
      db.blog.update({"comments.author":"John"}, {"$set":{"comments.$.author":"jim"}});
{% endcodeblock %}
<p>
      定位器只更新第一个匹配的元素，如果某人有多条评论，只会修改第一条评论中的数据
</p><ul>
<li>修改器的速度
      db.collname.stats()：可以查看collname集合的填充因子，paddingFactor就是填充因子。
      应该尽量让填充因子的值接近1.无法手动设定填充因子的值，除非要对集合进行压缩。
      如果你的模式在进行插入和删除时会进行大量的移动或者是经常打乱数据，可以使用usePowerOf2Sizes
      选项以提高磁盘复用率。可以使用collMod命令来设定这个选项。
      db.runCommand({"collMod":collectionName, "usePowerOf2Sizes":true}) //设为false就会关闭该机制。
      设定之后该集合之后进行的所有空间分配大小都是2的幂，应该只在需要经常打乱数据的集合上使用。
      在一个只进行插入或者原地更新数据的集合上使用该选项会导致速度变慢。
</li>
<li>upsert
      在使用update方法时，如果第三个参数设置为true，则会启用upsert功能，即当不存在时，就插入数据库
      db.users.update({"rep":25}, {"$inc":{"rep":3}}, true)
</li>
<li>$setOnInsert
      会在插入数据的时候设置字段的值
      db.users.update({}, {"$setOnInsert":{"createdAt": new Date()}}, true);
</li>
<li>save方法
      如果一个文档含有"<sub>id</sub>"见，save会调用upsert方法，否则调用insert方法
</li>
</ul>


</div>

</div>

<div id="outline-container-3-3-2" class="outline-4">
<h4 id="sec-3-3-2">findAndModify</h4>
<div class="outline-text-4" id="text-3-3-2">

<p>    用于处理有竞争的情况
    例如:
</p>


<pre class="example">ps = db.runCommand({"findAndModify":"collectname", 
                    "query":{"status":"READY"},
                    "sort": {"priority": -1},
                    "update": {"$set": {"status":"RUNNING"}}}).value
do_something(ps); // 运行ps后READY状态会变成RUNNING状态
db.process.update({"_id":ps._id}, {"$set":{"status":"DONE"}});
</pre>

<p>
    更多的用法，请google.
</p>
</div>
</div>

</div>

<div id="outline-container-3-4" class="outline-3">
<h3 id="sec-3-4">写入安全</h3>
<div class="outline-text-3" id="text-3-4">

<p>   两种最基本的写入安全机制是应答式写入和非应答式写入。
   应答式写入是默认的方式，数据库会给出响应，告诉你写入是否成功执行。
   非应答式写入不返回任何响应，无法知道写入是否成功。
</p>
</div>
</div>

</div>

<div id="outline-container-4" class="outline-2">
<h2 id="sec-4">查询</h2>
<div class="outline-text-2" id="text-4">

<p>  find(): 第一个参数是查询的条件(where条件)， 第二个参数是指定需要返回的键，其格式如下:
  db.user.find({}, {"username":1, "email":0})// username显示，email不显示
</p>
</div>

<div id="outline-container-4-1" class="outline-3">
<h3 id="sec-4-1">查询条件</h3>
<div class="outline-text-3" id="text-4-1">

<p>   db.users.find({"age":{"$gte":18, "$lte": 30}});
   可以使用的有:
   $lt(&lt;), $lte(&lt;=), $gt(&gt;), $gte(&gt;=), $ne(!=, 可以用于所有的类型的数据)
</p></div>

</div>

<div id="outline-container-4-2" class="outline-3">
<h3 id="sec-4-2">or查询</h3>
<div class="outline-text-3" id="text-4-2">

<p>   or查询有两种方式，一是: "$in", 另一种是"$or"
</p>


<pre class="example">db.user.find({"user_id": {"$in":[1, 2, 3]}})
db.user.find({"$or": [{"ticket_no":725}, {"winner":true}]})
db.user.find({"$or":[{"ticket_no": {"$in":[1, 2, 3]}}, {"winner":true}]})
</pre>

<p>
   与"$in"相对就是"$nin", 返回与数组中所有条件都不匹配的文档。
</p></div>

</div>

<div id="outline-container-4-3" class="outline-3">
<h3 id="sec-4-3">$not</h3>
<div class="outline-text-3" id="text-4-3">

<p>   "$not"是元条件句，可以用在任何其他条件之上。
   db.user.find({"id<sub>num</sub>":{"$not":{"$mod": [5, 1]}}})
   $mod会将查询的值初一第一个给定值，若余数等于第二个参数则匹配成功
   $not可以与正则表达式联合使用
</p></div>

</div>

<div id="outline-container-4-4" class="outline-3">
<h3 id="sec-4-4">条件语义</h3>
<div class="outline-text-3" id="text-4-4">

<p>   基本可以肯定的是: 条件语句是内层文档的键，而修改器是外层文档的键。
   一个键可以对应多个条件，但是不能对应多个更新修改器。
   有些”元操作符"也位于外层文档中，如:"$and", "$or", "$not"
   查询优化器不会对"$and"进行优化。
   db.usr.find({"$and":[{"x": {"$lt": 1}}, {"x":4}]}) //匹配{"x":[0, 4]}等价于
   db.user.find({"x": {"$lt":1, "$in":[4]}})// 效率更高
</p></div>

</div>

<div id="outline-container-4-5" class="outline-3">
<h3 id="sec-4-5">特定类型的查询</h3>
<div class="outline-text-3" id="text-4-5">

<p>   db.c.find({"y":null}); // 返回y键为null或者不含有y键的文档
   db.c.find({"y": {"$in": [null], "$exists": true}})// 返回有y键且y键为null
</p></div>

</div>

<div id="outline-container-4-6" class="outline-3">
<h3 id="sec-4-6">正则表达式</h3>
<div class="outline-text-3" id="text-4-6">

<p>   db.users.find({"name": <i>joe/i}) /</i> 查询joe，不区分大小写
   系统能识别正则表达式标志(i), mongodb使用perl兼容的正则表达式(PCRE)库来匹配正则表达式。
   mongodb可以为前缀型正则表达式如/<sup>joey</sup>/查询建立索引
   在查询时，正则表达式也可匹配自身。
</p></div>

</div>

<div id="outline-container-4-7" class="outline-3">
<h3 id="sec-4-7">查询数组</h3>
<div class="outline-text-3" id="text-4-7">

<p>   db.food.find({"fruit":"apple"})// 会查询出fruit中包含apple的数据
   $all:多个元素来匹配数组， db.food.find({"fruit":{"$all":["apple", "banana"]}})//同时包含
   可以使用整个数组进行精确匹配，精确匹配对于缺少元素或者元素冗余的情况就不大灵了。如:
</p>


<pre class="example">db.food.find({"fruit":["apple", "banana"]})，db.food.find({"fruit":["apple", "peach", "banana"]})
都不会匹配{"fruit":["apple", "banana", "peach"]}
</pre>

<p>
   查询数组特定位置的元素，需使用key.index语法指定，下标从0开始
   db.food.find({"fruit.2":"peach"})
   "$size": 查询特定长度的数组
   "$slice": 可以返回某个匹配键的数组元素的一个子集。
   db.blog.posts.findOne(criteria, {"comments":{"$slice":10}}); // 返回前10条， {"$slice":-10}, 返回后10条
   {"$slice": [23, 100]}: 指定偏移值以及返回的元素数量
</p></div>

</div>

<div id="outline-container-4-8" class="outline-3">
<h3 id="sec-4-8">返回一个匹配的数组元素</h3>
<div class="outline-text-3" id="text-4-8">

<p>   db.blog.posts.find({"comments.name":"bob"}, {"comments.$":1});// 只返回第一个匹配的文档
   如果某个文档的"x"字段是一个数组，此时如果"x"键的某一个值与查询条件任意一条语句向匹配，则该文档也会返回。
   因此对数组使用范围查询没有用。有几种方式可以得到预期的行为：
</p>


<pre class="example">db.test.find({"x": {"$elemMatch":{"$gt":10, "$lt":20}}})// 只会匹配数组，不会匹配非数组
// 如果当前查询的字段上创建过索引，可以使用min()/max()将查询条件遍历的索引范围限制为"$gt"/"$lt"的值
db.test.find({"x":{"$gt":10, "$lt":20}}).min({"x":10}).max({"x":20})
在可能包含数组的文档上应用范围查询时，使用min()/max()是很好的，如果在整个索引范围对数组使用$gt/$lt,效率会很低
</pre>

</div>

</div>

<div id="outline-container-4-9" class="outline-3">
<h3 id="sec-4-9">查询内嵌文档</h3>
<div class="outline-text-3" id="text-4-9">

<p>   用点表示法查询内嵌文档，这样的查询是与顺序无关的。
   一个复杂点的例子:
</p>


<pre class="example">文档内容如下:
{"content":"...",
 "comments":[{"author":"joe",
              "score":3,
              "comment":"nice post"},
             {"author":"mary",
              "score":6,
              "comment":"terrible post"}]}
查询由joe发布的5分以上的评论
db.blog.find({"comments":{"$elemMatch":{"author":"joe", "score":{"$gte":5}}}})
</pre>

</div>

</div>

<div id="outline-container-4-10" class="outline-3">
<h3 id="sec-4-10">$where查询</h3>
<div class="outline-text-3" id="text-4-10">

<p>   使用$where可以在查询中执行任意的js，应该严格限制或消除$where语句的使用。应该禁止终端用户使用任意的"$where"语句。
   在使用$where查询时，每个文档都要从BSON转换为JS对象，$where也不能使用索引。
   为了避免sql注入，可以使用作用域来传递name的值，
   func = pymongo.code.Code("function() {print('hello, '<del>username</del>'!');}", {"username":name});
</p></div>

</div>

<div id="outline-container-4-11" class="outline-3">
<h3 id="sec-4-11">游标</h3>
<div class="outline-text-3" id="text-4-11">

<p>   在shell中建立游标，只需要将查询结果赋值给某个变量即可，如: var cursor = db.blog.find();//构造查询
   cursor.hasNext(): 是否有后继结果存在， 查询发往服务器执行
   cursor.next(): 获得下一个结果
   可以在forEach循环中使用:
   cursor.forEach(function(x) {
       print(x.name);
   });
</p><ul>
<li>limit
     db.blog.find().limit(num): 返回num个结果
     db.blog.find().skip(num): 跳过前num个结果
     db.blog.find().sort({"key":1, "key2": -1}); // 排序，1：升序，-1：降序
     略过过多的结果会导致性能问题。
     处理方法之一是：可以利用第一页的数据直接查询数据库，
</li>
</ul>

</div>

</div>

<div id="outline-container-4-12" class="outline-3">
<h3 id="sec-4-12">高级查询选项</h3>
<div class="outline-text-3" id="text-4-12">

<p>   有一些选项可以用于对查询进行"封装",假如执行一个排序:
   var cursor = db.foo.find({"foo":"bar"}).sort({"x":1})。此时会将查询封装在一个更大的文档中，
   会转换成: {"$query":{"foo":"bar"}, "$orderby":{"x":1}}
   绝大多数驱动程序提供了辅助函数，用于向查询中添加各种选项。
   db.foo.find({"x":"xx"}).<sub>addSpecial</sub>("$maxscan", 20)//指定本次查询中扫描文档数量的上限
   $min, $max,
   db.foo.find({}).<sub>addSpecial</sub>("$showDiskLoc', true) //查询结果中添加一个"$diskLoc"字段，
   file号码显示了这个文档所在的文件，第二个字段显示的是该文档在文件中的偏移量。
</p>
</div>

<div id="outline-container-4-12-1" class="outline-4">
<h4 id="sec-4-12-1">获取一致的结果</h4>
<div class="outline-text-4" id="text-4-12-1">

<p>    数据处理通常的做法就是先把数据从Mongodb中取出来，然后做一些变换，最后在存回去，如果文档体积增加了，而预留空
    间不足，这时就需要对体积增大后的文档进行移动，通常会挪到末尾处。当游标移动到集合末尾时，就会返回因体积太大
    无法放回原位置而被移动到集合末尾的文档。应对该问题，就是对查询进行快照。如果使用了该选项，
    查询就在"<sub>id</sub>"索引上遍历执行，这样可以保证每个文档只被返回一次。例如:
    db.foo.find().snapshot(), 使用快照会使查询变慢，所以应该只在必要的时候使用快照。
    所有返回单批结果的查询都被有效的进行了快照。
</p></div>

</div>

<div id="outline-container-4-12-2" class="outline-4">
<h4 id="sec-4-12-2">游标生命周期</h4>
<div class="outline-text-4" id="text-4-12-2">

<p>    服务端的游标会消耗内存和其他资源。释放的资源可以被数据库另做他用。应该尽量保证尽快释放游标。
    如果一个游标在10分钟内没有被使用的话，数据库游标也会自动销毁。
    大部分驱动程序都实现了一个叫immortal的函数， 告知数据库不要让游标超时。如果关闭了游标超时时间，
    则一定要迭代完游标或者主动将其销毁，否则会一直在数据库中消耗资源。
</p></div>
</div>

</div>

<div id="outline-container-4-13" class="outline-3">
<h3 id="sec-4-13">数据库命令</h3>
<div class="outline-text-3" id="text-4-13">

<p>   db.listCommands(): 可以看到所有的数据库命令
</p>
</div>

<div id="outline-container-4-13-1" class="outline-4">
<h4 id="sec-4-13-1">数据库命令工作原理</h4>
<div class="outline-text-4" id="text-4-13-1">

<p>    数据库命令总会返回一个包含"ok"键的文档，如果"ok"的值为1，说明命令执行成功，为0，则失败。
    mongodb命令被实现为一种特殊类型的查询，这些特殊的查询会在$cmd集合上执行。runCommand只是
    接受一个命令文档，并且执行与这个命令文档等价的查询。因此drop命令会被转换为如下代码:
</p>


<pre class="example">db.$cmd.findOne({"drop":"test"});
</pre>

<p>
    当mongodb服务器得到一个在$cmd集合上的查询时，不会对这个查询进行通常的查询处理，而是会使用特殊的逻辑对其
    进行处理。
    如果需要执行一个管理员命令可以使用adminCommand。
    mongodb中，数据库命令是少数与字段顺序相关的地方之一，命令名称必须是命令中第一个字段
</p></div>
</div>
</div>

</div>

<div id="outline-container-5" class="outline-2">
<h2 id="sec-5">索引</h2>
<div class="outline-text-2" id="text-5">

<p>  db.user.find({username:"user1"}).explain() # 分析查询语句
  db.user.find({username:"user1"}).explain().hint({"age:1"}) #建议使用{age:1}索引进行分析
  db.user.ensureIndex({"key":1}) # 建立索引， 如果索引建立的很慢，可以在另一个shell运行: 
  db.user.ensureIndex({"key":1, "ke2":-1}) # 建立复合索引
  db.currentOp()或者检查mongod的日志来查看索引创建的进度。
  mongodb限制每个集合上最多只有64个索引。
  如果查询的结果集超过32MB，排序时mongodb就会出错。
  在实际应用中，{"sortkey":1, "queryCriteria":1}类型的索引通常是很有用的。
  相互反转的索引是等价的，例如： {"age":1} &lt;=&gt; {"age":-1}, 
  {"age":1, "name":-1} &lt;=&gt;{"age":-1, "name":1}
</p>
</div>

<div id="outline-container-5-1" class="outline-3">
<h3 id="sec-5-1">使用覆盖索引</h3>
<div class="outline-text-3" id="text-5-1">

<p>   当一个索引包含用户请求的所有字段，可以认为这个索引覆盖了本次查询。在实际中应该优先使用覆盖索引。
   如果在一个含有数组的字段上做索引，这个索引永远无法覆盖查询，即便将数组字段从需要返回的字段中
   剔除，也无法使用覆盖索引。
</p></div>

</div>

<div id="outline-container-5-2" class="outline-3">
<h3 id="sec-5-2">隐式索引</h3>
<div class="outline-text-3" id="text-5-2">

<p>   如果拥有一个N个键的索引，那么就同时"免费"得到了所有这N个键的前缀组成的索引。
</p></div>

</div>

<div id="outline-container-5-3" class="outline-3">
<h3 id="sec-5-3">索引的使用</h3>
<div class="outline-text-3" id="text-5-3">

<p>   应该尽可能的让索引是右平衡的。"<sub>id</sub>"索引就是典型的右平衡索引。
   如果应用程序需要使用数据的机会多于较老的数据，那么mongodb只需要在内存中保留这棵树最右侧的分支(最近的数据)，
   而不必将整棵树留在内存中。
   在索引中，不存在的字段和null字段的存储方式是一样的，查询必须遍历每一个文档检查这个值是否真的为null还是
   根本不存在。如果使用稀疏索引，就不能使用{"$exists":true},也不能使用{"$exists":false}
   $or可以对每个字句都使用索引。应该尽可能的使用$in代替$or.
</p></div>

</div>

<div id="outline-container-5-4" class="outline-3">
<h3 id="sec-5-4">索引对象和数组(P96)</h3>
<div class="outline-text-3" id="text-5-4">

<p>   mongodb允许深入文档内部，对嵌套字段和数组建立索引。嵌套对象和数组字段可以与复合索引中的顶级字段一起使用。
   对嵌套文档本身建立索引与对嵌套文档的某个字段建立索引是不同的。
   db.users.ensureIndex({"loc.city":1})
   <b>db.users.find({"loc":{"ip":"xxx", "city":"xx", "state":"xx"}})， 查询优化器才会使用loc上的索引。    无法对db.users.find({"loc.city":"xx"})的查询使用索引。 通过实验发现结果与这条测试相反。</b>
</p>
</div>

<div id="outline-container-5-4-1" class="outline-4">
<h4 id="sec-5-4-1">索引数组</h4>
<div class="outline-text-4" id="text-5-4-1">

<p>    对数组建立的索引并不包含任何位置信息，无法使用数组索引查找特定位置的数组元素。
</p></div>
</div>

</div>

<div id="outline-container-5-5" class="outline-3">
<h3 id="sec-5-5">explain 命令返回值的解释</h3>
<div class="outline-text-3" id="text-5-5">




<pre class="example">"cursor": 本次查询使用的索引名，如果是BasiciCursor, 表示未使用索引
"isMultike": 本次查询是否使用了多建索引
"n":xx 本次查询返回的文档数量
"nscannedObjects":xx mongodb按索引指针去磁盘上查找实际文档的次数。
"nscanned":xx 如果有使用索引， 那么该数字就是查找过的索引条目数量。
"scanAndOrder": 是否在内存中对结果进行排序
"indexOnly": 是否只使用索引就能完成此次查询
"nYields": 为了写入请求能够顺利进行，本次查询暂停的次数
"millis":xx 数据库执行本次查询所耗费的毫秒数
"indexBounds":xx 描述了索引使用的情况，给出了索引的遍历范围
</pre>

</div>

</div>

<div id="outline-container-5-6" class="outline-3">
<h3 id="sec-5-6">索引类型</h3>
<div class="outline-text-3" id="text-5-6">

<p>   db.user.ensureIndex({"username":1}, {"unique":true}) # 唯一索引
   db.user.ensureIndex({"username":1}, {"unique":true, "dropDups":true}) # 会删除重复的值，删除第一个
   db.user.ensureIndex({"username":1}, {"sparse":true}) 
   db.user.ensureIndex({"a":1}, {"name": "xx"}) # 为创建的索引取一个名字
   db.user.dropIndex("x<sub>1</sub><sub>y</sub><sub>1</sub>") #删除索引， 索引的名字可以通过db.user.getIndexes()来获取
</p></div>
</div>

</div>

<div id="outline-container-6" class="outline-2">
<h2 id="sec-6">特殊的索引和集合</h2>
<div class="outline-text-2" id="text-6">


</div>

<div id="outline-container-6-1" class="outline-3">
<h3 id="sec-6-1">固定集合</h3>
<div class="outline-text-3" id="text-6-1">

<p>   db.createCollectioin("my<sub>collection</sub>", {"capped":true, "size":10000, "max":100}) #创建固定集合，10000字节或100条数据
   db.runCommand({"convertToCapped":"test", "size":10000}) #将test集合转换为固定集合，但是无法将固定集合转换为非固定集合
   对固定集合可以进行一种特殊的排序，成为"自然排序", 返回结果集中文档的顺序就是文档在磁盘上的顺序
   循环游标是一种特殊的游标，只能用在固定集合上，如果超过10分钟没有新结果，循环游标会被释放。
</p></div>

</div>

<div id="outline-container-6-2" class="outline-3">
<h3 id="sec-6-2">TTL 索引</h3>
<div class="outline-text-3" id="text-6-2">

<p>   该索引允许为每一个文档设置一个超时时间，一个文档达到预设置的老化程度之后会被删除。
   db.user.ensureIndex({"lastUpdated":1}, {"expireAfterSecs": 60*60*2}) #
</p>
</div>
</div>
</div>

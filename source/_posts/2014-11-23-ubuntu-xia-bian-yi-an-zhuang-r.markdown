---
layout: post
title: "ubuntu 下编译安装 R"
date: 2014-11-23 09:21:01 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 使用包管理安装R语言</a>
<ul>
<li><a href="#sec-1-1">1.1 更新ubuntu的软件源</a></li>
<li><a href="#sec-1-2">1.2 安装指定版本的R</a></li>
</ul>
</li>
<li><a href="#sec-2">2 编译安装R</a>
<ul>
<li><a href="#sec-2-1">2.1 R的依赖软件</a></li>
<li><a href="#sec-2-2">2.2 R源码下载</a></li>
<li><a href="#sec-2-3">2.3 编译安装</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">使用包管理安装R语言</h2>
<div class="outline-text-2" id="text-1">


</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">更新ubuntu的软件源</h3>
<div class="outline-text-3" id="text-1-1">

<p>   找到R语言的ubuntu的软件源： <a href="http://mirror.bjtu.edu.cn/cran/bin/linux/ubuntu/">http://mirror.bjtu.edu.cn/cran/bin/linux/ubuntu/</a>
   ubuntu 14.04的代号是trusty, 因此需要在source.list文件最后一行添加如下内容：
</p>


<pre class="example">deb http://mirror.bjtu.edu.cn/cran/bin/linux/ubuntu trusty/
</pre>

<p>
   之后更新源： apt-get update
</p></div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">安装指定版本的R</h3>
<div class="outline-text-3" id="text-1-2">




<pre class="example">sudo apt-get install r-base-core=2.15.3
</pre>


</div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">编译安装R</h2>
<div class="outline-text-2" id="text-2">


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">R的依赖软件</h3>
<div class="outline-text-3" id="text-2-1">

<p>   在安装R前，需要先安装如下的依赖软件：
</p>


<pre class="example">sudo aptitude install build-essential gfortran libreadline6-dev libxt-dev
</pre>

</div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">R源码下载</h3>
<div class="outline-text-3" id="text-2-2">

<p>   R语言的官方网站是： <a href="http://www.r-project.org/">http://www.r-project.org/</a>
   可从该网站中下载相应的源码
</p></div>

</div>

<div id="outline-container-2-3" class="outline-3">
<h3 id="sec-2-3">编译安装</h3>
<div class="outline-text-3" id="text-2-3">

<p>   开始编译安装R，执行如下命令
</p>


<pre class="example">./configure --prefix=/which/path/to/install --enable-R-shlib
make
make install
</pre>

</div>
</div>
</div>

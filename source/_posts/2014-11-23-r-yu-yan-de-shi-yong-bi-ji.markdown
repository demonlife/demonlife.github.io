---
layout: post
title: "R 语言的使用笔记"
date: 2014-11-23 09:51:32 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 安装fortunes</a>
<ul>
<li><a href="#sec-1-1">1.1 安装过程如下</a></li>
<li><a href="#sec-1-2">1.2 fortunes包的使用</a></li>
</ul>
</li>
<li><a href="#sec-2">2 formatR包</a>
<ul>
<li><a href="#sec-2-1">2.1 安装</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">安装fortunes</h2>
<div class="outline-text-2" id="text-1">

<p>  fortunes库是一个R语言的语录集，这些都是R语言智慧的精华，让R语言的后辈使用者，可以更了解R语言的本身，了解R的精神。
</p>
</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">安装过程如下</h3>
<div class="outline-text-3" id="text-1-1">




<pre class="example">1. 先启动R， 在命令行中执行R即可
2. install.packages("fortunes") # 安装fortunes包
3. library(fortunes) #加载fortunes包 
4. ?fortunes # 查看帮助
</pre>

</div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">fortunes包的使用</h3>
<div class="outline-text-3" id="text-1-2">




<pre class="example">fortune() #随机查看一条语录
fortune(108) #指定查看一条记录
完整语录的下载地址是： http://cran.r-project.org/web/packages/fortunes/vignettes/fortunes.pdf
</pre>

</div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">formatR包</h2>
<div class="outline-text-2" id="text-2">

<p>  formatR包是一个实用的包，提供了R代码格式化功能，可以自动设置空格、缩进、换行等代码格式
</p>
</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">安装</h3>
<div class="outline-text-3" id="text-2-1">




<pre class="example">install.packages("formatR") # 3.1.2中已经没有formatR包了
library(formatR)
</pre>

</div>
</div>
</div>

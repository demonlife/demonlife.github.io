---
layout: post
title: "DNS常识"
date: 2014-12-05 14:41:31 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 DNS基本概念</a>
<ul>
<li><a href="#sec-1-1">1.1 域名的划分</a></li>
<li><a href="#sec-1-2">1.2 域名的相关术语</a></li>
</ul>
</li>
<li><a href="#sec-2">2 域名查询过程</a></li>
<li><a href="#sec-3">3 域名系统组织架构</a></li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">DNS基本概念</h2>
<div class="outline-text-2" id="text-1">

<ul>
<li>域名
    如在浏览器地址栏输入www.google.com等我们称之为域名，域名即网站名称
</li>
</ul>


</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">域名的划分</h3>
<div class="outline-text-3" id="text-1-1">

<p>   根域、顶级域、二级域、子域   
   域名采用层次化的方式进行组织，每一个点代表一个层级。一个域名完整的格式为www.google.com. 
   最末尾的点代表根域，常常省略；com即顶级域（TLD）；google.com即二级域。依次类推，
   还有三级域、四级域等等。子域是一个相对的概念，google.com是com的子域，
   www.google.com是baidu.com的子域。
</p><ul>
<li>域名系统  
     即DNS（Domain Name System）。DNS主要解决两方面的问题：域名本身的增删改查以及域名到IP如何映射。
</li>
</ul>

</div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">域名的相关术语</h3>
<div class="outline-text-3" id="text-1-2">

<p>   正向解析  查找域名对应IP的过程。
   反向解析  查找IP对应域名的过程。
   解析器  即resolver，处于DNS客户端的一套系统，用于实现正向解析或者反向解析。
   权威DNS  处于DNS服务端的一套系统，该系统保存了相应域名的权威信息。
   权威DNS即通俗上“这个域名我说了算”的服务器。
</p>
<p>
   递归DNS  又叫local dns。递归DNS可以理解为是一种功能复杂些的resolver，其核心功能一个是缓存、
   一个是递归查询。收到域名查询请求后其首先看本地缓存是否有记录，如果没有则一级一级的查询根、
   顶级域、二级域……直到获取到结果然后返回给用户。日常上网中运营商分配的DNS即这里所说的递归DNS。
</p>
<p>
   转发DNS  转发DNS是一种特殊的递归。如果本地的缓存记录中没有相应域名结果时，其将查询请求转发给另外
   一台DNS服务器，由另外一台DNS服务器来完成查询请求。
</p>
<p>
   公共DNS  公共DNS属于递归DNS。其典型特征为对外一个IP，为所有用户提供公共的递归查询服务。
</p></div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">域名查询过程</h2>
<div class="outline-text-2" id="text-2">

<ul>
<li>以输入www.google.com为例:
<ol>
<li>用户输入www.google.com，浏览器调用操作系统resolver发起域名查询，此处不考虑浏览器的域名缓存；
</li>
</ol>

<p>    resolver封装一个dns请求报文，并将其发给运营商分配的local dns地址（或者用户自己配置的公共dns）；
</p><ol>
<li>local dns查询缓存，如果命中则返回响应结果；否则向根服务器发起查询；
</li>
<li>根服务器返回com地址。每一层级的DNS服务器都有缓存，实际都是先查缓存，没有缓存才返回下级域，
</li>
</ol>

<p>    此处不再重复；
</p><ol>
<li>local dns查询com。com返回google.com地址；
</li>
<li>local dns查询google.com，google.com返回www.google.com对应记录结果。
</li>
</ol>

</li>
<li>理论上讲域名查询有两种方式:
    迭代查询 A问B一个问题，B不知道答案说你可以问C，然后A再去问C，C推荐D，然后A继续问D，如此迭代…
    递归查询  A问B一个问题，B问C，C问D… 然后D告诉C，C告诉B，B告诉A
    上述过程中从resolver到递归DNS再到根的查询过程为递归查询，递归DNS到根、
    到com、到google.com的过程为迭代查询。
    注意，递归查询需要从系统层面来看，很难单纯的说一台DNS实现了递归查询。
</li>
</ul>

</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">域名系统组织架构</h2>
<div class="outline-text-2" id="text-3">

<p>  DNS是全球互联网中最重要的基础服务之一，也是如今唯一的一种有中心点的服务。
</p>
</div>
</div>

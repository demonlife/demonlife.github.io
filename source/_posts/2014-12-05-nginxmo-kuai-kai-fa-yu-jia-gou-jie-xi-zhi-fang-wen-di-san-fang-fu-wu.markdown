---
layout: post
title: "nginx模块开发与架构解析之访问第三方服务"
date: 2014-12-05 08:48:35 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 简介</a></li>
<li><a href="#sec-2">2 upstream的使用方式</a>
<ul>
<li><a href="#sec-2-1">2.1 ngx_http_upstream_t</a></li>
<li><a href="#sec-2-2">2.2 设置upstream的限制性参数</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">简介</h2>
<div class="outline-text-2" id="text-1">

<p>  当需要访问第三方服务时，nginx提供了两种全异步方式来与第三方服务器通信：
  upstream与subrequest。upstream可以保证在与第三方服务器交互时(包括三次握手建立
  tcp连接，发送请求， 接受响应，四次握手关闭tcp连接等)不会阻塞Nginx进程处理其他请求。
  在开发http模块时，如果需要访问第三方服务是不能自己简单的用套接字编程实现的，这样
  会破坏nginx优秀的全异步架构。subrequest只是分解复杂请求的一种设计模式，本质上
  与访问第三方服务没有任何关系。subrequest访问第三方服务最终也是基于upstream实现的。
</p></div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">upstream的使用方式</h2>
<div class="outline-text-2" id="text-2">

<p>  upstream提供了8个回调方法，用于只需要视自己的需要实现其中几个回调方法就可以了。
</p><ul>
<li>upstream嵌入到一个请求中的步骤
    模块在处理任何一个请求时都有ngx<sub>http</sub><sub>request</sub><sub>t结构对象r</sub>, 而r中有一个
    ngx<sub>http</sub><sub>upstream</sub><sub>t类型的成员upstream。因此只要设置r</sub>-&gt;upstream成员即可。
</li>
</ul>


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">ngx_http_upstream_t</h3>
<div class="outline-text-3" id="text-2-1">

<ul>
<li>upstream有3中处理上游响应包体的方式
     当请求的ngx<sub>http</sub><sub>request</sub><sub>t结构体中subrequest</sub><sub>in</sub><sub>memory标志为1，将采用upstream不转发</sub>
     响应包体到下游，由http模块实现的input<sub>filter方法处理包体。当subrequest</sub><sub>in</sub><sub>memory为0时</sub>
     upstream会转发响应包体，当ngx<sub>http</sub><sub>upstream</sub><sub>conf</sub><sub>t配置结构体中的buffering为1时，将开启</sub>
     更多的内存和磁盘文件用于缓存上游的响应包体，即上游网速更快。当buffering为0时，将使用固定
     大小的缓冲区来转发响应包体。
</li>
</ul>

</div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">设置upstream的限制性参数</h3>
<div class="outline-text-3" id="text-2-2">

<p>   ngx<sub>http</sub><sub>upstream</sub><sub>t中的conf成员，用于设置upstream模块处理请求时的参数。包括连接，发送，</sub>
   接受的超时时间等。
   nginx.conf文件中提供的配置项大都是用来设置ngx<sub>http</sub><sub>upstream</sub><sub>conf</sub><sub>t结构体中的成员的。</sub>
   使用前面介绍的14个预设方法可以非常简单地通过nginx.conf配置文件设置ngx<sub>http</sub><sub>upstream</sub><sub>conf</sub><sub>t</sub>
   结构体。例如，可以将ngx<sub>http</sub><sub>upstream</sub><sub>conf</sub><sub>t类型的变量放到ngx</sub><sub>http</sub><sub>mytest</sub><sub>conf</sub><sub>t结构体中</sub>
</p></div>
</div>
</div>

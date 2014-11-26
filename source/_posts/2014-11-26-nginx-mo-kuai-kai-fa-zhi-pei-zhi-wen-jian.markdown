---
layout: post
title: "nginx 模块开发之配置文件"
date: 2014-11-26 08:51:53 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 运行中的nginx进程间的关系</a></li>
<li><a href="#sec-2">2 Nginx服务的基本配置(P34)</a>
<ul>
<li><a href="#sec-2-1">2.1 用于调试进程和定位问题的配置项</a></li>
<li><a href="#sec-2-2">2.2 正常运行的配置项</a></li>
<li><a href="#sec-2-3">2.3 优化性能的配置项</a></li>
<li><a href="#sec-2-4">2.4 事件类配置项</a></li>
<li><a href="#sec-2-5">2.5 虚拟主机与请求的分发</a></li>
<li><a href="#sec-2-6">2.6 文件路径的定义</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">运行中的nginx进程间的关系</h2>
<div class="outline-text-2" id="text-1">

<p>  在正式提供服务的产品环境下，部署nginx时都是使用一个master进程来管理多个worker进程
  一般情况下，worker进程的数量与服务器上的cpu核心数相等。
</p></div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">Nginx服务的基本配置(P34)</h2>
<div class="outline-text-2" id="text-2">


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">用于调试进程和定位问题的配置项</h3>
<div class="outline-text-3" id="text-2-1">

<ul>
<li>error日志的设置
     error<sub>log</sub> /path/file level;
     level是日志的输出级别， 取值范围是debug, info, notice, warn, error, crit,
     alert,emerg, 从左至右级别依次增大，当设定为一个级别时，大于或等于该级别的日志
     都会被输出到/path/file文件中，小于该级别的日志则不会输出。
</li>
</ul>

</div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">正常运行的配置项</h3>
<div class="outline-text-3" id="text-2-2">

</div>

</div>

<div id="outline-container-2-3" class="outline-3">
<h3 id="sec-2-3">优化性能的配置项</h3>
<div class="outline-text-3" id="text-2-3">

</div>

</div>

<div id="outline-container-2-4" class="outline-3">
<h3 id="sec-2-4">事件类配置项</h3>
<div class="outline-text-3" id="text-2-4">

</div>

</div>

<div id="outline-container-2-5" class="outline-3">
<h3 id="sec-2-5">虚拟主机与请求的分发</h3>
<div class="outline-text-3" id="text-2-5">

<p>   每一个server块就是一个虚拟主机
</p><ul>
<li>location 的匹配规则



<pre class="example">=: 把URI作为字符串，以便与参数中的uri做完全匹配
location = / {
    # 当只有用户请求是/时， 才会使用该location下的配置
}
～: 匹配URI时是字母大小写敏感的
~*: 表示匹配URI时忽略字母大小写问题
^~: 匹配URI时只需要其前半部分与URI参数匹配即可。
location ^~ /images/ {
    # 以/images/ 开始的请求都会匹配上
}
@: 仅用于Nginx服务内部请求之间的重定向，带有@的location不直接处理用户请求
在uri参数里是可以使用正则表达式的
location ~* \.(gif|jpg|jpeg)$ {
...
}
</pre>

<p>
     location是有顺序的，当一个请求有可能匹配多个location时，实际上这个请求
     会被第一个location处理。
</p></li>
</ul>

</div>

</div>

<div id="outline-container-2-6" class="outline-3">
<h3 id="sec-2-6">文件路径的定义</h3>
<div class="outline-text-3" id="text-2-6">



</div>
</div>
</div>

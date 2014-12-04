---
layout: post
title: "Nginx 模块开发与架构设计之请求上下文"
date: 2014-11-30 08:42:05 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 读取配置信息</a>
<ul>
<li><a href="#sec-1-1">1.1 分配用于保存配置参数的数据结构</a>
<ul>
<li><a href="#sec-1-1-1">1.1.1 设定配置项的解析方式</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">读取配置信息</h2>
<div class="outline-text-2" id="text-1">

<p>  nginx已经为用户提供了强大的配置项解析机制，同时它还支持“-s reload”命令—在不重启服务的情况下
  可使配置生效。
  处理http配置项可以分为下面4个步骤：
</p>


<pre class="example">1.创建数据结构用于存储配置项对应的参数。
2.设定配置项在nginx.conf中出现时的限制条件与回调方法。
3.实现第2步中的回调方法，或者使用Nginx框架预设的14个回调方法。
4.合并不同级别的配置块中出现的同名配置项。
</pre>

<p>
  这四个步骤是通过ngx_http_module_t和ngx_command_t有机的结合起来的。
</p>
</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">分配用于保存配置参数的数据结构</h3>
<div class="outline-text-3" id="text-1-1">

<p>   定义如下的结构，用于保存配置参数：
{% codeblock lang:c %}
   typedef struct {
       ngx_str_t my_str;
       ngx_int_t my_num;
       ngx_flag_t my_flag;
       size_t my_size;
       ngx_array_t* my_str_array;
       ngx_array_t* my_keyval;
       off_t my_off;
       ngx_msec_t my_msec;
       time_t my_sec;
       ngx_bufs_t my_bufs;
       ngx_uint_t my_enum_seq;
       ngx_uint_t my_bitmask;
       ngx_uint_t my_access;
       ngx_path_t* my_path;
   } ngx_http_mytest_conf_t;
{% endcodeblock %}
   HTTP框架在解析nginx.conf文件时只要遇到http{}、server{}或者location{}配置块就会立刻分配一个新的
   ngx_http_mytest_conf_t结构体。HTTP模块感兴趣的配置项需要统一地使用一个struct结构体来保存
   （否则HTTP框架无法管理），如果nginx.conf文件中在http{}下有多个server{}或者location{}，
   那么这个struct结构体在Nginx进程中就会存在多份实例。
   Nginx会使用前面讲到的ngx<sub>http</sub><sub>module</sub><sub>t中的回调方法来管理ngx</sub><sub>http</sub><sub>mytest</sub><sub>conf</sub><sub>t结构体。其中的</sub>
   create<sub>main</sub><sub>conf、create</sub><sub>srv</sub><sub>conf、create</sub><sub>loc</sub><sub>conf这3个回调方法负责把我们分配的用于保存</sub>
   配置项的结构体传递给HTTP框架。
</p>
</div>

<div id="outline-container-1-1-1" class="outline-4">
<h4 id="sec-1-1-1">设定配置项的解析方式</h4>
<div class="outline-text-4" id="text-1-1-1">

<p>    首先回顾一下ngx<sub>command</sub><sub>t结构体</sub>
</p></div>
</div>
</div>
</div>

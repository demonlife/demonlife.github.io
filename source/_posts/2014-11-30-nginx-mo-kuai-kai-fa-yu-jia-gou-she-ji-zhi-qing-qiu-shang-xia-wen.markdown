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
<li><a href="#sec-1-1-2">1.1.2 合并配置项</a></li>
<li><a href="#sec-1-1-3">1.1.3 http 配置模型</a></li>
<li><a href="#sec-1-1-4">1.1.4 http配置模型的内存布局(P144)</a></li>
</ul></li>
</ul>
</li>
<li><a href="#sec-2">2 error日志的用法</a>
<ul>
<li><a href="#sec-2-1">2.1 log参数说明</a></li>
</ul>
</li>
<li><a href="#sec-3">3 请求的上下文N</a>
<ul>
<li><a href="#sec-3-1">3.1 使用http上下文</a>
<ul>
<li><a href="#sec-3-1-1">3.1.1 http框架维护上下文</a></li>
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

<div id="outline-container-1-1-2" class="outline-4">
<h4 id="sec-1-1-2">合并配置项</h4>
<div class="outline-text-4" id="text-1-1-2">

<p>    如果需要在server块，http块内出现的配置项也生效，可以通过merge_loc_conf方法来实现。如果合并
    具体就取决于具体的merge_loc_conf实现。
</p></div>

</div>

<div id="outline-container-1-1-3" class="outline-4">
<h4 id="sec-1-1-3">http 配置模型</h4>
<div class="outline-text-4" id="text-1-1-3">

<p>    当nginx检测到http{&hellip;}配置项时，http配置模型就启动了，这时会首先建立一个ngx<sub>http</sub><sub>conf</sub><sub>ctx</sub><sub>t</sub>
    结构体。
</p></div>

</div>

<div id="outline-container-1-1-4" class="outline-4">
<h4 id="sec-1-1-4">http配置模型的内存布局(P144)</h4>
<div class="outline-text-4" id="text-1-1-4">

</div>
</div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">error日志的用法</h2>
<div class="outline-text-2" id="text-2">

<p>  nginx的日志模块(此处所说的日志模块是ngx<sub>errlog</sub><sub>module模块，而ngx</sub><sub>http</sub><sub>log</sub><sub>module模块是用于记录</sub>
  http请求的访问日志的。两者功能不同，在实现上也没有任何关系)为其他模块提供了基本的记录日志功能。
  出于跨平台的考虑， 日志模块提供了相当多的接口，主要是有些平台不支持可变参数。
</p><ul>
<li>nginx提供的接口
</li>
</ul>


{% codeblock lang:c %}
    #define ngx_log_error(level, log, args...) \
        if ((log)->log_level >= level) ngx_log_error_core(level, log, args)

    #define ngx_log_debug(level, log, args...) \
        if ((log)->log_level & level) \
            ngx_log_error_core(NGX_LOG_DEBUG, log, args)

    void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, \
                            ngx_err_t err, const char *fmt, ...)
{% endcodeblock %}
<p>
    日志模块记录日志的核心功能是由ngx<sub>log</sub><sub>error</sub><sub>core方法实现的</sub>
</p>
</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">log参数说明</h3>
<div class="outline-text-3" id="text-2-1">

<p>   在开发http模块时，并不关心log参数的构造，因为在处理请求时ngx<sub>http</sub><sub>request</sub><sub>t结构中的connection</sub>
   成员就有一个ngx<sub>log</sub><sub>t类型的log成员，可以传给ngx</sub><sub>log</sub><sub>error宏和ngx</sub><sub>log</sub><sub>debug宏记录日志。</sub>
   在读取配置阶段，ngx<sub>conf</sub><sub>t结构也有log成员可以用来记录日志</sub>(读取配置阶段时的日志信息将输出到
   控制台屏幕)。
   如果只想把相应的信息记录到日志文件中， 完全不需要关心ngx<sub>log</sub><sub>t类型的log参数是如何构造的，</sub>
   特别是在编写http模块时，http框架要求所有的http模块都使用它提供的log，如果重新定义ngx<sub>log</sub><sub>t中的</sub>
   handler方法，或者修改data指向的地址，那么很可能会造成一系列问题。
</p></div>
</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">请求的上下文N</h2>
<div class="outline-text-2" id="text-3">

<p>  nginx中，上下文有很多含义， 本节所指的是HTTP框架为每个http请求所准备的结构体。一个请求对应于每
  一个http模块都可以有一个独立的上下文结构体。
</p>
</div>

<div id="outline-container-3-1" class="outline-3">
<h3 id="sec-3-1">使用http上下文</h3>
<div class="outline-text-3" id="text-3-1">

<p>   ngx<sub>http</sub><sub>get</sub><sub>module</sub><sub>ctx和ngx</sub><sub>http</sub><sub>set</sub><sub>ctx这两个宏可以完成HTTP上下文的设置和使用</sub>
</p><ul>
<li>定义
</li>
</ul>


{% codeblock lang:c %}
     #define ngx_http_get_module_ctx(r, module) (r)->ctx[module.ctx_index]
     #define ngx_http_set_ctx(r, c, module) r->ctx[module.ctx_index] = c;
{% endcodeblock %}
<p>
     在任何一个http模块中，都可以使用ngx<sub>http</sub><sub>get</sub><sub>module</sub><sub>ctx获取所有http模块为该请求创建的</sub>
     上下文结构。
</p><ul>
<li>一个简单的例子
<ol>
<li>建立mytest模块的上下文结构体，如 ngx<sub>http</sub><sub>mytest</sub><sub>ctx</sub><sub>t</sub>
</li>
</ol>

</li>
</ul>


{% codeblock lang:c %}
        typedef struct {
            ngx_uint_t my_step;
        } ngx_http_mytest_ctx_t;
{% endcodeblock %}
<ol>
<li>当请求第一次进入mytest模块处理时， 创建ngx<sub>http</sub><sub>mytest</sub><sub>ctx</sub><sub>t结构体，并设置到这个请求</sub>
        的上下文
</li>
</ol>


{% codeblock lang:c %}
        static ngx_int_t ngx_http_mytest_handler(ngx_http_request_t *r) {
            // 获取上下文结构体
            ngx_http_mytest_ctx_t *myctx = ngx_http_get_module_ctx(r, ngx_http_mytest_module);
            // 如果之前没有设置过上下文，则应当返回NULL
            if (myctx == NULL) {
                myctx = ngx_palloc(r->pool, sizeof(ngx_http_mytest_ctx_t));
                if (myctx == NULL) {
                    return NGX_ERROR;
                }
                // 将刚分配的结构体设置到当前请求的上下文中
                ngx_http_set_ctx(r, myctx, ngx_http_mytest_module);
            }
            // 使用myctx这个上下文结构体
        }
{% endcodeblock %}
<p>
        http框架可以对一个请求保证，无论调用多少次ngx<sub>http</sub><sub>get</sub><sub>module</sub><sub>ctx宏都只取到同一个上下</sub>
        文结构体。
</p>
</div>

<div id="outline-container-3-1-1" class="outline-4">
<h4 id="sec-3-1-1">http框架维护上下文</h4>
<div class="outline-text-4" id="text-3-1-1">

<p>    ngx<sub>http</sub><sub>request</sub><sub>t结构的ctx成员是一个指针的指针，http框架就在ctx数组中保存所有HTTP模块</sub>
    上下文结构体的指针的。
</p></div>
</div>
</div>
</div>

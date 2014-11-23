---
layout: post
title: "Nginx 源码学习笔记"
date: 2014-11-21 13:18:32 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 error 日志的用法</a></li>
<li><a href="#sec-2">2 请求的上下文</a>
<ul>
<li><a href="#sec-2-1">2.1 上下文的使用</a></li>
<li><a href="#sec-2-2">2.2 http框架如何维护上下文结构</a></li>
</ul>
</li>
<li><a href="#sec-3">3 访问第三方服务</a>
<ul>
<li><a href="#sec-3-1">3.1 upstream的使用</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">error 日志的用法</h2>
<div class="outline-text-2" id="text-1">

<p>  nginx的日志模块（此处说的是ngx_errlog_module,而不是ngx_http_log_module
  ngx_http_log_module模块是用于记录http请求的访问日志的）为其他模块提供了基本
  的记录日志功能。出于夸平台的考虑，日志模块提供了很多的接口， 主要是因为有些平台
  不支持可变参数。首先看一下日志模块对于支持可变参数平台而提供的3个接口：
</p>


<pre class="example">#define ngx_log_error(level, log, args...) \
    if ((log)-&gt;log_level &gt;= level) ngx_log_error_core(level, log, args)
#define ngx_log_debug(level, log, args...) \
    if ((log)-&gt;log_level &amp; level)          \
        ngx_log_error_core(NGX_LOG_DEBUG, log, args)
void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err, const char* fmt, ...);
</pre>

<p>
  上述函数定义中各个参数的说明见P150
  使用ngx_log_error宏记录日志时，如果传人的level级别&lt;=log参数中的日志级别(通常由nginx.conf配置
  文件指定),就会输出日志内容，否则这条日志会被忽略。
  在使用ngx_log_debug宏时，level表达不再是级别，而是日志类型，应为ngx_log_debug宏记录的日志
  必须是NGX_LOG_DEBUG调试级别的，此处的level由各个子模块定义，level的取值见P151。
  例如:
</p>


<pre class="example">当http模块调用ngx_log_debug宏时，传人的level参数是NGX_LOG_DEBUG_HTTP,这时如果log参数
不属于HTTP模块，如使用了event事件模块的log，则不会输出任何日志。
</pre>

<p>
  log参数：
</p>


<pre class="example">在开发http模块时，不用关心log参数的构造，在处理请求时ngx_http_request_t结构中的connection
成员就有一个ngx_log_t类型的log成员，在读取配置阶段,ngx_conf_t 结构也有log成员可以用来记录
日志，读取配置阶段时的日志信息都将输出到控制台
</pre>

<p>
  注意： printf/sprintf支持的一些格式转换在ngx<sub>vslprintf中是不支持，或者意义不同</sub>
</p></div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">请求的上下文</h2>
<div class="outline-text-2" id="text-2">

<p>  此处的上下文是指http框架为每个http请求所准备的结构体。http框架定义的
  上下文是针对于http请求的，而且一个http请求对应于每一个http模块都可以
  有一个独立上下文结构体。
</p>
</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">上下文的使用</h3>
<div class="outline-text-3" id="text-2-1">

<p>   有两个宏可以完成上下文的设置和使用：
{% codeblock lang:c %}
   // r 参数是ngx_http_request_t指针， module是当前http模块对象，
   // 例如在之前的例子中定义的ngx_module_t类型的ngx_http_mytest_moudle结构体
   // 返回某个http模块的上下文结构体指针，如果该http模块没有设置过上下文
   // 将返回NULL。
   #define ngx_http_get_module_ctx(r, module) (r)->ctx[module.ctx_index]
   // r: ngx_http_request_t 指针
   // c: 准备设置的上下文结构体指针
   // module: http模块对象
   #define ngx_http_set_ctx(r, c, module) r->ctx[module.ctx_index] = c;
{% endcodeblock %}
   使用示例：
{% codeblock lang:c %}
   // 首先建立mytest模块的上下文结构体
   typedef struct {
       ngx_uint_t my_step;
   } ngx_http_mytest_ctx_t;

   static ngx_int_t ngx_http_mytest_handler(ngx_http_request_t *r) {
       // 首先调用宏获取上下文结构体
       // HTTP框架可以对一个请求保证，无论调用多少次ngx_http_get_module_ctx宏都只取到同一个上下文。
       ngx_http_mytest_ctx_t *myctx = ngx_http_get_module_ctx(r, ngx_http_mytest_module);
       // 如果之前没有设置过上下文， 那么应当返回NULL，此处开始设置
       if (myctx == NULL) {
           myctx = ngx_palloc(r->pool, sizeof(ngx_http_mytest_ctx_t));
           if (myctx == NULL) {
               return NGX_ERROR;
           }
           ngx_http_set_ctx(r, myctx, ngx_http_mytest_module);
       }
       //...
   }
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">http框架如何维护上下文结构</h3>
<div class="outline-text-3" id="text-2-2">

<p>   在结构体ngx<sub>http</sub><sub>request</sub><sub>s中有定义：</sub>
{% codeblock lang:c %}
   struct ngx_http_request_s {
       //...
       void **ctx;
       //...
   };
{% endcodeblock %}
   其实ngx_http_get_module_ctx和ngx_http_set_ctx只是去获取或设置
   ctx数组中相应HTTP模块的指针而已。
</p></div>
</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">访问第三方服务</h2>
<div class="outline-text-2" id="text-3">

<p>  nginx提供了两种全异步方式来与第三方服务器通信：upstream, subrequest.
  upstream可以保证在与第三方服务器交互时(包括三次握手建立TCP连接，发送
  请求，接受响应，四次握手关闭TCP连接等)不会阻塞nginx进程处理其他请求。
  subrequest只是分解复杂请求的一种设计模式， 本质上与访问第三方服务没有
  任何关系，subrequest访问第三方服务最终也是基于upstream实现的。
</p>
<p>  
  upstream和subrequest的设计目标是完全不同的， upstream被定义为访问上游
  服务器，即将nginx定义为代理服务器，首要功能是透传，其次才是以TCP获取
  第三方服务器的内容。subrequest是从属请求，将会为客户请求创建子请求。
</p>
</div>

<div id="outline-container-3-1" class="outline-3">
<h3 id="sec-3-1">upstream的使用</h3>
<div class="outline-text-3" id="text-3-1">

<p>   ngx_http_request_t结构体中有一个ngx_http_upstream_t的r成员
   启用upstream机制的示意图见P161
</p></div>
</div>
</div>

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
<li><a href="#sec-3-1">3.1 upstream的使用</a>
<ul>
<li><a href="#sec-3-1-1">3.1.1 ngx_http_upstream_t</a></li>
<li><a href="#sec-3-1-2">3.1.2 设置upstream的限制性参数</a></li>
</ul>
</li>
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
</p>
</div>

<div id="outline-container-3-1-1" class="outline-4">
<h4 id="sec-3-1-1">ngx_http_upstream_t</h4>
<div class="outline-text-4" id="text-3-1-1">

<p>    ngx_http_upstream_t结构体里有些成员仅仅是在upstream模块内部使用
    的。
{% codeblock lang:c %}
    struct ngx_http_upstream_s {
        ngx_http_upstream_handler_pt     read_event_handler;
        ngx_http_upstream_handler_pt     write_event_handler;
     
        ngx_peer_connection_t            peer;
     
        ngx_event_pipe_t                *pipe;
        
        // 决定发送什么请求给上游服务器，在实现create_request方法时需要设置它
        ngx_chain_t                     *request_bufs;
     
        ngx_output_chain_ctx_t           output;
        ngx_chain_writer_ctx_t           writer;
     
        // upstream访问时所有限制性参数
        ngx_http_upstream_conf_t        *conf;
     
        ngx_http_upstream_headers_in_t   headers_in;
        
        // 通过resolved可以直接指定上游服务器地址
        ngx_http_upstream_resolved_t    *resolved;
     
        ngx_buf_t                        from_client;
     
        ngx_buf_t                        buffer;
        off_t                            length;
     
        ngx_chain_t                     *out_bufs;
        ngx_chain_t                     *busy_bufs;
        ngx_chain_t                     *free_bufs;
     
        ngx_int_t                      (*input_filter_init)(void *data);
        ngx_int_t                      (*input_filter)(void *data, ssize_t bytes);
        void                            *input_filter_ctx;
     
    #if (NGX_HTTP_CACHE)
        ngx_int_t                      (*create_key)(ngx_http_request_t *r);
    #endif
        // 构造发往上游服务器的请求内容
        ngx_int_t                      (*create_request)(ngx_http_request_t *r);
        ngx_int_t                      (*reinit_request)(ngx_http_request_t *r);

        // 收到上游服务器的响应后会回调该方法
        ngx_int_t                      (*process_header)(ngx_http_request_t *r);
        void                           (*abort_request)(ngx_http_request_t *r);
        void                           (*finalize_request)(ngx_http_request_t *r,
                                             ngx_int_t rc);
        ngx_int_t                      (*rewrite_redirect)(ngx_http_request_t *r,
                                             ngx_table_elt_t *h, size_t prefix);
        ngx_int_t                      (*rewrite_cookie)(ngx_http_request_t *r,
                                             ngx_table_elt_t *h);
     
        ngx_msec_t                       timeout;
     
        ngx_http_upstream_state_t       *state;
     
        ngx_str_t                        method;
        ngx_str_t                        schema;
        ngx_str_t                        uri;
     
        ngx_http_cleanup_pt             *cleanup;
     
        unsigned                         store:1;
        unsigned                         cacheable:1;
        unsigned                         accel:1;
        unsigned                         ssl:1;
    #if (NGX_HTTP_CACHE)
        unsigned                         cache_status:3;
    #endif
     
        unsigned                         buffering:1;
        unsigned                         keepalive:1;
        unsigned                         upgrade:1;
     
        unsigned                         request_sent:1;
        unsigned                         header_sent:1;
    };
{% endcodeblock %}
    upstream有3中方法处理上游响应包体的方式，当请求的ngx_http_request_t结构体中
    subrequest_in_memory标志位为1时，upstream将不转发响应包体到下游，由http模块
    实现的input_filter方法处理包体，当subrequest_in_memory为0时，upstream会转发
    响应包体。当ngx_http_upstream_conf_t配置结构体中的buffering标志位为1时，将
    开启更多的内存和磁盘文件用于缓存上游的响应包体，意味着上游网速更快， 当buffering
    为0时，将使用固定大小的缓冲区来转发响应包体。
</p></div>

</div>

<div id="outline-container-3-1-2" class="outline-4">
<h4 id="sec-3-1-2">设置upstream的限制性参数</h4>
<div class="outline-text-4" id="text-3-1-2">


{% codeblock lang:c %}
    typedef struct {
        ngx_http_upstream_srv_conf_t    *upstream;

        // 以下三个超时时间是必须要设置的，应为默认值为0，如果不设置将永远无法与上游服务
        // 器建立起tcp连接， 可以将ngx_http_upstream_conf_t类型的变量放到ngx_http_mytest_conf_t
        // 结构体中，之后使用ngx_command_t来设置处理方法。
        // 连接上游服务器的超时时间，单位为毫秒
        ngx_msec_t                       connect_timeout;
        // 发送tcp包到上游服务器的超时时间，毫秒
        ngx_msec_t                       send_timeout;
        // 接受tcp包到上游服务器的超时时间，毫秒
        ngx_msec_t                       read_timeout;

        ngx_msec_t                       timeout;
     
        size_t                           send_lowat;
        size_t                           buffer_size;
     
        size_t                           busy_buffers_size;
        size_t                           max_temp_file_size;
        size_t                           temp_file_write_size;
     
        size_t                           busy_buffers_size_conf;
        size_t                           max_temp_file_size_conf;
        size_t                           temp_file_write_size_conf;
     
        ngx_bufs_t                       bufs;
     
        ngx_uint_t                       ignore_headers;
        ngx_uint_t                       next_upstream;
        ngx_uint_t                       store_access;
        ngx_flag_t                       buffering;
        ngx_flag_t                       pass_request_headers;
        ngx_flag_t                       pass_request_body;
     
        ngx_flag_t                       ignore_client_abort;
        ngx_flag_t                       intercept_errors;
        ngx_flag_t                       cyclic_temp_file;
     
        ngx_path_t                      *temp_path;
     
        ngx_hash_t                       hide_headers_hash;
        ngx_array_t                     *hide_headers;
        ngx_array_t                     *pass_headers;
     
        ngx_http_upstream_local_t       *local;
     
    #if (NGX_HTTP_CACHE)
        ngx_shm_zone_t                  *cache;
     
        ngx_uint_t                       cache_min_uses;
        ngx_uint_t                       cache_use_stale;
        ngx_uint_t                       cache_methods;
     
        ngx_flag_t                       cache_lock;
        ngx_msec_t                       cache_lock_timeout;
     
        ngx_flag_t                       cache_revalidate;
     
        ngx_array_t                     *cache_valid;
        ngx_array_t                     *cache_bypass;
        ngx_array_t                     *no_cache;
    #endif
     
        ngx_array_t                     *store_lengths;
        ngx_array_t                     *store_values;
     
        signed                           store:2;
        unsigned                         intercept_404:1;
        unsigned                         change_buffering:1;
     
    #if (NGX_HTTP_SSL)
        ngx_ssl_t                       *ssl;
        ngx_flag_t                       ssl_session_reuse;
    #endif
     
        ngx_str_t                        module;
    } ngx_http_upstream_conf_t;
{% endcodeblock %}
</div>
</div>
</div>
</div>

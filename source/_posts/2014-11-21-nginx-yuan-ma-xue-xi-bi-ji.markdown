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
<li><a href="#sec-1">1 配置, error日志和请求上下文</a>
<ul>
<li><a href="#sec-1-1">1.1 使用http配置</a>
<ul>
<li><a href="#sec-1-1-1">1.1.1 分配用于保存配置参数的数据结构</a></li>
<li><a href="#sec-1-1-2">1.1.2 设定配置项的解析方式</a></li>
<li><a href="#sec-1-1-3">1.1.3 自定义配置项处理方法</a></li>
<li><a href="#sec-1-1-4">1.1.4 合并配置项</a></li>
<li><a href="#sec-1-1-5">1.1.5 http 配置模型</a></li>
<li><a href="#sec-1-1-6">1.1.6 解析http配置的流程</a></li>
</ul></li>
</ul>
</li>
<li><a href="#sec-2">2 error 日志的用法</a></li>
<li><a href="#sec-3">3 请求的上下文</a>
<ul>
<li><a href="#sec-3-1">3.1 上下文的使用</a></li>
<li><a href="#sec-3-2">3.2 http框架如何维护上下文结构</a></li>
</ul>
</li>
<li><a href="#sec-4">4 访问第三方服务</a>
<ul>
<li><a href="#sec-4-1">4.1 upstream的使用</a>
<ul>
<li><a href="#sec-4-1-1">4.1.1 ngx_http_upstream_t</a></li>
<li><a href="#sec-4-1-2">4.1.2 设置upstream的限制性参数</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">配置, error日志和请求上下文</h2>
<div class="outline-text-2" id="text-1">


</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">使用http配置</h3>
<div class="outline-text-3" id="text-1-1">


</div>

<div id="outline-container-1-1-1" class="outline-4">
<h4 id="sec-1-1-1">分配用于保存配置参数的数据结构</h4>
<div class="outline-text-4" id="text-1-1-1">


{% codeblock lang:c %}
    typedef struct {
        ngx_str_t my_str;
        ngx_str_t       my_str;
        ngx_int_t       my_num;
        ngx_flag_t      my_flag;
        size_t      my_size;
        ngx_array_t*    my_str_array;
        ngx_array_t*    my_keyval;
        off_t       my_off;
        ngx_msec_t      my_msec;
        time_t      my_sec;
        ngx_bufs_t      my_bufs;
        ngx_uint_t      my_enum_seq;
        ngx_uint_t  my_bitmask;
        ngx_uint_t      my_access;
        ngx_path_t* my_path;
         
        ngx_str_t       my_config_str;
        ngx_int_t       my_config_num;
    } ngx_http_mytest_conf_t;
{% endcodeblock %}
<p>
    当http框架在解析nginx.conf文件时只要遇到http{}, server{}和location{}配置块
    都会立刻分配一个新的ngx_http_mytest_conf_t结构体。因此http模块感兴趣的配置
    项需要统一地使用一个struct结构体来保存, 否则http框架无法管理，如果nginx.conf文件
    中在http{}下有多个server{}/location{}，那么这个struct结构体在nginx进程中就会存在
    多份实例。
</p>
<p>    
    nginx是通过ngx_http_module_t中的回调方法与结构体ngx_http_mytest_conf_t
    关联。ngx_http_module_t结构体定义如下:
{% codeblock lang:c %}
    typedef struct {
     ngx_int_t   (*preconfiguration)(ngx_conf_t *cf);
     ngx_int_t   (*postconfiguration)(ngx_conf_t *cf);
  
     void       *(*create_main_conf)(ngx_conf_t *cf);
     char       *(*init_main_conf)(ngx_conf_t *cf, void *conf);
  
     void       *(*create_srv_conf)(ngx_conf_t *cf);
     char       *(*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf);
  
     void       *(*create_loc_conf)(ngx_conf_t *cf);

     // 会把所属父配置块的配置项与子配置块的同名配置项合并
     char       *(*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf);
    } ngx_http_module_t;
{% endcodeblock %}
    create_main_conf, create_srv_conf, create_loc_conf这3个回调方法负责将我们
    分配的用于保存配置项的结构体传递给HTTP框架。
</p>
<p>    
    解析nginx.conf的流程如下:
</p>


<pre class="example">当遇到http{}配置块时， http框架会调用所有http模块可能实现的create_main_conf, 
create_srv_conf, create_loc_conf方法生成存储main级别配置参数的结构体。
在遇到server{}配置块时，会调用所有http模块的create_srv_conf, create_loc_conf
回调方法生成存储srv级别配置参数的结构体。
在遇到location{}配置块时，会再次调用create_loc_conf回调方法生成存储loc级别配置
参数的结构体。
</pre>

<p>
    例如定义一个create_loc_conf方法：
{% codeblock lang:c %}
    static void* ngx_http_mytest_create_loc_conf(ngx_conf_t *cf) {
        ngx_http_mytest_conf_t *mycf;
        mycf = (ngx_http_mytest_conf_t *)ngx_pcalloc(cf->pool, sizeof(ngx_http_mytest_conf_t));
        if (mycf == NULL) {
            return NULL;
        }
        mycf->test_flag = NGX_CONF_UNSET;
        mycf->test_num = NGX_CONF_UNSET;
        mycf->test_str_array = NGX_CONF_UNSET_PTR;
        mycf->test_keyval = NULL;
        mycf->test_off = NGX_CONF_UNSET;
        mycf->test_msec = NGX_CONF_UNSET_MSEC;
        mycf->test_sec = NGX_CONF_UNSET;
        mycf->test_size = NGX_CONF_UNSET_SIZE;
        return mycf;
    }
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-1-1-2" class="outline-4">
<h4 id="sec-1-1-2">设定配置项的解析方式</h4>
<div class="outline-text-4" id="text-1-1-2">

<p>    在读取http配置时会使用ngx_command_t结构。
{% codeblock lang:c %}
    struct ngx_command_s {
        ngx_str_t             name; // 配置项名称
        ngx_uint_t            type; // 配置项类型，指定配置项可以出现的位置，以及携带的参数个数
        // type的取值见P120

        // 出现了name中指定的配置项后，将会调用set方法处理配置项的参数
        // 处理配置项，即可以自己实现一个回调方法来处理配置项，也可以使用nginx预设的14个解析
        // 配置项方法， 各个方法的用法见P122
        char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
     
        // 在配置文件中的偏移量, 用于只是配置项所处内存的相对偏移位置， 仅在type中没有设置
        // NGX_DIRECT_CONF和NGX_MAIN_CONF时才会生效。对于http模块，conf是必须要设置的
        // 其取值见P123 表4-3， NGX_HTTP_MAIN_CONF_OFFSET, NGX_HTTP_SRV_CONF_OFFSET,
        // NGX_HTTP_LOC_CONF_OFFSET, 
        // 如果这个配置项所属的数据结构是由create_main_conf回调方法完成的，必须将conf设置
        // 为NGX_HTTP_MAIN_CONF_OFFSET, 以此类推。
        ngx_uint_t            conf;
     
        // 通常用于使用预设的解析方法解析配置项，需要与conf配置使用
        // 当前配置项在这个存储配置项的结构体中的偏移位置，该值可以由offsetof处理得到
        // 使用Nginx预设的解析配置项方法，就必须设置offset，如果是自定义的专用配置项
        // 解析方法，则可以不设置offset的值
        ngx_uint_t            offset;
     
        // 配置项读取后的处理方法， 必须是ngx_conf_post_t结构的指针
        // 如果自定义了配置项的回调方法，则post指针的用途完全由用来定义，如果不使用它，可设为NULL
        // 如果想将一些数据结构或者方法的指针传递过来，使用post也可以
        // 说明见： P125
        void                 *post;
    };
{% endcodeblock %}
    ngx_conf_post_t的定义：
{% codeblock lang:c %}
    typedef char *(*ngx_conf_post_handler_pt) (ngx_conf_t *cf,
        void *data, void *conf);
     
    typedef struct {
        ngx_conf_post_handler_pt  post_handler;
    } ngx_conf_post_t;
{% endcodeblock %}
    如果需要在解析完配置项后回调某个方法，就要实现ngx_conf_post_handler_pt方法，
    并将包含post_handler的ngx_conf_post_t结构体传给post指针。目前，该方法限制
    过多且post成员过于灵活，一般完全可以init_main_conf这样的方法统一处理解析完的配置项
</p>
</div>

</div>

<div id="outline-container-1-1-3" class="outline-4">
<h4 id="sec-1-1-3">自定义配置项处理方法</h4>
<div class="outline-text-4" id="text-1-1-3">

<p>    假设如下场景
</p>


<pre class="example">假设要处理的配置项名称是test_config, 接受1个或者2个参数，且第一个参数
类型是字符串，第2个参数必须是整型。定义结构体来存储这两个参数：
typedef struct {
    ngx_str_t my_config_str;
    ngx_int_t my_config_num;
} ngx_http_mtest_conf_t;
</pre>

<p>
    处理方法：
</p>


<pre class="example">定义ngx_command_s中set方法指针格式来定义这个配置项处理方法
static char* ngx_conf_set_myconfig(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
之后填充ngx_command_t结构体的数据：
static ngx_command_t ngx_http_mytest_command[] = {
    ...
    { ngx_string("test_myconfig"),
      NGX_HTTP_LOC_CONF | NGX_CONF_TAKE12,
      ngx_conf_set_myconfig, // 自定义的函数
      NGX_HTTP_LOC_CONF_OFFSET,
      0,
      NULL
    },
    ngx_null_command
}
</pre>



{% codeblock lang:c %}
    static char* ngx_conf_set_myconfig(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)  {
        // 参数conf就是http框架传给用户的在ngx_http_mytest_create_loc_conf回调方法中分配的结构体
        // ngx_http_mytest_conf_t
        ngx_http_mytest_conf_t *mycf = conf; 
        ngx_stsr_t* value = cf->args->elts;
        if (cf->args->nelts > 1) {
            mycf->my_config_str = value[1];
        }
        if (cf->args->nelts > 2) {
            mycf->my_config_num = ngx_atoi(value[2].data, value[2].len);
            if (mycf->my_config_num == NGX_ERROR) { // 转换失败
                return "invalid number"
            }
        }
        return NGX_CONF_OK;
    }
{% endcodeblock %}

</div>

</div>

<div id="outline-container-1-1-4" class="outline-4">
<h4 id="sec-1-1-4">合并配置项</h4>
<div class="outline-text-4" id="text-1-1-4">

<p>    此处使用merge_loc_conf方法来实现合并，该方法有3个参数， 第一个参数任然是ngx<sub>conf</sub><sub>t</sub> *cf,
    提供一些基本的数据结构，如内存池，日志等。第2个参数void *prev是指解析父配置块时生成的结构体
    第3个参数void *conf，是指出保存子配置块的结构体。
{% codeblock lang:c %}
    static char* ngx_http_mytest_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child) {
        ngx_http_mytest_conf_t *prev = (ngx_http_mytest_conf_t *)parent;
        ngx_http_mytest_conf_t *child = (ngx_http_mytest_conf_t*)child;
        // nginx预设的一种方法， nginx预设的配置项合并方法有10个，见P139
        ngx_conf_merge_str_value(conf->my_str, prev->my_str, "default"); 
        return NGX_CONF_OK;
    }
{% endcodeblock %}
    在解析server块时，传入的child参数就是当前server块的ngx<sub>http</sub><sub>mytest</sub><sub>conf</sub><sub>t结构，父配置块</sub>
    就是http{}
</p></div>

</div>

<div id="outline-container-1-1-5" class="outline-4">
<h4 id="sec-1-1-5">http 配置模型</h4>
<div class="outline-text-4" id="text-1-1-5">

<p>    P140
</p></div>

</div>

<div id="outline-container-1-1-6" class="outline-4">
<h4 id="sec-1-1-6">解析http配置的流程</h4>
<div class="outline-text-4" id="text-1-1-6">

<p>    P141
</p>


<pre class="example">发现http{}时，启动http框架，http框架会初始化所有http模块的序列号，并创建3个数组用于
存储所有http模块的create_main_conf, create_srv_conf, create_loc_conf方法返回的指针地址
并将这3个数组的地址保存到ngx_http_conf_ctx_t结构体中，调用每个http模块的create_main_conf,
create_srv_conf, create_loc_conf方法，将各http模块上述3个方法返回的地址依次保存到ngx_http_conf_ctx_t
结构体的3个数组中。调用每个http模块的preconfiguration方法（该方法返回失败，则nginx进程会终止）。

</pre>

</div>
</div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">error 日志的用法</h2>
<div class="outline-text-2" id="text-2">

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

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">请求的上下文</h2>
<div class="outline-text-2" id="text-3">

<p>  此处的上下文是指http框架为每个http请求所准备的结构体。http框架定义的
  上下文是针对于http请求的，而且一个http请求对应于每一个http模块都可以
  有一个独立上下文结构体。
</p>
</div>

<div id="outline-container-3-1" class="outline-3">
<h3 id="sec-3-1">上下文的使用</h3>
<div class="outline-text-3" id="text-3-1">

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

<div id="outline-container-3-2" class="outline-3">
<h3 id="sec-3-2">http框架如何维护上下文结构</h3>
<div class="outline-text-3" id="text-3-2">

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

<div id="outline-container-4" class="outline-2">
<h2 id="sec-4">访问第三方服务</h2>
<div class="outline-text-2" id="text-4">

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

<div id="outline-container-4-1" class="outline-3">
<h3 id="sec-4-1">upstream的使用</h3>
<div class="outline-text-3" id="text-4-1">

<p>   ngx_http_request_t结构体中有一个ngx_http_upstream_t的r成员
   启用upstream机制的示意图见P161
</p>
</div>

<div id="outline-container-4-1-1" class="outline-4">
<h4 id="sec-4-1-1">ngx_http_upstream_t</h4>
<div class="outline-text-4" id="text-4-1-1">

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

<div id="outline-container-4-1-2" class="outline-4">
<h4 id="sec-4-1-2">设置upstream的限制性参数</h4>
<div class="outline-text-4" id="text-4-1-2">


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

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
</div>
</div>

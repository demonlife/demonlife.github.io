---
layout: post
title: "nginx常用数据结构汇总"
date: 2014-12-04 17:08:30 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 配置文件中可以使用的变量</a></li>
<li><a href="#sec-2">2 整型数的封装</a></li>
<li><a href="#sec-3">3 ngx_str_t</a></li>
<li><a href="#sec-4">4 ngx_list_t</a></li>
<li><a href="#sec-5">5 ngx_table_elt_t</a></li>
<li><a href="#sec-6">6 ngx_buf_t</a></li>
<li><a href="#sec-7">7 ngx_chain_t</a></li>
<li><a href="#sec-8">8 HTTP模块的数据结构</a>
<ul>
<li><a href="#sec-8-1">8.1 ngx_module_t</a></li>
<li><a href="#sec-8-2">8.2 ngx_http_module_t</a></li>
<li><a href="#sec-8-3">8.3 ngx_command_t</a></li>
<li><a href="#sec-8-4">8.4 ngx_http_request_t</a></li>
<li><a href="#sec-8-5">8.5 ngx_http_request_t</a></li>
<li><a href="#sec-8-6">8.6 ngx_http_headers_out_t</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">配置文件中可以使用的变量</h2>
<div class="outline-text-2" id="text-1">


</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">整型数的封装</h2>
<div class="outline-text-2" id="text-2">


{% codeblock lang:c %}
  typedef intptr_t ngx_int_t; // 有符号数
  typedef uintptr_t ngx_uint_t; // 无符号数
{% endcodeblock %}
</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">ngx_str_t</h2>
<div class="outline-text-2" id="text-3">


{% codeblock lang:c %}
  typedef struct {
      size_t      len; // 字符串的有效长度, 因此使用时需要根据长度len来使用data成员
      u_char     *data; // 指向字符串起始地址
  } ngx_str_t;
{% endcodeblock %}
</div>

</div>

<div id="outline-container-4" class="outline-2">
<h2 id="sec-4">ngx_list_t</h2>
<div class="outline-text-2" id="text-4">

<p>  是一个链表容器
{% codeblock lang:c %}
  typedef struct ngx_list_part_s  ngx_list_part_t;
  struct ngx_list_part_s {
      void             *elts;
      ngx_uint_t        nelts;
      ngx_list_part_t  *next;
  };
  typedef struct {
      ngx_list_part_t  *last;
      ngx_list_part_t   part;
      size_t            size;
      ngx_uint_t        nalloc;
      ngx_pool_t       *pool;
  } ngx_list_t;
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-5" class="outline-2">
<h2 id="sec-5">ngx_table_elt_t</h2>
<div class="outline-text-2" id="text-5">

<p>  是一个key/value对
{% codeblock lang:c %}
  typedef struct {
      ngx_uint_t        hash;
      ngx_str_t         key;
      ngx_str_t         value;
      u_char           *lowcase_key;
  } ngx_table_elt_t;
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-6" class="outline-2">
<h2 id="sec-6">ngx_buf_t</h2>
<div class="outline-text-2" id="text-6">

<p>  ngx<sub>buf</sub><sub>t是nginx处理大数据的关键数据结构，</sub> 即用于内存也用于磁盘数据
{% codeblock lang:c %}
  typedef struct ngx_buf_s  ngx_buf_t;
{% endcodeblock %}
  对于HTTP模块来说，需要注意HTTP框架，事件框架是如何设置和使用pos，last等指针
  以及如何处理这些标志位的。
</p></div>

</div>

<div id="outline-container-7" class="outline-2">
<h2 id="sec-7">ngx_chain_t</h2>
<div class="outline-text-2" id="text-7">

<p>  ngx<sub>chain</sub><sub>t是与ngx</sub><sub>buf</sub><sub>t配合使用的链表数据结构</sub>
{% codeblock lang:c %}
  typedef struct ngx_chain_s       ngx_chain_t;
  struct ngx_chain_s {
      ngx_buf_t    *buf; // 指向当前的ngx_buf_t缓冲区
      ngx_chain_t  *next; // 指向下一个ngx_chain_t， 如果是最后一个需设置为NULL
  };  
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-8" class="outline-2">
<h2 id="sec-8">HTTP模块的数据结构</h2>
<div class="outline-text-2" id="text-8">


</div>

<div id="outline-container-8-1" class="outline-3">
<h3 id="sec-8-1">ngx_module_t</h3>
<div class="outline-text-3" id="text-8-1">

<p>   ngx<sub>module</sub><sub>t</sub> 是一个nginx模块的数据结构
{% codeblock lang:c %}
   typedef struct ngx_module_s      ngx_module_t;
   struct ngx_module_s {
       ...
       void *ctx; // 对于http类型的模块来说， 该指针必须指向ngx_http_module_t接口
       ngx_command_t *commands; // 用于定义模块的配置文件参数， 每一个数组元素都是
       // ngx_command_t类型， 数组的结尾用ngx_null_command表示
       ...
   }   
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-8-2" class="outline-3">
<h3 id="sec-8-2">ngx_http_module_t</h3>
<div class="outline-text-3" id="text-8-2">

<p>   HTTP框架在读取，重载配置文件时定义了由ngx<sub>http</sub><sub>module</sub><sub>t结构描述的8个阶段，</sub>
   在启动过程中会在每个阶段调用ngx<sub>http</sub><sub>module</sub><sub>t中相应的方法。</sub>
</p></div>

</div>

<div id="outline-container-8-3" class="outline-3">
<h3 id="sec-8-3">ngx_command_t</h3>
<div class="outline-text-3" id="text-8-3">


{% codeblock lang:c %}
   typedef struct ngx_command_s ngx_command_t;
   struct ngx_command_s {
       ngx_str_t             name;
       ngx_uint_t            type;
       char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
       ngx_uint_t            conf;
       ngx_uint_t            offset;
       void                 *post;
   };   
{% endcodeblock %}
</div>

</div>

<div id="outline-container-8-4" class="outline-3">
<h3 id="sec-8-4">ngx_http_request_t</h3>
<div class="outline-text-3" id="text-8-4">

<p>   请求的所有信息都可以在传入的ngx<sub>http</sub><sub>request</sub><sub>t类型参数r中取得</sub>
   typedef struct ngx<sub>http</sub><sub>request</sub><sub>s</sub> ngx<sub>http</sub><sub>request</sub><sub>t</sub>;
   在对一个用户请求进行解析时， 可得到下列4类信息：
   1.方法名
     method的类型是ngx<sub>uint</sub><sub>t，其取值在文件：</sub>
     code/nginx-1.6.2.source/src/http/ngx<sub>http</sub><sub>request</sub>.h::27
     中, 当需要了解请求中的http方法时， 应该使用r-&gt;method与上述
     文件中定义的15个宏进行比较，当然也可以使用r-&gt;method<sub>name与字符串比较</sub>
     只是效率会差些。P96
</p><ol>
<li>URI
</li>
<li>URL 参数
</li>
<li>协议版本
</li>
</ol>

</div>

</div>

<div id="outline-container-8-5" class="outline-3">
<h3 id="sec-8-5">ngx_http_request_t</h3>
<div class="outline-text-3" id="text-8-5">

<p>   在ngx<sub>http</sub><sub>request</sub><sub>t</sub> 中可以取到请求中的http头部信息
</p></div>

</div>

<div id="outline-container-8-6" class="outline-3">
<h3 id="sec-8-6">ngx_http_headers_out_t</h3>
<div class="outline-text-3" id="text-8-6">

</div>
</div>
</div>

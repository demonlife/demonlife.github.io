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
<li><a href="#sec-2">2 整型数的封装</a>
<ul>
<li><a href="#sec-2-1">2.1 intptr_t</a></li>
</ul>
</li>
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
<p>
  其中intptr_t的定义可以在/usr/include/stdint.h文件中找到定义
</p>


<pre class="example">注意：
定义intptr_t的原因是: 
尽管在混合不同数据类型时你必须小心, 有时有很好的理由这样做. 一种情况是因为内存存取, 与内核相关时是特殊的. 
概念上, 尽管地址是指针, 内存管理常常使用一个无符号的整数类型更好地完成; 内核对待物理内存如同一个大数组, 
并且内存地址只是一个数组索引. 进一步地, 一个指针容易解引用; 当直接处理内存存取时, 你几乎从不想以这种方式解引用. 
使用一个整数类型避免了这种解引用, 因此避免了 bug. 因此, 内核中通常的内存地址常常是 unsigned long, 
利用了指针和长整型一直是相同大小的这个事实, 至少在 Linux 目前支持的所有平台上.
因为其所值的原因, C99 标准定义了 intptr_t 和 uintptr_t 类型给一个可以持有一个指针值的整型变量. 
但是, 这些类型几乎没在 2.6 内核中使用

以上引用自: http://oss.org.cn/kernel-book/ldd3/ch11.html
</pre>


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">intptr_t</h3>
<div class="outline-text-3" id="text-2-1">

<p>   /usr/include/stdint.h 文件中定义了一些与平台无关的数据类型
{% codeblock lang:c %}
   /* Types for `void *' pointers.  */
   #if __WORDSIZE == 64
   # ifndef __intptr_t_defined
   typedef long int     intptr_t;
   #  define __intptr_t_defined
   # endif
   typedef unsigned long int    uintptr_t;
   #else
   # ifndef __intptr_t_defined
   typedef int          intptr_t;
   #  define __intptr_t_defined
   # endif
   typedef unsigned int     uintptr_t;
   #endif
{% endcodeblock %}
   从定义可以看出，intptr<sub>t在不同的平台是不一样的，始终与地址位数相同，因此用来存放地址，即地址</sub>
   使用int时也可以使用intptr<sub>t来保证平台的通用性，它在不同的平台上编译时长度不同，但都是标准的平台字长，</sub>
   比如64位机器它的长度就是8字节，32位机器它的长度是4字节，使用它可以安全地进行整数与指针的转换运算，
   也就是说当需要将指针作为整数运算时，将它转换成intptr<sub>t进行运算才是安全的。</sub>
</p></div>
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
<p>
  该结构在nginx的领域中，就是当做字符串来使用的，该字符串可以用来存储含有\0的字符串。
  使用示例:
{% codeblock lang:c %}
  if (0 == ngx_strncmp(r->method_name.data, "PUT", r->method_name.len)) {
      //...
  }
  // ngx_strncmp其实就是strncmp函数， 为了跨平台nginx对其进行了名称上的封装
  #define ngx_strncmp(s1, s2, n) strncmp((const char*)s1, (const char*)s2, n)
{% endcodeblock %}
  使用ngx<sub>str</sub><sub>t可以有效的降低内存使用量，例如：用户请求</sub>"GET /test?a=1 http/1.1\r\n"存储到地址
  0x1d0b0110上， 这时只需要将r-&gt;method<sub>name设置为</sub>{len=3, data=0x1d0b0110}就可以表示方法名GET，
  而不再需要单独为method<sub>name再分配内存。</sub>
</p></div>

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
</p>
<p>  
  使用示例:
{% codeblock lang:c %}
  // 创建一个链表
  // ngx_list_create被调用后至少会创建一个数组，即part成员。
  // 创建一个可以容纳4个ngx_str_t元素的链表
  ngx_list_t *testlist = ngx_list_create(r->pool, 4, sizeof(ngx_str_t));
  if (testlist == NULL) {
      return NGX_ERROR;
  }

  // 添加元素， 传人的参数是ngx_list_t链表，正常返回新分配的元素首地址，如果返回NULL，表示添加
  // 失败，在使用时，调用ngx_list_push得到返回元素地址，再对返回地址进行赋值。
  ngx_str_t *str = ngx_list_push(testlist);
  if (str == NULL) {
      return NGX_ERROR;
  }
  str->len = sizeof("Hello world");
  str->val = "Hello world";

  // 遍历链表
  ngx_list_part_t *part = &testlist.part;
  // 根据链表中的数据类型， 把数组里的elts转化为该类型使用
  ngx_str_t *str = part->elts;
  for (i = 0; /* void */; ++i) {
      if (i >= part->nelts) {
          if (part->next == NULL) {
              // 如果某个ngx_list_part_t数组的next指针为空，则说明遍历已经完成
              break;
          }
          // 访问下一个ngx_list_part_t
          part = part->next;
          str = part->elts;
          // 将i置为0，准备重新访问下一个数组
          i = 0;
      }
      // 访问当前节点的内容
      printf("list element: %*s\n", str[i].len, str[i].data);
  }
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
  ngx<sub>table</sub><sub>elt</sub><sub>t主要是为http头部“量身定制”的，hash用于快速检索头部</sub>
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
  在向用户发送HTTP包体时，就要传人ngx<sub>chain</sub><sub>t链表对象，如果是最后一个ngx</sub><sub>chain</sub><sub>t，必须</sub>
  将next设置为NULL， 否则永远不会发送成功，而且这个请求将一直不会结束
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

---
layout: post
title: "nginx 模块开发与架构解析之helloworld模块"
date: 2014-11-26 13:51:52 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 http模块的调用</a>
<ul>
<li><a href="#sec-1-1">1.1 nginx使用的数据类型(P72)</a></li>
<li><a href="#sec-1-2">1.2 HTTP模块的数据结构</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1 处理用户请求</a></li>
<li><a href="#sec-1-2-2">1.2.2 获取http包体</a></li>
<li><a href="#sec-1-2-3">1.2.3 发送响应</a></li>
</ul>
</li>
<li><a href="#sec-1-3">1.3 将磁盘文件作为包体发送</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">http模块的调用</h2>
<div class="outline-text-2" id="text-1">

<p>  worker进程会在一个for循环语句里反复调用事件模块检测网络事件， 当事件模块检测到
  某个客户端发起的TCP请求时，会建立tcp连接，成功建立连接后根据nginx.conf文件中的
  配置交由http框架处理。http框架会试图接受完整的http头部，并在接受到完整的http
  头部后将请求分发到具体的http模块中处理。
</p>
</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">nginx使用的数据类型(P72)</h3>
<div class="outline-text-3" id="text-1-1">

</div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">HTTP模块的数据结构</h3>
<div class="outline-text-3" id="text-1-2">


</div>

<div id="outline-container-1-2-1" class="outline-4">
<h4 id="sec-1-2-1">处理用户请求</h4>
<div class="outline-text-4" id="text-1-2-1">

<p>    在出现mytest配置项时， ngx_http_mytest方法会被调用，
    这时将ngx_http_core_loc_conf_t
    结构的handler成员指定为ngx_http_mytest_handler, 当http框架接受完http请求的头部
    后，会调用handler指向的方法。
</p><ul>
<li>方法的返回值
      方法的返回值取值定义在 src/http/ngx_http_request.h文件中
      在ngx<sub>http</sub><sub>mytest</sub><sub>handler的返回值中，如果是正常的HTTP返回码</sub>, nginx会按照规范构
      造合法的响应包发送给用户。当返回的是错误码时，会构造响应错误的返回值。
      在处理方法中除了返回HTTP响应码外，还可以返回Nginx全局定义的几个错误码。
      这些错误码对应Nginx自身提供的大部分方法来说都是通用的。
</li>
<li>nginx定义的方法名
      nginx中method的类型是ngx<sub>uint</sub><sub>t，是Nginx忽略大小写等情形时解析完用户请求后得到的</sub>
      方法类型，当需要了解用户请求中的http方法时， 应该使用r-&gt;method整型与nginx定义
      的15个宏进行比较，这样速度是最快的
</li>
<li>获取非RFC2616标准的HTTP头部(P100)
      需要遍历r-&gt;headers<sub>in</sub>.headers链表才能获得
</li>
</ul>

</div>

</div>

<div id="outline-container-1-2-2" class="outline-4">
<h4 id="sec-1-2-2">获取http包体</h4>
<div class="outline-text-4" id="text-1-2-2">

<p>    ngx<sub>http</sub><sub>read</sub><sub>client</sub><sub>request</sub><sub>body方法是一个异步方法，用于接收http包体</sub>
    包体接收完毕后会回调方法，该方法的原型如下:
{% codeblock lang:c %}
    typedef void (*ngx_http_client_body_handler_pt)(ngx_http_request_t *r);
{% endcodeblock %}
    也可以使用ngx_http_discard_request_body丢弃包体。
</p></div>

</div>

<div id="outline-container-1-2-3" class="outline-4">
<h4 id="sec-1-2-3">发送响应</h4>
<div class="outline-text-4" id="text-1-2-3">

<p>    请求处理完毕后，需要向用户发送HTTP响应，告知客户端Nginx的执行结果。HTTP响应主要包括
    响应行，响应头部，包体三部分。发送http响应时需要执行发送HTTP头部和发送HTTP包体两步
    操作。
</p><ul>
<li>发送http头部
</li>
</ul>


{% codeblock lang:c %}
      ngx_int_T ngx_http_send_header(ngx_http_request_t *r);
{% endcodeblock %}
<p>
      向headers链表中添加自定义的HTTP头部(P104)
</p><ul>
<li>将内存中的字符串作为包体发送(P104)
</li>
</ul>


{% codeblock lang:c %}
      ngx_int_t ngx_http_output_filter(ngx_http_request_t *r, ngx_chain_t *in);
{% endcodeblock %}

</div>
</div>

</div>

<div id="outline-container-1-3" class="outline-3">
<h3 id="sec-1-3">将磁盘文件作为包体发送</h3>
<div class="outline-text-3" id="text-1-3">

<p>   发送磁盘中的文件内容也是使用上面介绍的方法。不同之处在于设置ngx_buf_t缓冲区.
   将其中的in_file设置为1就表示这次ngx_buf_t缓冲区发送的是文件而不是内存.
   当nginx检测到in_file为1时， 会从ngx_buf_t缓冲区中的file成员处获取实际的文件。
   file的类型是ngx<sub>file</sub><sub>t</sub>.
   nginx简单封装了一个宏用来代替open系统调用,
{% codeblock lang:c %}
   // 对于打开标志位， Nginx也定义了几个宏来加以封装
   #define ngx_open_file(name, mode, create, access) \
       open((const char*)name, mode|create, access)
{% endcodeblock %}
</p><ul>
<li>清理文件句柄
     Nginx会异步地将整个文件高效的发送给用户，要求HTTP框架在响应发送完毕后关闭已经
     打开的文件句柄，否则出现句柄泄露问题。设置清理文件句柄时，需要定义一个ngx_pool_cleanup_t
     结构体，将得到的文件句柄等信息赋给它，并将nginx提供的ngx<sub>pool</sub><sub>cleanup</sub><sub>file函数设置到它的</sub>
     handler回调方法中即可。
</li>
<li>关闭文件句柄的函数
     ngx<sub>pool</sub><sub>cleanup</sub><sub>file</sub>: 将文件句柄关闭， 需要一个ngx<sub>pool</sub><sub>cleanup</sub><sub>file</sub><sub>t类型的参数</sub>
     在ngx<sub>pool</sub><sub>cleanup</sub><sub>t结构体的data成员上赋值即可。</sub>
</li>
</ul>


</div>
</div>
</div>

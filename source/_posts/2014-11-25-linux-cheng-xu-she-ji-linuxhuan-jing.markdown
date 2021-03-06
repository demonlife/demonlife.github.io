---
layout: post
title: "Linux 程序设计-Linux环境"
date: 2014-11-25 08:23:31 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 程序参数</a>
<ul>
<li><a href="#sec-1-1">1.1 getopt</a></li>
<li><a href="#sec-1-2">1.2 getopt_long</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">程序参数</h2>
<div class="outline-text-2" id="text-1">


</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">getopt</h3>
<div class="outline-text-3" id="text-1-1">

<p>   该函数用于处理命令行参数，支持需要关联值和不需要关联值的选项。
</p>


<pre class="example">#include &lt;unistd.h&gt;
int getopt(int argc, char* const argv[], const char* optstring);
extern char *optarg;
extern int optind, opterr, optopt;
</pre>

<p>
   getopt函数将传递给程序的main函数的argc和argv作为参数， 同时接受一个选项
   指定字符串optstring，用于告诉getopt哪些选项可用，以及他们是否有关联值。
   关于optstring的说明：
</p>


<pre class="example">optstring只是一个字符串列表，每个字符代表一个单字符选项。如果一个字符后
紧跟一个冒号，则表面该选项有一个关联值作为下一个参数。例如
getopt(argc, argv, "if:lr")表示支持-i, -l, -r, -f, 其中-f后要紧跟一个参数
循环调用getopt可以依次得到每个选项。
</pre>

<p>
   getopt有如下的行为：
</p>


<pre class="example">如果选项有一个关联值，外部变量optarg指向这个值。
处理完选项返回-1， 特殊参数--将使函数停止扫描选项
遇到无法识别的选项，返回一个问号，并将值保存到optopt中。
如果选项有一个关联值，但用户未提供该值，通常会返回一个问号，如果将选项字符串
的第一个字符设置为冒号，则在未提供值的情况下返回冒号。
外部变量optind被设置为下一个待处理参数的索引。当所有参数都处理完毕后， optind
将指向argv数组尾部可以找到其余参数的位置。
</pre>

<p>
   注意： 有些版本的getopt会在第一个非选项参数处停下来， 返回-1并设置optind的值
   而其他一些版本，如Linux提供的版本，能够处理出现在程序参数中任意的选项。在这种
   情况下，getopt实际上重写了argv数组，把所有非选项参数都集中在一起，从argv[optind]
   位置开始，对GNU版本的getopt而言，这一行为是由环境变量POSIXLY<sub>CORRECT控制的，如果</sub>
   被设置，getopt就会在第一个非选项参数处停下来。还有一些getopt版本会在遇到未知选项
   时打印错误信息。根据POSIX规范的规定，如果opterr变量是非零值，getopt就会向stderr
   打印一条出错信息。
</p></div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">getopt_long</h3>
<div class="outline-text-3" id="text-1-2">

<p>   接受以&ndash;开始的长参数
</p>


<pre class="example">getopt_long比getopt多两个参数，第一个附加参数是一个结构数组，描述了每个长选项
并告诉getopt_long如何处理他们。第二个附加参数是一个变量指针，可以作为optind的
长选项版本使用，对于识别的长选项，在长选项数组中的索引就写入该变量。
长选项数组由一些类型为struct option的结构组成，每一个结构描述了一个长选项的行为
该数组必须一个包含全0的结构结尾。
长选项在头文件getopt.h中定义，并且该头文件必须与常量_GNU_SOURCE一同包含进来，
该常量启用getopt_long功能。
</pre>



{% codeblock lang:c %}
   struct option {
       const char *name; // 长选项的名字，缩写也可以接受，只要不与其他选项混淆
       int has_arg; // 该选项是否带参数，0：不带，1：必须有一个参数， 2：有一个可选参数
       //设置为NULL表示当找到该选项时，返回val里给出的值，否则返回0，并将val的值写入flag指向的变量
       int *flag; 
       int val; // getopt_long为该选项返回的值, 相当于短参数
   }
{% endcodeblock %}
</div>
</div>
</div>

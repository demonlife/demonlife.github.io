---
layout: post
title: "python的小技巧"
date: 2014-12-08 08:16:49 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 分解序列</a></li>
<li><a href="#sec-2">2 命名切片</a></li>
<li><a href="#sec-3">3 常用模块</a></li>
<li><a href="#sec-4">4 内置函数</a>
<ul>
<li><a href="#sec-4-1">4.1 filter</a></li>
<li><a href="#sec-4-2">4.2 map</a></li>
<li><a href="#sec-4-3">4.3 reduce</a></li>
</ul>
</li>
<li><a href="#sec-5">5 进制之间的转换</a>
<ul>
<li><a href="#sec-5-1">5.1 其他进制转十进制</a></li>
<li><a href="#sec-5-2">5.2 十进制转其他进制</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">分解序列</h2>
<div class="outline-text-2" id="text-1">


{% codeblock lang:python %}
  p = (4, 5)
  x, y = p
   
  data = ['acme', 50, 91.1, (2012, 12, 11)]
  name, shares, price, date = data
   
  s = 'hello'
  a, b, c, c, e = s
   
  data = ['acme', 50, 91.1, (2012, 12, 11)]
  _, shares, price, _ = data #忽略第一个和最后一个
   
  first, *middle, last = data #middle 存储除第一个和最后一个的数据
  name, *_, last = data, #只取第一个和最后一个，其他的丢弃
{% endcodeblock %}
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">命名切片</h2>
<div class="outline-text-2" id="text-2">

<p>  使用内置的slice()函数创建有一个切片对象。
{% codeblock lang:python %}
  items = [0, 1, 2, 3]
  a = slice(2, 4)
  # 打印slice对象的信息
  print a.start, a.stop, a.step
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">常用模块</h2>
<div class="outline-text-2" id="text-3">

<p>  os: 含有对文件的一些操作。
  sys:  
</p></div>

</div>

<div id="outline-container-4" class="outline-2">
<h2 id="sec-4">内置函数</h2>
<div class="outline-text-2" id="text-4">

<p>  Python是将复杂的数据结构隐藏在内置函数中，用C语言来实现
</p>
</div>

<div id="outline-container-4-1" class="outline-3">
<h3 id="sec-4-1">filter</h3>
<div class="outline-text-3" id="text-4-1">

<p>   相当于一个迭代器，调用一个布尔函数func来迭代seq中的每个元素，
   返回一个是bool<sub>seq返回为True的序列</sub>
   参数说明:
   如果第一个参数为function,那么返回条件为真的序列(列表，元祖或字符串),
   如果第一个参数为None的话,那么返回序列中所有为True的项目
{% codeblock lang:python %}
   print filter(None, [-2, 0, 2, '', {}, ()]) #输出[-2, 2]
   print filter(lambda x: x!='a', 'abcd') #输出bcd
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-4-2" class="outline-3">
<h3 id="sec-4-2">map</h3>
<div class="outline-text-3" id="text-4-2">

<p>   对一个及多个序列执行同一个操作，返回列表
   说明：
   返回一个列表，该列表是参数func对seq1， seq2处理的结果
   可以有多个序列，如果函数为None的话，返回一个序列的列表
{% codeblock lang:python %}
   map(lambda x: x+1, [1,2,3,4]) #[2,3,4,5]
   map(None, [1, 2, 3, 4], (1, 2)) #[(1,1), (2, 2), (3, None), (4, None)]
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-4-3" class="outline-3">
<h3 id="sec-4-3">reduce</h3>
<div class="outline-text-3" id="text-4-3">

</div>
</div>

</div>

<div id="outline-container-5" class="outline-2">
<h2 id="sec-5">进制之间的转换</h2>
<div class="outline-text-2" id="text-5">


</div>

<div id="outline-container-5-1" class="outline-3">
<h3 id="sec-5-1">其他进制转十进制</h3>
<div class="outline-text-3" id="text-5-1">


{% codeblock lang:python %}
   int('110', 2) # 将二进制转换为十进制
   int('04', 8) # 将八进制转换为十进制
   int('0x4', 16) # 将16进制转换为十进制
{% endcodeblock %}
</div>

</div>

<div id="outline-container-5-2" class="outline-3">
<h3 id="sec-5-2">十进制转其他进制</h3>
<div class="outline-text-3" id="text-5-2">

<p>   bin(): 将十进制转换为二进制
   oct(): 将十进制转换为二进制
   hex(): 将十进制转换为十六进制
</p></div>
</div>
</div>

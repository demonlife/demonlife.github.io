---
layout: post
title: "awk 教程"
date: 2014-12-02 16:56:12 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 awk 练习</a></li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">awk 练习</h2>
<div class="outline-text-2" id="text-1">

<ul>
<li>文本处理
    题目如下



<pre class="example">如果第1行和第2行都不以;开头，则合并这两行为新行，并继续处理新行和第3行；
如果第1行以;开头，则继续处理第2行和第3行。
以上流程仅为方便描述，只要能达到相同效果即可。
如输入为：
;a
;;b
c
d;
e;;;
;f
g
期望输出：
;a
;;b
cd;e;;;
;f
g
</pre>

<p>
    解决方法
</p>


<pre class="example">awk '!/^;/{a=a$0}/^;/{if(a!="")print a;print $0;a="";}END{if(a!="")print a;}'
解析：
awk语法 test1{statements1}test2{statements2}...
针对每一行，如果满足test1，则执行statement1，如果满足test2，则执行statement2 ...
所以分3部分：
!/^;/{a=a$0} 如果不以;开头，则将当前行追加到临时变量a(作为缓冲区)中
!/^;/
    ! 否定
    /.../ 表示测试当前行是否满足给定正则表达式
    ^; 正则表达式，表示以;开头
a=a$0
    a 变量，无需声明，直接使用，默认值是0、null、""，根据使用场景自动转换，这里第一次用就是空字符串
    $0 表示整个一行的内容
    a$0 两个字符串写在一起，表示字符串拼接
/^;/{if(a!="")print a;print $0;a="";} 如果以;开头，先输出临时拼接的变量a(若有)，再输出当前行
    /^;/ 判断当前行是否以;开头
if(a!="")print a;print $0;a="";
    if(a!="")print a; 如果a不为空，则输出a的值(print自动换行)
    print $0; 打印当前行
    a=""; 清空a的值，以备下次使用
END{if(a!="")print a;} 处理完所有行，最后再判断缓冲区a中是否有内容，若有，则打印
    如果最后几行都不以;开头，会全部追加到a中，一直没有机会输出出来，因为碰到;开头的行才会输出
    END条件表示处理完最后一行之后（相对的当然有BEGIN，表示处理第一行之前）
</pre>

</li>
</ul>

</div>
</div>

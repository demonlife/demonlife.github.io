---
layout: post
title: "C语言学习之小提示"
date: 2014-12-08 11:37:25 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 continue的注意事项</a></li>
<li><a href="#sec-2">2 宏展开</a></li>
<li><a href="#sec-3">3 浮点数</a></li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">continue的注意事项</h2>
<div class="outline-text-2" id="text-1">

<p>  For the for loop, continue causes the conditional test and increment 
  portions of the loop to execute. 
  For the while and do&hellip;while loops, program control passes to the conditional tests.
  for 循环遇到 continue 会执行for 小括号内的第三个语句。
  while 和 do&hellip;while 则会跳到循环判断的地方。
  例如：
{% codeblock lang:c %}
  #include <stdio.h>

  int main() {
      int i = 1;
      do {
          printf("%d\n", i);
          i++;
          if (i < 15) {
              continue;
          }
      }while(0);
      return 0;
  }
{% endcodeblock %}
</p></div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">宏展开</h2>
<div class="outline-text-2" id="text-2">

<p>  Macro arguments are completely macro-expanded before they are substituted(代替) into 
  a macro body, unless they are stringified or pasted with other tokens. 
  After substitution, the entire macro body, including the substituted arguments, is scanned 
  again for macros to be expanded. 
  The result is that the arguments are scanned twice to expand macro calls in them.
  简单的说就是 宏会扫描一遍，把可以展开的展开，展开一次后会再扫描一次看又没有可以展开的宏。
  例如：
{% codeblock lang:c %}
  #include <stdio.h>

  #define f(a, b) a##b
  #define g(a) #a
  #define h(a) g(a)

  int main() {
      printf("%s\n", h(f(1, 2)));
      printf("%s\n", g(f(1, 2)));
      return 0;
  }
{% endcodeblock %}
</p><ul>
<li>分析:
    对于第一个宏展开，过程如下:
    h(f(1,2)) 读到此处发现h是一个宏，进行替换
    g(f(1,2)) 读到此处发现f是一个宏，进行替换
    g(12) 读到此处，无宏定义，返回
    g(12) 又发现宏， 展开
    "12" 无宏， 改行宏展开完毕

<p>    
    对于第二个宏展开，过程如下:
    g(f(1, 2)) 读到此处发现g是一个宏，进行替换
    "f(1,2)" 读到此处发现是字符串， 宏展开完成
</p></li>
</ul>

</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">浮点数</h2>
<div class="outline-text-2" id="text-3">


{% codeblock lang:c %}
  #include <stdio.h>

  int main() {
      float a = 12.5;
      printf("%d\n", a);
      printf("%d\n", *(int*)&a);
      return 0;
  }
{% endcodeblock %}
<ul>
<li>解释
    int和float在32位机器上都是4字节的。
    对于一个浮点数，可以表示为(-1)<sup>S</sup> * X *2<sup>Y</sup>
    其中S是符号，使用1位表示
    X是一个二进制在[1,2)内的小数，一般称为尾数，用23位表示， 一般不表示小数前的1
    Y是一个整数，代表幂，一般称为阶码，用8位表示， Y也有符号问题，小于127的数是负数，大于的是整数，
    如果阶码为0，为数x就不在加1了。
    float在内存中的位序如下:
    1位符号位8位阶码23位尾数
</li>
</ul>



</div>
</div>

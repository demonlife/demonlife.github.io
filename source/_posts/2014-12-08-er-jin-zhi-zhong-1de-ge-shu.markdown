---
layout: post
title: "二进制中1的个数"
date: 2014-12-08 14:10:44 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 二进制中1的个数</a></li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">二进制中1的个数</h2>
<div class="outline-text-2" id="text-1">

<p>  详细讲解: <a href="http://github.tiankonguse.com/blog/2014/11/16/bit-count-more/">http://github.tiankonguse.com/blog/2014/11/16/bit-count-more/</a>
  假设是32位无符号整数
  算法1:
{% codeblock lang:c %}
  unsigned countbits(unsigend x) {
      unsigned n = 0;
      while(n+=(x&1), x >>= 1);
      return n;
  }
{% endcodeblock %}
  算法2:
  原理是x &amp; (x - 1)可以把x的最低位1置换成0
{% codeblock lang:c %}
  unsigned countbits(unsigned x) {
      unsigned n = 0;
      while(x && (++n, x &= x - 1));
      return n;
  }
{% endcodeblock %}
  算法3: 分治法
  思路:
  首先将32位数看成相互独立的32个 0-1 数字. 
  然后两两相加, 这样我们就有16个数字了. 
  然后再两两相加, 就变成8个数字了, 接着再两两相加变成4个, 然后两个, 最后一个数字. 
  由于是32位数字的和, 所以答案刚好就是1的个数啦
{% codeblock lang:c %}
  // 该段程序只对32位有效
  unsigned countbits(unsigned x) {
      unsigned mask[] = {0x55555555, 0x33333333, 0x0F0F0F0F,
                         0x00FF00FF, 0x0000FFFF};
      for(unsigned i=0, j=1; i < 5; ++i, j <<= 1) {
          x = (x & mask[i]) + ((x>>j) & mask[i]);
      }
      return x;
  }
{% endcodeblock %}
  算法4: 打表法
{% codeblock lang:c %}
  const int OneNumMax = 1 << 16;
  int OneNum[OneNumMax];
  int init() {
      int i = 0;
      for(i = 1; i < OneNumMax; ++i) {
          OneNum[i] = OneNum[i >> 1] + (i & 1);
      }
      return 0;
  }

  unsigned countbits(unsigned x) {
      static int a = init();
      return OneNum[x >> 16] + OneNum[x & ((1<<16) - 1)];
  }
{% endcodeblock %}
  算法5:
{% codeblock lang:c %}
  int bitCount ( unsigned n ) {
      unsigned tmp = n - ((n >> 1) & 033333333333) - ((n >> 2) & 011111111111);
      return ( (tmp + (tmp >> 3) ) & 030707070707) % 63;
  }
{% endcodeblock %}
</p></div>
</div>

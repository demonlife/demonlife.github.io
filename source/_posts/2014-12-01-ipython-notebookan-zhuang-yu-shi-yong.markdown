---
layout: post
title: "IPython notebook安装与使用"
date: 2014-12-01 15:30:48 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 安装</a>
<ul>
<li><a href="#sec-1-1">1.1 python的开发组件</a></li>
<li><a href="#sec-1-2">1.2 pip安装python插件</a></li>
<li><a href="#sec-1-3">1.3 ipython notebook安装</a></li>
<li><a href="#sec-1-4">1.4 启动</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">安装</h2>
<div class="outline-text-2" id="text-1">


</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">python的开发组件</h3>
<div class="outline-text-3" id="text-1-1">




<pre class="example">sudo apt-get install build-essential
sudo apt-get install python-dev   
</pre>

</div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">pip安装python插件</h3>
<div class="outline-text-3" id="text-1-2">




<pre class="example">NumPy、SciPy：處理科學數據用的算式
matplotlib：繪畫圖表用
SymPy：符號算術處理套件
scikits-learn：機器學習用
pandas：處理數據套件
NetworkX：處理網絡數據結構用
</pre>

</div>

</div>

<div id="outline-container-1-3" class="outline-3">
<h3 id="sec-1-3">ipython notebook安装</h3>
<div class="outline-text-3" id="text-1-3">




<pre class="example">sudo pip install "ipython[all]"
</pre>

</div>

</div>

<div id="outline-container-1-4" class="outline-3">
<h3 id="sec-1-4">启动</h3>
<div class="outline-text-3" id="text-1-4">




<pre class="example">ipython notebook --ip='*' --pylab inline 
</pre>

</div>
</div>
</div>

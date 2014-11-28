---
layout: post
title: "Linux 下的帮助工具"
date: 2014-11-28 21:15:27 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 cheat 命令</a>
<ul>
<li><a href="#sec-1-1">1.1 cheat安装</a></li>
<li><a href="#sec-1-2">1.2 配置cheat</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">cheat 命令</h2>
<div class="outline-text-2" id="text-1">

<p>  cheat命令那个依赖python和pip，因此需要先安装python和pip
</p>
</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">cheat安装</h3>
<div class="outline-text-3" id="text-1-1">




<pre class="example">pip install docopt pygments
从github下载源码: https://github.com/chrisallenlane/cheat.git
python setup.py install #进入cheat目录，安装cheat
</pre>

</div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">配置cheat</h3>
<div class="outline-text-3" id="text-1-2">

<ul>
<li>为cheat命令添加自动补全功能
     可以使用源码中的cheat/autocompletion/目录中的相应的shell的配置文件
     bash自动补全方法：
     mv &lt;cheat-source&gt;/cheat/autocompletion/cheat.bash <i>etc/bash<sub>completion</sub>.d</i>
</li>
<li>添加自定义的命令解释
     cheat -e my-cmd: 会使用EDITOR指定的编辑器编辑
</li>
<li>高亮显示
     export CHEATCOLORS=true
</li>
</ul>


</div>
</div>
</div>

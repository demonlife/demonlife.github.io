---
layout: post
title: "virtualenv 使用方法"
date: 2014-11-24 11:45:20 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 python环境搭建</a>
<ul>
<li><a href="#sec-1-1">1.1 ubuntu 系统下的搭建</a></li>
</ul>
</li>
<li><a href="#sec-2">2 virtualenv的安装</a>
<ul>
<li><a href="#sec-2-1">2.1 virtualenv的安装</a></li>
<li><a href="#sec-2-2">2.2 virutalenv使用方法</a></li>
<li><a href="#sec-2-3">2.3 最佳实践</a></li>
<li><a href="#sec-2-4">2.4 virtualenv的bootstrap脚本</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">python环境搭建</h2>
<div class="outline-text-2" id="text-1">


</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">ubuntu 系统下的搭建</h3>
<div class="outline-text-3" id="text-1-1">

<ul>
<li>apt软件包的安装



<pre class="example">sudo aptitude -y update
sudo aptitude -y upgrade
sudo aptitude -y install build-essential libsqlite3-dev libreadline6-dev
sudo aptitude -y install libgdbm-dev zlib1g-dev libz2-dev sqlite3 tk-dev
sudo aptitude -y instal zip
</pre>

</li>
<li>python 相关包的安装



<pre class="example">sudo aptitude -y install python-dev
distribute包的安装（该工具是支持python模块的构建与导入的工具包，使用apt也能安装，
但版本可能比较旧）
使用源码安装：
wget http://python-distribute.org/distribute_setup.py
sudo python distribute_setup.py

pip的安装：
pip的安装可以使用easy_install
从pypi安装功能包和模块时可以使用easy_install和pip
使用命令安装pip：
wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
sudo python get-pip.py
确认pip安装完成: pip --version
</pre>

</li>
</ul>

</div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">virtualenv的安装</h2>
<div class="outline-text-2" id="text-2">


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">virtualenv的安装</h3>
<div class="outline-text-3" id="text-2-1">

<ul>
<li>使用pip安装(推荐)



<pre class="example">使用pip安装： pip install virtualenv
</pre>

</li>
<li>使用源码安装



<pre class="example">curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-x-x.tar.gz
cd virtualenv-x-x.tar.gz &amp;&amp; sudo python setup.py install
//非全局安装
python virtualenv.py myve
</pre>

</li>
<li>使用包管理器安装
<ul>
<li>ubuntu



<pre class="example">sudo aptitude install python-virtualenv python-pip
</pre>

</li>
<li>centos



<pre class="example">sudo yum install python-virtualenv python-pip
</pre>

</li>
</ul>

</li>
</ul>

</div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">virutalenv使用方法</h3>
<div class="outline-text-3" id="text-2-2">

<ul>
<li>确认已经安装好的所有包的版本信息
     pip freeze
</li>
<li>创建virtualenv



<pre class="example">virtualenv env: 创建env环境目录， 会在当前目录下创建env目录， 并且安装了env/bin/python,
创建lib,bin,include,local目录以及安装pip，在env目录下，安装的python库都会放入env/lib下
而且默认的python是env/bin/python,包括写在python程序中的#!/usr/bin/env python
</pre>

</li>
<li>激活virtualenv



<pre class="example">cd env
source ./bin/activate
使用pip安装软件
pip freeze &gt; requirements.txt // 生成requirements.txt文件
pip install -r requirements.txt // 根据requirements.txt生成相同的环境
</pre>

</li>
</ul>

</div>

</div>

<div id="outline-container-2-3" class="outline-3">
<h3 id="sec-2-3">最佳实践</h3>
<div class="outline-text-3" id="text-2-3">

<ul>
<li>指定python版本



<pre class="example">virtualenv -p /usr/bin/python3.4 env3.4
virtualenv --python=/python/path env
</pre>

</li>
<li>生成可打包环境
     在某些特殊情况下，可能没有网络，期望直接打包一个env，可以在解压后直接使用
     此时可以使用virtualenv &ndash;relocatable指令将一个env修改为可更改位置的env
     virtualenv &ndash;relocatable ./ ，执行该命令后，将当前env相关的可执行文件都
     被修改为相对路径，可以打包当前目录，上传到其他位置直接使用
</li>
</ul>

</div>

</div>

<div id="outline-container-2-4" class="outline-3">
<h3 id="sec-2-4">virtualenv的bootstrap脚本</h3>
<div class="outline-text-3" id="text-2-4">

<p>   <a href="http://blogs.360.cn/blog/how-360-uses-python-1-virtualenv/">http://blogs.360.cn/blog/how-360-uses-python-1-virtualenv/</a>
</p></div>
</div>
</div>

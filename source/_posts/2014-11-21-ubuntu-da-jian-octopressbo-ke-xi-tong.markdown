---
layout: post
title: "Ubuntu 搭建octopress博客系统"
date: 2014-11-21 12:32:40 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 安装rvm</a>
<ul>
<li><a href="#sec-1-1">1.1 rvm 的使用</a></li>
<li><a href="#sec-1-2">1.2 gem命令的使用</a></li>
<li><a href="#sec-1-3">1.3 bundler 的使用</a></li>
</ul>
</li>
<li><a href="#sec-2">2 安装octopress</a>
<ul>
<li><a href="#sec-2-1">2.1 配置octopress</a></li>
<li><a href="#sec-2-2">2.2 部署到github上</a></li>
<li><a href="#sec-2-3">2.3 octopress的个性化修改</a></li>
</ul>
</li>
<li><a href="#sec-3">3 使用org-mode 生成octopress博客</a>
<ul>
<li><a href="#sec-3-1">3.1 修改emacs配置文件</a></li>
</ul>
</li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">安装rvm</h2>
<div class="outline-text-2" id="text-1">

<p>  curl -sSL <a href="https://get.rvm.io">https://get.rvm.io</a> | bash -s stable
</p>
</div>

<div id="outline-container-1-1" class="outline-3">
<h3 id="sec-1-1">rvm 的使用</h3>
<div class="outline-text-3" id="text-1-1">

<ul>
<li>注意： 使用时需要将当前的shell设置为login-shell
     设置方法是： Edit-&gt;profile preferences-&gt;Title and Command, 勾选Run commands as a login shell
</li>
<li>rvm 的命令



<pre class="example">rvm info: 查看rvm的信息
rvm info &lt;ruby-version&gt;: 查看相应ruby版本下的信息
rvm list known: 查看哪些rvm可用的ruby版本     
</pre>

</li>
<li>修改rvm的ruby安装源到国内的淘宝镜像服务器,可以提高下载的速度



<pre class="example">淘宝镜像服务的地址是： http://ruby.taobao.org/
sed -i.bak 's!ftp.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
</pre>

</li>
<li>ruby的安装与切换
<ul>
<li>列出已知的ruby版本
       rvm list known
</li>
<li>安装ruby
       rvm install &lt;ruby-version&gt;,最新的ruby-version可以到ruby的官网查看
</li>
<li>卸载一个已经安装的版本
       rvm remove &lt;ruby-version&gt;
</li>
<li>查询已经安装的ruby
       rvm list
</li>
</ul>

</li>
<li>gemset的使用
     一个gemset就是一个目录，是某一个Ruby版本的Gem使用集，通过环境变量配置，
     使该gemset下的gem命令导入到Shell。
     gemset是附加在ruby语言版本下面的，例如用ruby-1.9.2建立了一个叫rail192的gemset，当切换到
     ruby-2.1.5的时候，rail192的gemset并不存在。
<ul>
<li>建立gemset
       rvm use &lt;ruby-version&gt;
       rvm gemset create &lt;name&gt;
</li>
<li>设定某个gemset为当前环境
       rvm use &lt;gemset-name&gt;
</li>
<li>列出当前ruby的gemset
       rvm gemset list
</li>
<li>清空gemset中的gem
       rvm gemset empty &lt;gemset-name&gt;: 清空某个gemset中的所有gem
</li>
<li>删除一个gemset
       rvm gemset delete &lt;gemset-name&gt;
</li>
</ul>

</li>
<li>项目自动加载gemset
     进入到项目目录，建立一个.rvmrc文件，在该文件中可以简单的加一个命令：
     rvm use &lt;gemset-name&gt;
     之后，无论当前ruby的设置是什么，只要cd到这个项目的时候，rvm会自动加载相应的&lt;gemset-name&gt;
</li>
<li>删除rvm自身



<pre class="example">rvm implode
在rvm的Troubleshooting(https://rvm.io/support/troubleshooting )页面上有一段脚本

</pre>

</li>
</ul>


{% codeblock lang:bash %}
     #!/bin/bash
     /usr/bin/sudo rm -rf $HOME/.rvm $HOME/.rvmrc /etc/rvmrc /etc/profile.d/rvm.sh /usr/local/rvm /usr/local/bin/rvm
     /usr/bin/sudo /usr/sbin/groupdel rvm
     /bin/echo "RVM is removed. Please check all .bashrc|.bash_profile|.profile|.zshrc for RVM source lines and delete
     or comment out if this was a Per-User installation."
{% endcodeblock %}
</div>

</div>

<div id="outline-container-1-2" class="outline-3">
<h3 id="sec-1-2">gem命令的使用</h3>
<div class="outline-text-3" id="text-1-2">




<pre class="example">RubyGems是随着Ruby一起安装的，当安装好Ruby之后，RubyGems对应的命令gem便可以使用，
gem命令用来安装Ruby世界的第三方软件包，这些软件包被称作gems，类似于Wndows下dll文件
或Linux下的so文件，但是一个Gem并不是单个文件，而是具有一定目录结构的文件夹。
请注意，“gem”既表示RubyGems的命令，又可表示某个Ruby软件包.
</pre>

<ul>
<li>gem 安装其他软件
     gem install &lt;Gems-name&gt;
</li>
<li>列出已经安装的软件
     gem list
</li>
</ul>

</div>

</div>

<div id="outline-container-1-3" class="outline-3">
<h3 id="sec-1-3">bundler 的使用</h3>
<div class="outline-text-3" id="text-1-3">




<pre class="example">bundler主要用于管理Ruby应用程序的依赖关系，并按照此依赖关系安装所需的Gems。
当运行bundle install命令来安装Gems时，bundler会使用当前目录下的名为Gemfile的文件来处理依赖关系。
一个简单的Gemfile内容：
  source "http://rubygems.org"
  gem "nokogiri"
  gem "rack", "~&gt;1.1"
  gem "rspec", :require =&gt; "spec"
解释：
文件第1行表明bundler会从http://rubygems.org下载Gems；
第2行表明需要名为nokogiri的Gem；
第3行表明需要名为rack的Gem，并且版本必须高于1.1；
第4行表明rspec依赖于spec， 所以spec将先于rspec安装。
</pre>


</div>
</div>

</div>

<div id="outline-container-2" class="outline-2">
<h2 id="sec-2">安装octopress</h2>
<div class="outline-text-2" id="text-2">

<p>  参见官方的安装步骤： <a href="http://octopress.org/docs/setup/">http://octopress.org/docs/setup/</a>
</p>


<pre class="example">git clone git://github.com/imathis/octopress.git &lt;your-name&gt;
cd &lt;your-name&gt;
gem install bundler
bundle install
rake install
git remote rm origin # 删掉octopress源码下载的地址
git remote add origin github_blog_git # 添加github博客地址的
git checkout -b blogsrc #新建一个博客源码的分支
git add . # 添加所有的文件到版本库中
git push origin blogsrc # 推送到github的blogsrc分支中
</pre>


</div>

<div id="outline-container-2-1" class="outline-3">
<h3 id="sec-2-1">配置octopress</h3>
<div class="outline-text-3" id="text-2-1">




<pre class="example">修改_config.yml的url, title, subtitle, author, email等信息
其中url为在github创建的仓库地址
source/_include/custom/head.html和source/_include/head.html存有
博客使用的google字体信息，如果访问慢可以删掉。
</pre>

</div>

</div>

<div id="outline-container-2-2" class="outline-3">
<h3 id="sec-2-2">部署到github上</h3>
<div class="outline-text-3" id="text-2-2">

<ul>
<li>生成发布文件



<pre class="example">rake setup_github_pages: 最主要的作用就是创建一个_deploy目录，
目录用来存放部署到master分支的内容。期间会要求你输入仓库的url，
根据提示，进行输入即可。
rake generate: 生成静态html
rake deploy: 如果配置好了github地址，会自动推送到github的master， 
需要确认Rakefile中的deploy_branch为master分支
rake preview: 在推送前可以在本地查看生成的网站内容信息
</pre>

</li>
<li>部署时遇到的问题
<ul>
<li>Could not find a JavaScript runtime.
       此时需要安装nodejs，然后将nodejs的执行路径加入到环境变量PATH中
</li>
</ul>

</li>
<li>nodejs的安装



<pre class="example">去nodejs的官网： www.nodejs.org下载最新的nodejs，解压
执行./configure --prefix=&lt;nodejs-install-path&gt; 
make &amp;&amp; make install
# 添加nodejs的执行路径到path环境变量中
echo 'export PATH="$PATH:&lt;nodejs-install-path&gt;/bin/"' &gt;&gt; ~/.zshrc
source ~/.zshrc # 让配置文件立即生效
</pre>

</li>
<li>为了便于随时修改blog的内容的，可以建立一个src分支，用来存放所有
     博客的源码文件，而master分支则存放由octopress处理生成的结果
</li>
<li>多人协作写博客



<pre class="example">在另外一台电脑上克隆该版本库
执行rake generate
执行rake setup_github_pages, 按提示输入github版本库地址
之后进入_deploy目录，先git pull一下，会报错，按提示执行
git branch --set-upstream-to=origin/master master
发布代码： 执行rake deploy     
</pre>

</li>
</ul>


</div>

</div>

<div id="outline-container-2-3" class="outline-3">
<h3 id="sec-2-3">octopress的个性化修改</h3>
<div class="outline-text-3" id="text-2-3">

<ul>
<li>汉化octopress



<pre class="example">sed -i 's/&lt;h1&gt;Recent Posts&lt;/&lt;h1&gt;最新文章&lt;/' source/_includes/asides/recent_posts.html
sed -i 's/&gt;Blog&lt;/&gt;首页&lt;/;s/&gt;Archives&lt;/&gt;全部文章&lt;/;' source/_includes/custom/navigation.html
sed -i 's/&gt;Blog Archives&lt;/&gt;全部文章&lt;/; s/Older&lt;/较旧&lt;/; s/&gt;Newer/&gt;较新/' source/index.html
sed -i 's/Blog Archive/全部文章/' source/blog/archives/index.html 
</pre>

</li>
<li>个性化
<ul>
<li>页脚说明
       修改source/_includes/custom/footer.html



<pre class="example">&lt;p&gt;
Copyright &amp;copy; date - name -
&lt;span class="credit"&gt;Powered by 
&lt;a href="http://www.gnu.org/software/emacs"&gt;Emacs&lt;/a&gt; +
&lt;a href="http://orgmode.org"&gt;Org mode&lt;/a&gt; +
&lt;a href="http://octopress.org"&gt;Octopress&lt;/a&gt; +
&lt;a href="http://github.com"&gt;GitHub&lt;/a&gt;
&lt;/span&gt;
&lt;/p&gt;
</pre>

</li>
<li>增加标签列表
       安装category_list.rb到octopress/plugins/
       category_list.rb下载地址以及用法：
       <a href="https://github.com/ctdk/octopress-category-list">https://github.com/ctdk/octopress-category-list</a>
       之后在添加文件source/_includes/asides/categories.html
       文件内容是：



<pre class="example">&lt;section&gt;
  &lt;h1&gt;标签列表&lt;/h1&gt;
  &lt;ul id="category-list"&gt;
  &lt;/ul&gt;
&lt;/section&gt;
</pre>

<p>
       之后修改_config.yml中的default_asides增加asides/categories.html
</p></li>
<li>增加个人说明
       添加文件source/_include/asides/aboutme.html
       内容如下：



<pre class="example">&lt;section&gt;
&lt;h1&gt;关于我&lt;/h1&gt;
&lt;p&gt;xxxx&lt;/p&gt;
&lt;/section&gt;
</pre>

<p>
       之后在_config.yml中的default_asides增加asides/aboutme.html
</p></li>
<li>修改导航栏
       修改文件: source/_includes/custom/navigation.html
       可以修改为如下内容：



<pre class="example">&lt;ul class="main-navigation"&gt;
&lt;li&gt;&lt;a href="/"&gt;主页&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href="/blog/archives"&gt;所有文章&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href="/aboutme"&gt;关于我&lt;/a&gt;&lt;/li&gt;
&lt;/ul&gt;
</pre>

</li>
</ul>

</li>
</ul>

</div>
</div>

</div>

<div id="outline-container-3" class="outline-2">
<h2 id="sec-3">使用org-mode 生成octopress博客</h2>
<div class="outline-text-2" id="text-3">


</div>

<div id="outline-container-3-1" class="outline-3">
<h3 id="sec-3-1">修改emacs配置文件</h3>
<div class="outline-text-3" id="text-3-1">

<p>   在配置文件中添加如下的内容：
</p><ul>
<li>定义发布函数
</li>
</ul>


{% codeblock lang:lisp %}
     (defun save-then-publish ()
       (interactive)
       (save-buffer)
       (org-save-all-org-buffers)
       (org-publish-current-project))
{% endcodeblock %}
<ul>
<li>定义org-mode的工程工作目录
</li>
</ul>


{% codeblock lang:lisp %}
     (setq org-publish-project-alist
        '(("blog-org" .  (:base-directory "octorpress/path/org_posts/"
                          :base-extension "org"
                          :publishing-directory "octorpres/path/_posts/"
                          :sub-superscript ""
                          :recursive t
                          :publishing-function org-publish-org-to-octopress
                          :headline-levels 4
                          :html-extension "markdown"
                          :octopress-extension "markdown"
                          :body-only t))
          ("blog-extra" . (:base-directory "octorpress/path/org_posts/"
                           :publishing-directory "octorpress/path/"
                           :base-extension "css\\|pdf\\|png\\|jpg\\|gif\\|svg"
                           :publishing-function org-publish-attachment
                           :recursive t
                           :author nil))
          ("blog" . (:components ("blog-org" "blog-extra")))))
{% endcodeblock %}
<ul>
<li>修改Rakefile
<ul>
<li>查找到 Misc Configs， 修改内容如下：



<pre class="example">new_post_ext    = "org"  # default new post file extension when using the new_post task
new_page_ext    = "org"  # default new page file extension when using the new_page task
org_posts_dir   = "org_posts" #新添加的
</pre>

</li>
<li>修改生成文件的初始内容



<pre class="example">在该文件中查找到rake new_post命令执行的代码，在其post.puts "---" 前添加post.puts "#+BEGIN_HTML"
在后一个post.puts "---"添加 post.puts "#+END_HTML"
</pre>

</li>
</ul>

</li>
<li>下载org-octopress.el
     <a href="https://github.com/craftkiller/orgmode-octopress">https://github.com/craftkiller/orgmode-octopress</a>
     之后修改emacs的配置文件：



<pre class="example">(add-to-list 'load-path "orgmode-octopress/path")
(require 'org-octopress)
</pre>

</li>
<li>使用方法
     rake "new_post[title]"
     mv source/_posts/xxx.org octorpress/path/org_posts
     之后在emacs中执行命令:



<pre class="example">M-x save-then-publish 
</pre>

</li>
</ul>

</div>
</div>
</div>

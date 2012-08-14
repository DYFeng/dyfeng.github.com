---
layout: post
title: "Using Octopress as Github Blog on Ubuntu 12.04"
description: ""
category: Server
tags: [github , octopress]
---

# 前言

本来我的Github blog是使用*Jekyll-Bootstrap*作为生成引擎。不过他是把文章发给Github解析的，由于Github出于安全考虑，限制ruby脚本的运行，这样就导致很多功能实现不了。这时候我发现了*Octopress*引擎，他把你的branch分成两个，一个master，负责放置静态的页面，一个是source，放置程序本身和Markdown日志。


# 名词解释

无论是*Jekyll-Bootstrap*还是*Octopress*，他们都用到了*ruby*写的后端*jekyll*，我对*ruby*基本上没有多少接触，看到N个*ruby*的名词更是晕了，在此记录下

- **gem**
  *ruby*的*easy_install*，用来安装各种库，是用*ruby*写的，全称叫*rubygems*。

- **bundler**
  基于*gem*的更高级管理工具，*bundler*相对于*gem*就好比*apt-get*相对于*aptitude*。不过他不是单纯的下载安装，他会根据本目录的*Gemfile*文件，把你缺少的包给装上。
  
- **rvm**
  *Ruby Version Manager*，用来安装各种版本的*ruby*，问题是*ubuntu*有apt-get，这个不大派上用场。

- **rbenv**
  *Simple Ruby Version Management*，也是用来安装各种版本的*ruby*。
  
- **rake**
  *Ruby Make*，顾名思义就是*ruby*写的*make*，他对于的*Makefile*是*Rakefile*
  
# 安装Octopress

## 配置安装环境

### 安装*ruby 1.9.3*
要安装*Octopress*，必须有*ruby 1.9.3*。因为我不喜欢用第三方软件来安装*ruby*，所以我是[用apt-get的方式安装Ruby][Installing Ruby]。

	sudo apt-get update
	sudo apt-get install ruby1.9.1 ruby1.9.1-dev \
	rubygems1.9.1 irb1.9.1 ri1.9.1 rdoc1.9.1 \
	build-essential libopenssl-ruby1.9.1 libssl-dev zlib1g-dev
	
	sudo update-alternatives --install /usr/bin/ruby ruby /usr/bin/ruby1.9.1 400 \
    --slave   /usr/share/man/man1/ruby.1.gz ruby.1.gz \
    /usr/share/man/man1/ruby1.9.1.1.gz \
    --slave   /usr/bin/ri ri /usr/bin/ri1.9.1 \
    --slave   /usr/bin/irb irb /usr/bin/irb1.9.1 \
    --slave   /usr/bin/rdoc rdoc /usr/bin/rdoc1.9.1
	
	sudo update-alternatives --config ruby
	sudo update-alternatives --config gem

	ruby --version

这样子，我们就不需要用到*rvm*或者*rbenv*这些东西了。要注意一点的是，虽然看上去我们装的是*1.9.1*，其实他是*Ruby 1.9.3-p0*。为什么他又叫*1.9.1*呢？因为这个*1.9.1*只是一个*ABI*版本号而已。*ubuntu*有一个*ruby1.9.3*的虚包，其实也是指向*1.9.1*的。

### 下载安装Octopress

下载Octopress，再次确定*ruby*版本

	git clone git://github.com/imathis/octopress.git octopress
	cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
	ruby --version  # Should report Ruby 1.9.3

安装*bundler*并使用*bundler*安装依赖

	gem install bundler
	bundle install

安装默认Octopress主题

	rake install

## 部署到Github

其实部署Github非常简单。

交互式输入你的Github仓库地址（*git@github.com:username/username.github.com.git*，把*username*替换成你的用户名）。

	rake setup_github_pages

把你的域名提交给Github。

	echo 'www.dyfeng.org' > source/CNAME

在*public*目录生成页面。

	rake generate

将*public*里的静态页面复制到*_deploy*目录,上传到master branch

	rake deploy
	
上传程序本身到source branch

	git add .
	git commit -m 'init'
	git push origin source

这个时候，如果一切正常，你应该可以看到你的*Octopress Blog*了。

## 本地预览

打开[http://localhost:4000](http://localhost:4000)即可看到效果。

	rake preview

# 配置Octopress

看看你的*_config.yml*文件，blog的基本设置都在里面了，包括第三方插件的设置。

# 使用Octopress

新建文章，很贴心的一点是中文的题目他会转换为拼音，一些特殊符号他都会帮你转成相应的英文。不过为了更好地SEO，最好还是起一个有意义的题目，而不是一堆拼音。

	cd octopress
	rake new_post["The Title of Your Article"]

```js
	alert('hello')
```
[Installing Ruby]: http://lenni.info/blog/2012/05/installing-ruby-1-9-3-on-ubuntu-12-04-precise-pengolin/ "Installing Ruby on Ubuntu 12.04"

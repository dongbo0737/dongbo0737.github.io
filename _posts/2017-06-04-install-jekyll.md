---
layout: post
title:  "Windows10 下安装Jekyll"
date:   2017-06-04
categories: jekyll windows
tags: jekyll windows
---

* content
{:toc}

搭建1个博客。将自己碰到的问题，解决的办法与大家一起分享。




## 搭建过程

主要安装：安装Ruby，安装RubyGems，安装jekyll

### 安装 Ruby

1.安装rvm(rvm ruby的包管理工具)

	curl -L https://get.rvm.io | bash -s stable

2.使rvm生效

	source ~/.rvm/scripts/rvm

3.修改rvm下载ruby的源

	echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db

4.查询rvm是否安装成功

	rvm -v

5.安装RVM的环境依赖

	rvm requirements

6.安装ruby 下载2.3.0版本的ruby

	rvm install 2.3.0

7,切换ruby版本

	rvm use 2.3.0 --default

8.查询版本是否切换成功

	ruby -v
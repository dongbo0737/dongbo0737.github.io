---
layout: post
title:  "Windows10 下安装Jekyll"
date:   2017-06-05
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

9.查询gem版本

    gem -v


### 安装jekyll

1.安装jekyll

	gem install jekyll

2.查询jekyll版本

	jekyll --version
	gem list jekyll

3.查询gem中所有jekyll版本

	gem search jekyll --remote

4.安装指定版本jekyll

	gem install jekyll -v '2.0.0.alpha.1'


## 异常

1.启动异常

	jekyll 3.4.3 | Error:  Invalid argument - Failed to watch "/mnt/c/Users/dongbo/Desktop/jekyll/dongbo.github.io/.git/branches": the given event mask contains no legal events; or fd is not an inotify file descriptor.

解决:

	使用 jekyll server --force_polling启动
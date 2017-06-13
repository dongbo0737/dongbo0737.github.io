---
layout: post
title:  "docker 安装"
date:   2017-06-13
categories: docker
tags: docker
---

* content
{:toc}

Docker 是 PaaS 提供商 dotCloud 开源的一个基于 LXC 的高级容器引擎，源代码托管在 Github 上, 基于go语言并遵从Apache2.0协议开源。






## 必要的环境：

Ubuntu版本:

Yakkety 16.10
Xenial 16.04
Trusty 14.04

## 安装 repository

Set up the Docker CE repository on Ubuntu. The lsb_release -cs sub-command prints the name of your Ubuntu version, like xenial or trusty.

### 1. 安装一些ubuntu 上的支持插件

如果机器上本来就有，就可以不用下了

	sudo apt-get -y install \
	  apt-transport-https \
	  ca-certificates \
	  curl

### 2.添加证书

	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

	如果出现异常
	sudo: add-apt-repository: command not found
	下载包
	$ sudo apt-get install software-properties-common python-software-properties

### 3. 添加源

sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"

## 4.更新包

	sudo apt-get update

	安装 Docker CE
	Install the latest version of Docker CE on Ubuntu:

	sudo apt-get -y install docker-ce

	 Test your Docker CE installation


### 5.测试

测试安装是否成功，运行一个hello-world:

	#给你自己的用户加入docker组，否则会出现权限问题

	sudo usermod -aG docker $USER 

	测试
	sudo docker run hello-world








---
layout: post
title:  "Rancher日志搜集"
date:   2017-06-29
categories: docker rancher logstash logspout elasticsearch
tags: docker rancher logstash logspout elasticsearch
---

* content
{:toc}

收集Rancher中容器的日志推送到es中，进行统计管理，计算，集合，解析





## 简介

抓取rancher中容器的日志，推送到es中

## 目标

通过logspout抓如rancher中的容器日志，推送到logstash容器中的 5000 upd端口

logstash监听udp5000 端口，收集日志，并将日志推送到es中

## 注意

需要先部署logstash容器

logspout和logstash需要部署到所有docker host上

Dockerfile定制：不可以有中文注释

## 流程

![流程](/images/rancher-log/process.png)

## Logstash

### dockerfile

通过dockerfile构建镜像部署到rancher平台


	FROM logstash:5.2.2 #官方镜像
	MAINTAINER dongbo (dongbo01@docker.com) #定义作者
	COPY conf.d/ /etc/logstash/conf.d #拷贝配置文件
	EXPOSE 5000 #开放端口
	EXPOSE 6000 #开放端口
	CMD ["-f","/etc/logstash/conf.d"] #容器启动执行命令


生成镜像

	docker build -t dongbo01/test-logstash:5.2.2 .

推送镜像到docker  hub 上

	docker push  dongbo01/test-logstash:5.2.2

### 配置文件

100-input.conf:监听udp 5000端口


	input {
	   udp {
	    port => 5000
	    codec => "json"
	  }
	}


300-es-output.conf：将日志推送到日志服务器，索引rancher-beta-log-YYYY.MM.dd,类型：log

推送的同时会把推送的数据打一份在控制台：主要用于调试（上生产需要关闭）


	output { 
	    elasticsearch {
	            hosts => ["172.25.200.82:9200"]
	            index => "rancher-beta-%{+YYYY.MM.dd}"
	           document_type => "log"
	    }
	    stdout {
	            codec => rubydebug
	    }
	}


## 部署

### 安装

通过rancher平台部署

1.添加stack

![addstack](/images/rancher-log/addstack.png)

![addstack-2](/images/rancher-log/addstack-2.png)

2.添加service

![addservice-1](/images/rancher-log/addservice-1.png)

![addservice-2](/images/rancher-log/addservice-2.png)

### 查询状态

![addservice-2](/images/rancher-log/addservice-3.png)


## Logspout

安装：通过rancher平台安装，从catalog中找到人家封装好的镜像

### 安装

![logspout-1](/images/rancher-log/logspout-1.png)

![logspout-2](/images/rancher-log/logspout-2.png)

![logspout-3](/images/rancher-log/logspout-3.png)

### 查询部署状态

![logspout-4](/images/rancher-log/logspout-4.png)

### 查询es数据

去我们的es监控里面查看是否收集到日志,我们前面配置文件中定义的索引：rancher-beta-%{+YYYY.MM.dd}

![logspout-5](/images/rancher-log/logspout-5.png)







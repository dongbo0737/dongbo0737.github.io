---
layout: post
title:  "Elasticsearch 5.2.1安装"
date:   2017-06-06
categories: elasticsearch java
tags: elasticsearch java
---

* content
{:toc}

安装5.2.1ES流程，安装使用tar包安装，去官网下载:https://www.elastic.co/downloads/past-releases
安装过程出现问题，或者启动出现问题，可以参考：Elasticsearch 2.4 升级 5.2 注意事项及问题汇总





## 注意

java版本一定要在1.8_73以上


### 系统参数修改

增大max_map_count

	sudo su
	sysctl -w vm.max_map_count=262144

第一步

	sudo vi /etc/security/limits.conf

	*    soft    nofile    655360
	*    hard    nofile    655360
	*    soft    nproc     2048
	*    hard    nproc     4096
	*    soft    memlock   unlimited
	*    hard    memlock   unlimited
 

第二步

	sudo vim /etc/pam.d/su
	session required pam_limits.so#注释放开，使limits.conf生效



## 插件安装

### X-PACK

安装文档流程：Elasticsearch5.2.1扩展X-PACK实施流程

 

### IK分词

install IK analysis
ik分词器，版本需要在1.10.0

1:git上下载源码

	git clone https://github.com/medcl/elasticsearch-analysis-ik.git

2：切换版本

	cd elasticsearch-analysis-ik/
	git checkout v1.10.0

3:重新编译，打包

	mvn clean
	mvn compile
	mvn package

4:解压zip包，将解压后的文件移动到/opt/elasticsearch/plugins目录

	mkdir ik
	mv target/releases/elasticsearch-analysis-ik-1.10.0.zip ik/
	unzip elasticsearch-analysis-ik-1.10.0.zip
	rm elasticsearch-analysis-ik-1.10.0.zip
	mv ik/ /opt/elasticsearch/plugins/
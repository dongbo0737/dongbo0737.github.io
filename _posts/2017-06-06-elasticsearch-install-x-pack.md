---
layout: post
title:  "Elasticsearch5.2.1扩展X-PACK实施流程"
date:   2017-06-06
categories: elasticsearch java
tags: elasticsearch java
---

* content
{:toc}

elasticsearch5.2.1安装X-PACK，对ES集群进行监控，报警，安全验证，报告，图形化操作






## 注意

版本号需要固定，小版本都不能差，要不然会报错

在es安装完x-pack后，logstash配置文件中的output需要修改，增加username和password参数，要不然logstash中的数据无法推送到es中


## 安装

分线上和脱机2种情况安装，如果可以联通外网推荐使用线上安装，会自动匹配版本，如果是脱机安装需要下载对应的zip包

### 线上

#### 1.ES安装x-pack

	bin/elasticsearch-plugin install x-pack

如果es没有自动创建x-pack所需要的索引，需要手动在elasticsearch.yml中配置

	action.auto_create_index: .security,.monitoring*,.watches,.triggered_watches,.watcher-history*

#### 2.启动ES

	bin/elasticsearch

#### 3.kibana安装x-pack

	bin/kibana-plugin install x-pack

#### 4.启动kibana

	bin/kibana

#### 5.logstash安装x-pack

	bin/logstash-plugin install x-pack

#### 6.启动logstash

	bin/logstash -f logstash.conf
 

### 脱机

x-pack安装包

下载地址： https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.2.1.zip

下载完成后不用解压，放在一个临时目录

#### 1.ES安装x-pack

	bin/elasticsearch-plugin install file:///path/to/file/x-pack-5.2.1.zip

如果es没有自动创建x-pack所需要的索引，需要手动在elasticsearch.yml中配置

	action.auto_create_index: .security,.monitoring*,.watches,.triggered_watches,.watcher-history*

#### 2.启动ES

	bin/elasticsearch

#### 3.kibana安装x-pack

	bin/kibana-plugin install file:///path/to/file/x-pack-5.2.1.zip

#### 4.启动kibana

	bin/kibana

#### 5.logstash安装x-pack

	bin/logstash-plugin install file:///path/to/file/x-pack-5.2.1.zip

#### 6.启动logstash

	bin/logstash -f logstash.conf


### 配置

setting	描述

<table>
    <tr>
        <td>xpack.security.enabled</td>
        <td>默认为true</td>
        <td>配置是否开启安全验证</td>
        <td>在elasticsearch.yml和kibana.yml中都可以配置</td>
    </tr>
     <tr>
        <td>xpack.monitoring.enabled</td>
        <td>默认为true</td>
        <td>是否开启monitoring功能</td>
        <td>在elasticsearch.yml和kibana.yml中都可以配置</td>
    </tr>
     <tr>
        <td>xpack.graph.enabled</td>
        <td>默认为true</td>
        <td>是否开启graph功能</td>
        <td>在elasticsearch.yml和kibana.yml中都可以配置</td>
    </tr>
     <tr>
        <td>xpack.watcher.enabled</td>
        <td>默认为true</td>
        <td>是否开启watcher功能</td>
        <td>只在elasticsearch.yml中配置</td>
    </tr>
     <tr>
        <td>xpack.reporting.enabled</td>
        <td>默认为true</td>
        <td>是否开启watcher功能</td>
        <td>只在kibana.yml中配置</td>
    </tr>
</table>



### 权限管理

x-pack默认账号:elastic 密码:changeme



修改密码

	curl -XPUT -u elastic 'localhost:9200/_xpack/security/user/elastic/_password' -d '{
	  "password" : "elasticpassword"
	}'
	 
	curl -XPUT -u elastic 'localhost:9200/_xpack/security/user/kibana/_password' -d '{
	  "password" : "kibanapassword"
	}'
	 
	curl -XPUT -u elastic 'localhost:9200/_xpack/security/user/logstash_system/_password' -d '{
	  "password" : "logstashpassword"
	}'


新增角色

	curl -XPOST -u elastic 'localhost:9200/_xpack/security/role/events_admin' -d '{
	  "indices" : [
	    {
	      "names" : [ "events*" ],
	      "privileges" : [ "all" ]
	    },
	    {
	      "names" : [ ".kibana*" ],
	      "privileges" : [ "manage", "read", "index" ]
	    }
	  ]
	}'
 

关联用户

	curl -XPOST -u elastic 'localhost:9200/_xpack/security/user/dongbo' -d '{
	  "password" : "123456",#需要在6位以上
	  "full_name" : "dongbo",
	  "email" : "dongbo01@ule.com",
	  "roles" : [ "events_admin" ]
	}'
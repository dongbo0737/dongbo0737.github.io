---
layout: post
title:  "Elasticsearch 2.4 升级 5.2 注意事项及问题汇总"
date:   2017-06-06
categories: elasticsearch java
tags: elasticsearch java
---

* content
{:toc}

Elasticsearch 2.4 升级 5.2

此文档只适合elasticsearch2.x升级到5.x

如果想要1.x升级到5.x请查看官网

升级之前需要先备份重要数据，防止数据丢失






## 检查：安装 Migration Helper

这个工具有3个功能

1).Cluster Checkup

    Runs a series of checks on your cluster, nodes, and indices and alerts you to any known problems that need to be rectified before upgrading.

2).Reindex Helper

    Indices created before v2.0.0 need to be reindexed before they can be used in Elasticsearch 5.x. The reindex helper upgrades old indices at the click of a button.

3).Deprecation Logging

     Elasticsearch comes with a deprecation logger which will log a message whenever deprecated functionality is used. This tool enables or disables deprecation logging on your cluster.

### 1.安装

	./bin/plugin install https://github.com/elastic/elasticsearch-migration/releases/download/v2.0.4/elasticsearch-migration-2.0.4.zip

### 2.登陆检查页面，根据页面按钮操作
 
http://ip:9200/_plugin/elasticsearch-migration
 

### 3.如果需要检查远程集群中的节点，需要在elasticsearch.yml文件增加
 

	http.cors.enabled: true
	http.cors.allow-origin: # the hostname or a regex which matches the hostname
 

### 4.如果你的es集群有权限控制
 
	http.cors.allow-credentials: true
 

## 升级Elasticsearch

### 1.停止自动分片

	curl -XPUT 'http://ip:9200/_cluster/settings?pretty' -H 'Content-Type: application/json' -d'
	{
	  "persistent": {
	    "cluster.routing.allocation.enable": "none"
	  }
	}'

### 2.同步刷新

	curl -XPOST 'http://192.168.211.130:9200/_flush/synced?pretty'

### 3.停止es服务，升级所有节点

登陆官网查询不同的升级方式（Debian or RPM or zip or tar）
https://www.elastic.co/guide/en/elasticsearch/reference/5.2/rolling-upgrades.html#upgrade-node

如果是tar包升级，下载当前5.2版本的tar包进行解压

	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.1.tar.gz
	sha1sum elasticsearch-5.2.1.tar.gz 
	tar -xzf elasticsearch-5.2.1.tar.gz
	cd elasticsearch-5.2.1/ 
 
将Migration Helper校验过的2.4的yml配置文件进行覆盖

### 4.查询节点是否正常启动，并且加入集群中

	curl -XGET 'http://ip:9200/_cat/nodes?pretty'

### 5.开启自动分片

	curl -XPUT 'http://ip:9200/_cluster/settings?pretty' -H 'Content-Type: application/json' -d'
	{
	  "transient": {
	    "cluster.routing.allocation.enable": "all"
	  }
	}'

### 6.等待集群恢复，查询集群状态

	curl -XGET 'http://ip:9200/_cat/health?pretty'

### 7.查询分片恢复情况

	curl -XGET 'http://ip:9200/_cat/recovery?pretty'
 

## 问题修复

### 1 max_map_count 参数太小

报错信息

	ERROR:max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决

	sudo su
	sysctl -w vm.max_map_count=262144
 

### 2.当前用户的最大线程数量太小

报错信息

	max number of threads [1024] for user [elasticsearch] is too low, increase to at least [2048]

解决

	vi /etc/security/limits.conf
	*               soft     nofile          65536
	*               hard     nofile         65536
	*               soft     nproc          2048
	*               hard     nproc         4096
	sudo vim /etc/pam.d/su
	session required pam_limits.so#注释放开，使limits.conf生效
 

### 3 bootstrap checks failed

报错信息

	ERROR: bootstrap checks failed  memory locking requested for elasticsearch process but memory is not locked
 
解决

	vi /etc/security/limits.conf
	 * soft memlock unlimited
	 * hard memlock unlimited
	 sudo vim /etc/pam.d/su
	 session required pam_limits.so#注释放开，使limits.conf生效
 

   
### 4.ES 2.4升级ES 5.2，IK分词器升级后，分词不能兼容，出现异常

报错信息

	Caused by: java.lang.IllegalArgumentException: Unknown tokenizer type [ik] for [my_tokenizer]
	 at org.elasticsearch.index.analysis.AnalysisRegistry.getAnalysisProvider(AnalysisRegistry.java:387) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.analysis.AnalysisRegistry.buildMapping(AnalysisRegistry.java:338) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.analysis.AnalysisRegistry.buildTokenizerFactories(AnalysisRegistry.java:176) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.analysis.AnalysisRegistry.build(AnalysisRegistry.java:154) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.IndexService.(IndexService.java:146) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.IndexModule.newIndexService(IndexModule.java:363) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.indices.IndicesService.createIndexService(IndicesService.java:425) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.indices.IndicesService.verifyIndexMetadata(IndicesService.java:458) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.cluster.metadata.MetaDataIndexStateService$2.execute(MetaDataIndexStateService.java:173) ~[elasticsearch-5.2.1.jar:5.2.1]
	   [2017-02-28T10:04:11,039][WARN ][o.e.g.Gateway ] [dev211130] recovering index [ule-webaccess-2017.02.28/9vjPkRjdTzmwCrTtenJlaA] failed - recovering as closed
	 org.elasticsearch.index.mapper.MapperParsingException: Failed to parse mapping [logs]: analyzer [ik] not found for field [message]
	 at org.elasticsearch.index.mapper.MapperService.internalMerge(MapperService.java:323) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.MapperService.internalMerge(MapperService.java:277) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.MapperService.merge(MapperService.java:256) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.indices.IndicesService.verifyIndexMetadata(IndicesService.java:461) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.gateway.Gateway.performStateRecovery(Gateway.java:135) [elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.gateway.GatewayService$1.doRun(GatewayService.java:229) [elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:596) [elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:37) [elasticsearch-5.2.1.jar:5.2.1]
	 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [?:1.8.0_101]
	 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [?:1.8.0_101]
	 at java.lang.Thread.run(Thread.java:745) [?:1.8.0_101]
	 Caused by: org.elasticsearch.index.mapper.MapperParsingException: analyzer [ik] not found for field [message]
	 at org.elasticsearch.index.mapper.TypeParsers.parseAnalyzersAndTermVectors(TypeParsers.java:125) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.TypeParsers.parseTextField(TypeParsers.java:210) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.StringFieldMapper$TypeParser.parse(StringFieldMapper.java:312) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.ObjectMapper$TypeParser.parseProperties(ObjectMapper.java:281) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.ObjectMapper$TypeParser.parseObjectOrDocumentTypeProperties(ObjectMapper.java:203) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.RootObjectMapper$TypeParser.parse(RootObjectMapper.java:102) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.DocumentMapperParser.parse(DocumentMapperParser.java:111) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.DocumentMapperParser.parse(DocumentMapperParser.java:91) ~[elasticsearch-5.2.1.jar:5.2.1]
	 at org.elasticsearch.index.mapper.MapperService.internalMerge(MapperService.java:320) ~[elasticsearch-5.2.1.jar:5.2.1]
	 ... 10 more
  
解决：
 

	ule-business索引解决方式
	 1：先关闭索引
	curl -XPOST 'ip:9200/my_index/_close?pretty'
	 2：修改索引的setting
	 curl -XPUT 'ip:9200/my_index/_settings' -d '{
	    "analysis": {
	        "analyzer": {
	            "camel_analyzer": {
	                "type": "custom",
	                "tokenizer": "ik_smart"
	            }
	        },
	        "tokenizer": {
	            "my_tokenizer": {
	                "type": "whitespace",
	                "enable_lowercase": "false"
	            }
	        }
	    }
	}'
	   
	3：打开索引
	 curl -XPOST 'ip:9200/my_index/_open?pretty
	 
	ule-webaccess索引解决方式
	 
	 1：先关闭索引
	curl -XPOST 'ip:9200/my_index/_close?pretty'
	 2：修改索引的setting
	 curl -XPUT 'ip:9200/my_index/_settings' -d '{
	    "analysis": {
	        "analyzer": {
	            "ik": {
	                "type": "custom",
	                "tokenizer": "ik_smart"
	            }
	        }
	    }
	}'
	 
	3：打开索引
	curl -XPOST 'ip:9200/my_index/_open?pretty'
	 

### 5.   2.x客户端连接5.x ES出现异常

报错信息

	Received message from unsupported version: [2.0.0] minimal compatible version is: [5.0.0]

解决：
 
	找到2.4的客户端升级到5.2
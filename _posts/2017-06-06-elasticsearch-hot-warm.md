---
layout: post
title:  "Elasticsearch 主节点和暖热节点"
date:   2017-06-06
categories: elasticsearch java
tags: elasticsearch java
---

* content
{:toc}

Elasticsearch 主节点和暖热节点解析





## 主节点

控制整个集群，进行一些轻量级操作，列如：跟踪哪些节点是集群中的一部分，决定节点分片分配，负责集群健康，
不包含数据，也不参与搜索和索引操作，对于CPU,RAM，磁盘的性能要求比Data节点小很多

## Data节点

### 热节点


这个数据节点执行集群内的所有索引操作。他们也持有最近的指数，因为这些指数通常最常被查询。

由于索引是CPU和IO密集型操作，因此这些服务器需要功能强大，并附带SSD存储。

建议至少运行3个热节点以实现高可用性。根据收集的日志量，再确定需要多少机器

通过elasticsearch.yml中配置：node.attr.box_type: hot 

也可以通过启动的时候命令：./bin/elasticsearch -Enode.attr.box_type=hot



### 暖节点


这种类型的数据节点被设计为处理不太长用的索引数据，是只读索引。

暖节点倾向于利用大型连接的磁盘（通常是旋转磁盘）而不是SSD。

与热节点一样，我们建议至少有3个暖节点用于高可用性。与之前一样。

要注意的是，大量的数据可能需要额外的节点来满足性能要求。

还要注意，CPU和内存配置一般都是和热节点一样。具体配置需要根据生产环境实际情况来配置

通过elasticsearch.yml中配置：node.attr.box_type: warm

也可以通过启动的时候命令：./bin/elasticsearch -Enode.attr.box_type=warm

可以在暖节点的elasticsearch.yml配置： index.codec: best_compression 来对分配到暖节点的数据进行压缩


### API接口

ES会根据索引的setting来自动将数据分配到热节点或者暖节点

分配logs_2016-12-26到热节点上

	PUT /logs_2016-12-26/_settings
	{
	  "settings": {
	    "index.routing.allocation.require.box_type": "hot"
	  }
	}


分配logs_2016-12-26到暖节点上

	PUT /logs_2016-12-26/_settings
	{ 
	  "settings": { 
	    "index.routing.allocation.require.box_type": "warm"
	  } 
	}
	 

通过模板来分配索引数据该分配到热节点还是暖节点

热节点：

	{
	  "template" : "indexname-*",#如果需要把所有的索引默认都指向热节点，可以配置为*
	  "version" : 50001,
	  "settings" : {
	             "index.routing.allocation.require.box_type": "hot"
	 ...
	}
	 

暖节点：

	{
	  "template" : "indexname-*",
	  "version" : 50001,
	  "settings" : {
	           "index.routing.allocation.require.box_type": "warm"
	 ...
	}

---
layout: post
title:  "filebeat.yml"
date:   2017-06-13
categories: filebeat
tags: filebeat config
---

* content
{:toc}

filebeat 一个ELK架构中，专门用来收集数据的插件

官网：https://www.elastic.co/guide/en/beats/filebeat/5.2/index.html

filebeat安装很简单，推荐使用tar包下载，解压

配置文件需要放置在一个专门的目录中






## filebeat.yml

	shipper:
	  name: 147240-filebeat-1
	  queue_size: 1000
	filebeat:
	  spool_size: 2048
	  idle_timeout: 2s
	  registry_file: /data/logs/filebeat/filebeat-1/.filebeat #存储偏移量文件
	  publish_async: false #异步推送
	  prospectors:
	  - paths:
	    - /data/logs/tomcat/aa-am.log #定义被收集的日志的路径
	    document_type: logs #文档乐行
	    ignore_older: 24h # 忽略24小时之前的日志
	    close_older: 1h
	    scan_frequency: 10s # 10秒扫描一次文件
	    tail_files: true
	    force_close_files: true
	    fields_under_root: true
	    fields: #定义收集的日志特有属性
	      app: sam
	      server: tomcat
	      ver: 1
	      hostip: 127.0.0.1
	      logname: test-am
	      module: dm
	      assigner: DongBo
	      logid: 1476203703774
	    multiline:
	      pattern: '^[[:space:]]+|^Caused by:'
	      negate: false
	      match: after
	      max_lines: 500
	output.kafka: # 将收集到的日志推送到kafka中
	  hosts:  ["kafkahost:port1","kafkahost:port2"]
	  topic: business #定义topic
	  key: business
	  partition.round_robin:
	      reachable_only: false
	  required_acks: 1
	  compression: gzip
	  max_message_bytes: 1000000
	logging: # filebeat自身打印日志
	  level: info
	  to_files: true
	  to_syslog: false
	  files:
	    path: /data/logs/filebeat/filebeat-1
	    rotateeverybytes: 104857600
	    name: filebeat.log
	    keepfiles: 7






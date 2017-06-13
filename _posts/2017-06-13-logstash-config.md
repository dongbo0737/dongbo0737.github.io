---
layout: post
title:  "Logtash 5.0 配置文件解析"
date:   2017-06-13
categories: logstash
tags: logstash config
---

* content
{:toc}

logstash 一个ELK架构中，专门用来进行接受数据进行处理，可以和很好的扩展节点

官网：https://www.elastic.co/guide/en/logstash/5.2/index.html

logstash安装很简单，推荐使用tar包下载，解压


## logstash.yml

	//配置文件，以英文为准
	# Settings file in YAML
	#
	# Settings can be specified either in hierarchical form, e.g.:
	#
	#   pipeline:
	#     batch:
	#       size: 125
	#       delay: 5
	#
	# Or as flat keys:
	#
	#   pipeline.batch.size: 125
	#   pipeline.batch.delay: 5
	#
	# ------------  Node identity ------------
	#
	# Use a descriptive name for the node:
	#
	//节点名称
	 node.name: dev211133
	#
	# If omitted the node name will default to the machine's host name
	#
	# ------------ Data path ------------------
	#
	# Which directory should be used by logstash and its plugins
	# for any persistent needs. Defaults to LOGSTASH_HOME/data
	#
	//数据存储路径
	 path.data: /data/logstash/data
	#
	# ------------ Pipeline Settings --------------
	#
	# Set the number of workers that will, in parallel, execute the filters+outputs
	# stage of the pipeline.
	#
	# This defaults to the number of the host's CPU cores.
	#
	//输出通道的工作workers数据量（提升输出效率）
	 pipeline.workers: 8
	#
	# How many workers should be used per output plugin instance
	#
	//每个输出插件的工作wokers数量
	# pipeline.output.workers: 1
	#
	# How many events to retrieve from inputs before sending to filters+workers
	#
	//每次input数量
	 pipeline.batch.size: 4000
	#
	# How long to wait before dispatching an undersized batch to filters+workers
	# Value is in milliseconds.
	#
	# pipeline.batch.delay: 5
	#
	# Force Logstash to exit during shutdown even if there are still inflight
	# events in memory. By default, logstash will refuse to quit until all
	# received events have been pushed to the outputs.
	#
	# WARNING: enabling this can lead to data loss during shutdown
	#
	# pipeline.unsafe_shutdown: false
	#
	# ------------ Pipeline Configuration Settings --------------
	#
	# Where to fetch the pipeline configuration for the main pipeline
	#
	//过滤配置文件目录
	 path.config: /opt/logstash/config/conf.d
	#
	# Pipeline configuration string for the main pipeline
	#
	# config.string:
	#
	# At startup, test if the configuration is valid and exit (dry run)
	#
	# config.test_and_exit: false
	#
	# Periodically check if the configuration has changed and reload the pipeline
	# This can also be triggered manually through the SIGHUP signal
	#
	//自动从新加载被修改配置
	# config.reload.automatic: false
	#
	# How often to check if the pipeline configuration has changed (in seconds)
	#
	//配置文件检查时间
	# config.reload.interval: 3
	#
	# Show fully compiled configuration as debug log message
	# NOTE: --log.level must be 'debug'
	#
	//开始debug日志
	# config.debug: false
	#
	# ------------ Metrics Settings --------------
	#
	# Bind address for the metrics REST endpoint
	#
	//绑定主机地址，用户指标收集
	 http.host: "192.168.211.133"
	#
	# Bind port for the metrics REST endpoint, this option also accept a range
	# (9600-9700) and logstash will pick up the first available ports.
	#
	//绑定端口
	 http.port: 5000-9700
	#
	# ------------ Debugging Settings --------------
	#
	# Options for log.level:
	#   * fatal
	#   * error
	#   * warn
	#   * info (default)
	#   * debug
	#   * trace
	#
	//日志输出级别和路径，如果config.debug开启，这里一定要是debug日志
	 log.level: info
	 path.logs: /data/logstash/logs
	#
	# ------------ Other Settings --------------
	#
	# Where to find custom plugins
	//自定义插件
	# path.plugins: []


## startup.options

	################################################################################启动配置，以英文为准
	# These settings are ONLY used by $LS_HOME/bin/system-install to create a custom
	# startup script for Logstash.  It should automagically use the init system
	# (systemd, upstart, sysv, etc.) that your Linux distribution uses.
	#
	# After changing anything here, you need to re-run $LS_HOME/bin/system-install
	# as root to push the changes to the init script.
	################################################################################

	# Override Java location
	#本地jdk
	JAVACMD=/usr/bin/java

	# Set a home directory
	#logstash所在目录
	LS_HOME=/opt/logstash

	# logstash settings directory, the path which contains logstash.yml
	#默认logstash配置文件目录
	LS_SETTINGS_DIR="${LS_HOME}/config"

	# Arguments to pass to logstash
	#logstash启动命令参数 指定配置文件目录
	LS_OPTS="--path.settings ${LS_SETTINGS_DIR}"

	# Arguments to pass to java
	#指定jdk目录
	#LS_JAVA_OPTS=""

	# pidfiles aren't used the same way for upstart and systemd; this is for sysv users.
	#logstash.pid所在目录
	LS_PIDFILE=/var/run/logstash.pid

	# user and group id to be invoked as
	#logstash启动组和用户
	LS_USER=logstash
	LS_GROUP=logstash

	# Enable GC logging by uncommenting the appropriate lines in the GC logging
	# section in jvm.options
	#logstash jvm gc日志路径
	LS_GC_LOG_FILE=/var/log/logstash/gc.log

	# Open file limit
	#logstash最多打开监控文件数量
	LS_OPEN_FILES=65534

	# Nice level
	LS_NICE=19

	# Change these to have the init script named and described differently
	# This is useful when running multiple instances of Logstash on the same
	# physical box or vm
	#初始化脚本和描述名称
	SERVICE_NAME="logstash"
	SERVICE_DESCRIPTION="logstash"

	# If you need to run a command or script before launching Logstash, put it
	# between the lines beginning with `read` and `EOM`, and uncomment those lines.
	###
	## read -r -d '' PRESTART << EOM
	## EOM

## 集成的插件配置，数据处理

logstash加载配置是循序加载

### 100-input.conf

输入插件

	input {
	    kafka {
	        bootstrap_servers => "172.25.156.113:9092,172.25.156.114:9092,172.25.156.115:9092"
	        group_id => "clio-consr-weba-go1"#consumergroup
	        topics => ["ule-webaccess"]#topic
	        session_timeout_ms => "60000"#session超时
	        request_timeout_ms => "180000"#request超时
	        max_poll_records => "500"
	        check_crcs => "true"
	        codec => "json"
	        decorate_events => true#输出kafka信息
	        consumer_threads => 3#消费的线程，根据threads*workers*服务器数量=partition
	        add_field => {
	         "processor_host" => "172.25.156.74"
	        }
	    }
	}


### 200-initialize-filter.conf

初始化配置

	filter {
	    mutate {
	        add_tag => [ "invalid" ] #添加invaild标签
	        add_field => [ "receive" , "%{[@timestamp]}" ]#将filebeat带过来的timestamp赋值给receive
	    }
	    ruby {
	        init => "require 'time'"
	        code => "event.set('processor_timestamp' , Time.now());event.set('lag' , Time.now().to_i-event.get('@timestamp').to_i)"#ruby计算时间差，进入logstash时间- filebeat抓取日志时间
	    }
	#进行kafka.topic判断，如果是app是audit,topic赋值为ule-audit，、
	#后面输入到es会根据topic 生成索引
	    if [kafka][topic] == "ule-business" {
	        if [app] == "audit" {
	            mutate {
	                update => { "[kafka][topic]" => "ule-audit" }
	            }
	        }  else {
	            mutate { 
	                remove_tag => [ "invalid" ]#删除invalid标签
	            }
	        }
	    }
	  
	   #
	    if [kafka][topic] == "system-logstash" {
	       mutate {
	          remove_tag => [ "invalid" ]
	        }
	    }
	}


### 210-webaccess-filter.conf

apache日志接受处理

	#apache日志过滤
	filter {
	    if [kafka][topic] == "ule-webaccess" {
	        grok {
	#根据apache日志格式解析
	            match => { "message" => "\"%{DATA:xforward}\" %{COMBINEDAPACHELOG} (?:%{NUMBER:duration:float}) (?:%{DATA:domain}) \"(?:%{DATA:protocol}|)\" \"(?:%{DATA:rawurlpath})\" \"(?:%{DATA:rawurlquery}|)\" (?:%{DATA:method}) (?:%{NUMBER:ibytes:int}) (?:%{NUMBER:obytes:int}) \"(?:%{DATA:uleck}|)\"" }
	        }
	        date { 
	#格式化时间戳
	            match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
	        }
	        mutate { 
	#删除timestamp字段和invalid标签
	            remove_field => [ "timestamp" ]
	            remove_tag => [ "invalid" ] 
	        }
	    }

	}


### 211-audit-filter.conf 

审计日志处理

	filter {
	    if [kafka][topic] == "ule-audit" {
	        grok {
	#格式化日志信息
	            match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}" }
	        }
	        date { 
	#格式化时间戳
	            match => [ "timestamp" , "yyyy-MM-dd HH:mm:ss,SSS" , "ISO8601" ] 
	        }
	        mutate { 
	#删除invalid标签
	            remove_tag => [ "invalid" ] 
	        }
	    }
	}


### 299-common-filter.conf

数据处理完成后的通用过滤

	filter {
	#判断浏览器agent,进行解析
	    if [agent] and [agent] != "-" and [agent] != "" {
	        useragent {
	            source => "agent"
	            prefix => "UA-"
	        }
	    }
	#如果有timestamp字段，则删除
	    if [timestamp] {
	        mutate {
	            remove_field => ["timestamp"]
	        }
	    }
	}	


### 300-elasticsearch-output.conf

输出插件，一般都是输出到es中，当然也可以输出到redis，kafka中，具体官网上查询


	output {
	#如果标签为invalid，不进行输出到es
	  if "invalid" not in [tags]  {
	#如果topic是system-logstash生成是索引是按月来的
	      if [kafka][topic] == "system-logstash" {
	        elasticsearch {
	           hosts => ["172.25.156.62:9202","172.25.156.66:9202"]#es服务器
	           index => "%{[kafka][topic]}-%{+YYYY.MM}"#指定索引格式
	           document_type => "%{[type]}"#文档类型
	           flush_size => 5000#缓存数量
	          }
	      } else {
	#生成索引按天来
	         elasticsearch {
	           hosts => ["172.25.156.62:9202","172.25.156.66:9202"]
	           index => "%{[kafka][topic]}-%{+YYYY.MM.dd}"
	           document_type => "%{[type]}"
	           flush_size => 5000
	          }
	      }
	    }
	}
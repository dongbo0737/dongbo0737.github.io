---
layout: post
title:  "Logtash遇到的异常和注意点"
date:   2017-06-13
categories: logstash
tags: logstash config
---

* content
{:toc}

logstash 遇到的一些异常解决





## 1：logstash使用kafka插件和es集成

如果logstash使用kafka插件和es集成，必须设置kafka插件参数 
session_timeout_ms => "10000"
max_poll_records => "500"
如果这2个值过高会导致es重复消费，而kafka中的offset偏移不会进行增加

## 2："retrying failed action with response code: 429 

Logstash提示这样的错误是因为bulk operations queue满了，要么调小flush_size的值，或者增大elasticsearch的thread
增大Elasticsearch的bulk线程池队列
配置文件中增加
threadpool.bulk.queue_size: 1000

## 3：logstash数据插入es中效率太慢

增到配置文件中的batch-size和workers

## 4：使用ruby函数

logstash5.0使用ruby设置值和取值
例子：code => "event.set('server_time' , Time.now())"设置当前server_time值为当时时间
          code => "event.get('server_time')" 获取server_time的值
logstash5.0之前使用ruby
 列子：code => "event.['se ， rver_time'] = Time.now()"设置当前server_time值为当时时间
          code => "event.['server_time']"获取server_time的值

## 5: Auto offset commit failed for group clio-consr-biz-go1:

  Commit cannot be completed since the group has already rebalanced and assigned the partitions to another member. 
  This means that the time between subsequent calls to poll() was longer than the configured session.timeout.ms,
   which typically implies that the poll loop is spending too much time message processing.
 You can address this either by increasing the session timeout or by reducing the maximum size of batches returned in poll() with max.poll.records.


提交消费组offset失败，无法完成重新分配partition。
网上描述：该问题是因为logstash无法在限定时间内消费完所有的数据，超出了kafka端设定的session timeout，导致session挂掉，且之前消费过的数据offset未能返回给kafka。在kafka端会认为该数据没有正确消费，并进行重新partition。logstash端超时后会重新建立consumer进行数据拉取，而kafka端会因为offset的问题重新发送之前“消费失败”的数据。
解决办法，增加session.timout.ms的值或者减少max.poll.records。
注意session.timout.ms增大的同事也要增加request.timeout.ms参数，而已session.timeout.ms要小于request.timeout.ms

## 6：一定要记得在使用的机器上修改hosts文件！

否则会造成无法连接kafka的情况。因为在某些情况下，连接是直接使用hostname进行的。

## 7：filebeat数据进入logstash

所有filebeat定义的field都是string类型，而进入kafka再进去logstash，kafka会自动识别field转换类型
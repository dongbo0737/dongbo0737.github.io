---
layout: post
title:  "Eclipse Memory Analyzer 堆转储文件分析"
date:   2017-06-19
categories: logstash
tags: logstash config
---

* content
{:toc}

参考：https://www.ibm.com/developerworks/cn/opensource/os-cn-ecl-ma/index.html

在生产上的项目出现内存泄漏，内存溢出等异常时，无法进行项目重新发布，项目中打印的日志又无法找到具体的问题时，就需要我们进行一些别的方式来找到问题，解决问题了，这里推荐可以使用Eclipse Memory Analyzer 插件来解析内存







## 一 在服务器上生成hprof(堆存储文件)文件

### 1.使用jdk自带的jmap工具来生成文件

	jmap -dump:format=b,file=#{filename}.hprof  #{PID}

在运行应用之前，指定参数参数

### 2.在内存泄露的时候生成hprof

	-XX:+HeapDumpOnOutOfMemoryError

### 3.按需获取hprof 文件

	-XX:+HeapDumpOnCtrlBreak

## 二 eclipse 安装 Memory Analyzer 分析插件


### 1.到eclipse商店中找到 插件 直接安装

![第一一步](/images/eclipse-memory-analyzer/1.png)

![第一二步](/images/eclipse-memory-analyzer/2.png)

### 2.切换view

![第二步](/images/eclipse-memory-analyzer/3.png)


### 3. 打开我们通过jmap生成的dump 文件

![第三步](/images/eclipse-memory-analyzer/4.png)

### 4.选择Leak Suspects 查询具体内存泄漏详情

![第四步](/images/eclipse-memory-analyzer/5.png)


### 5.选择有问题的提示，detail查询详情

![第五步](/images/eclipse-memory-analyzer/6.png)

### 6.找到具体异常的对象

![第六步](/images/eclipse-memory-analyzer/6.png)
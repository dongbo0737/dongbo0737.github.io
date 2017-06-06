---
layout: post
title:  "ubuntu jdk 安装"
date:   2017-06-06
categories: ubuntu java
tags: ubuntu java
---

* content
{:toc}


ubuntu 上安装jdk



## 方法一：

### 1:下载jdk 

	sudo curl -o jdk-8u101.tar.gz -L --cookie "oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz//官网下载地址

由于 Oracle 的下载页面必须要先点击接受用户协议才能下载, 所以我们用 curl 来模拟 cookie 的发送, 方能下载

### 2:解压压缩包

	tar -zxvf jdk-xxx.tar.gz

### 3:将jdk放入/usr/local/目录下

	mv jdk-xxx /usr/local/java/

### 4:编辑/etc/profile文件 相当于添加windows的环境变量 在最后添加

	export JAVA_HOME=/usr/local/java/jdk-xxx
	export JRE_HOME=$JAVA_HOME/jre
	export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
	export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin


### 5:使配置文件生效

	source /etc/profile

### 6::查看是否生效

	java -version


## 方法二(推荐)：

### 1:下载

	sudo curl -o jdk-xxx.tar.gz -L --cookie "oraclelicense=accept-securebackup-cookie" 	http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz

### 2:解压

	tar -xzvf jdk-8u101.tar.gz
	sudo mv ~/download/jdk1.8.0_101/ /usr/lib/jvm/

### 3:追加 jdk 的 update-alternatives， 方便多版本间切换

	sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_101/jre/bin/java 2000
	sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_101/bin/javac 2000

### 4：检查设置

	sudo update-alternatives --config java
	sudo update-alternatives --config javac

### 5:创建 /etc/profile.d/oraclejdk.sh 初始化文件,内容：

	export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_101
	export J2SDKDIR=/usr/lib/jvm/jdk1.8.0_101
	export J2REDIR=$JAVA_HOME/jre
	export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/db/bin:$JAVA_HOME/jre/bin
	export DERBY_HOME=$JAVA_HOME/db

执行source /etc/profile.d/oraclejdk.sh 完成初始化



---
layout: post
title:  "Rancher实现自动化部署"
date:   2017-06-29
categories: docker rancher harbor jenkins
tags: docker rancher harbor jenkins
---

* content
{:toc}

使用docker rancher harbor jenkins 这些技术来实现自动化部署项目
先将timer部署到rancher中





## 简介

jenkins自动构建项目生成image,发布到harbor中

harbor发布成功通过rancher cli在rancher中生成一个service

官网

jenkins：https://jenkins.io/doc/

harbor：https://github.com/vmware/harbor

docker:https://docs.docker.com/engine/

rancher:http://docs.rancher.com/rancher/v1.6/en/



## 目标

将TIMER项目部署至rancher环境


## 注意

1.docker镜像需要小写

2.rancher service name 命名规则[a-zA-Z0-9],符号支持-，其他基本都不支持

3.rancher cli语句需要在一行，如果需要多行需要加 \  再换行

4.需要在jenkins所在主机调用需要下载rancher cli加入环境变量

进入rancher server，右下角下载

![1](/images/autodeploy/rancher-client.png)

5.Dockerfile定制：不可以有中文注释


## 流程描述

![1](/images/autodeploy/process.png)


## Jenkins

### 安装

先到官网下载jenkins.war

2种运行方式：

1：jenkins.war放入tomcat webapps，启动tomcat
2：java -jar jenkins.war


### 插件管理

系统管理-插件管理-可选插件
ssh plugin
Maven Integration plugin
Rebuilder

![jenkins-1](/images/autodeploy/jenkins-1.png)

配置jenkins工作目录

![jenkins-2](/images/autodeploy/jenkins-2.png)

配置Jenkins location

![jenkins-3](/images/autodeploy/jenkins-3.png)

Global Tool配置修改

![jenkins-4](/images/autodeploy/jenkins-4.png)

添加各个版本jdk

![jenkins-5](/images/autodeploy/jenkins-5.png)

添加各版本maven

![jenkins-6](/images/autodeploy/jenkins-6.png)

### 新增Jenkins任务

1.新增maven任务

![jenkins-7](/images/autodeploy/jenkins-7.png)

![jenkins-8](/images/autodeploy/jenkins-8.png)


2.新增参数

![jenkins-9](/images/autodeploy/jenkins-9.png)

通过String Parameter添加的参数可以通过url调用参数来传过来，参数名就是名字

3.选择JDK版本

![jenkins-10](/images/autodeploy/jenkins-10.png)

4.选择源码仓库，目前使用的svn

![jenkins-11](/images/autodeploy/jenkins-11.png)

5.选择maven版本，定义maven构建命令

![jenkins-12](/images/autodeploy/jenkins-12.png)

6.maven构建成功后执行命令

![jenkins-13](/images/autodeploy/jenkins-13.png)

test.sh内容：

	#!/bin/bash
	SERVER=$1
	MODULE=$2
	APP=$3
	VERSION=$4
	ENV=$5
	#tomcat
	if [ "$SERVER" = "tomcat" ]
	then
	 echo 'tomcat'
	elif [ "$SERVER" = "jboss" ]
	then
	 echo 'jboss'
	elif [ "$SERVER" = "timer" ]
	then
	 echo 'timer'
	   
	  if [ "$ENV" = "prd" ];
	   then
	      /bin/bash -x /data/script/jenkins/createTimerPrdDockerFile.sh $MODULE $APP $VERSION
	  else
	     /bin/bash -x /data/script/jenkins/createTimerDockerFile.sh $MODULE $APP $VERSION
	  fi
	elif [ "$SERVER" = "jar" ]
	then
	 echo 'jar'
	fi


createTimerDockerFile.sh:

	#!/bin/bash
	SERVER=timer
	declare -l MODULE=$1
	APP=$2
	VERSION=$3
	#echo $MODULE $APP
	#修改打出来了包名字为应用名
	mkdir -p /tmp/jenkins/$MODULE-$APP-$SERVER
	mv /data/jenkins/workspace/timer/clio/logsearchtimer/target/*.jar /tmp/jenkins/$MODULE-$APP-$SERVER/$APP.jar
	#替换Dockerfile中的MODEL-APP.war
	cp /data/script/jenkins/timer/Dockerfile /tmp/jenkins/$MODULE-$APP-$SERVER/Dockerfile
	sed -i "s/MODULE-APP.jar/$APP.jar/g"  /tmp/jenkins/$MODULE-$APP-$SERVER/Dockerfile
	declare -l APP=$APP
	#build image
	docker build -t $MODULE-$APP-$SERVER:$VERSION /tmp/jenkins/$MODULE-$APP-$SERVER
	#tag
	docker tag $MODULE-$APP-$SERVER:$VERSION 192.168.154.216:8080/timer/$MODULE-$APP:$VERSION
	#push image
	docker push 192.168.154.216:8080/timer/$MODULE-$APP:$VERSION

createTimerDockerFile.sh和createTimerPrdDockerFile.sh不同在于，harbor仓库地址不一样


Dockerfile模板:

	FROM 192.168.154.216:8080/java/jdk1.8:111
	#update time zone
	RUN echo "Asia/Shanghai" > /etc/timezone && \
	    dpkg-reconfigure -f noninteractive tzdata
	RUN mkdir -p /data/logs/timer
	 
	COPY MODULE-APP.jar /MODULE-APP.jar


解析rancher cli命令：

![rancher-client](/images/autodeploy/rancher-client.png)


	rancher  --url http://192.168.112.126:8080/v2-beta \ #定义rancher部署的环境url
	--access-key DFCA9E38C06961CE11DF   --secret-key FhiA24kNZKyd7nP158vnFrDsUcdegtDPSeKkTppL \ #定义请求的账号key，是rancher中自动生成的
	run -name "TIMER/logsearchtimer-aggs" \ # 定义rancher stack和service名称
	--label io.rancher.project_service.name="TIMER/logsearchtimer-5" \ #定义service下的容器名称
	--label io.rancher.stack_service.name="TIMER/logsearchtimer-5" \ #定义stack下的容器名称 需要和io.rancher.project_service.name统一
	--label io.rancher.scheduler.affinity:host_label:service="timer"  \ #定义service运行的宿主机标签
	--label io.rancher.container.start_once=true \#只启动一次。默认如果容器停止会一直重启，但是定时不需要。会有自己的重启机制
	--label cron.schedule="$RULE" \ #定时规则
	-v /data/logs/timer:/data/logs/timer \ #挂载目录
	 192.168.154.216:8080/timer/${MODULE}-${APP}:${REVISION} java -jar ${APP}.jar ${PARAM} #从指定harbor上拉去镜像，容器启动后立即执行 java -jar命令


### 执行jenkins任务

点击 Build with Parameters 填写参数，开始构建

![jenkins-14](/images/autodeploy/jenkins-14.png)

点击 Rebuild Last，可查看上一次任务执行的参数，并重新构建

![jenkins-15](/images/autodeploy/jenkins-15.png)

通过  配置 选项可以修改我们的 jenkins任务

![jenkins-16](/images/autodeploy/jenkins-16.png)

通过  工作空间 可以查看我们从svn上拉去的代码

![jenkins-17](/images/autodeploy/jenkins-17.png)

## Harbor

### 安装
具体安装参考:

[Harbor 安装和使用](/2017/06/29/harbor-install/)

github：https://github.com/vmware/harbor

### 镜像

jenkins构建完成，上传的镜像

![harbor-image](/images/autodeploy/harbor-image.png)

## Rancher

### 安装

参考: [Rancher 安装和使用](/2017/06/29/rancher-install/)

官网：http://docs.rancher.com/rancher/v1.6/en/


### service



通过rancher cli生成的docker容器已经正常运行

![rancher-service-1](/images/autodeploy/rancher-service-1.png)

详情：

![rancher-service-2](/images/autodeploy/rancher-service-2.png)

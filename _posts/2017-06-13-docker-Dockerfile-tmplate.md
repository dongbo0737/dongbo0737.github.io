---
layout: post
title:  "Dockerfile template"
date:   2017-06-13
categories: docker Dockerfile
tags: docker Dockerfile
---

* content
{:toc}


Dockerfile 用于docker快速构建镜像

这里定义个可以从主机上ssh到docker容器的 Dockerfile

注意:Dockerfile中的注释不可以是中文，docker镜像名需要小写









	#set bash image
	FROM ubuntu:14.04

	#author
	MAINTAINER dongbo (dongbo01@docker.com)

	#RUN command
	RUN apt-get update && \
	    apt-get install -y openssh-server && \
	    sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd 
	    
	RUN mkdir -p /var/run/sshd
	RUN mkdir -p /root/.ssh

	#add run.sh authorized_keys
	ADD authorized_keys /root/.ssh/authorized_keys
	ADD run.sh /run.sh
	RUN chmod 755 /run.sh

	#open 22 port
	EXPOSE 22

	#startup
	CMD ["/run.sh"]
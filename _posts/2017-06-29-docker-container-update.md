---
layout: post
title:  "切换docker root dir"
date:   2017-06-29
categories: docker
tags: docker
---

* content
{:toc}

修改docker的container目录,如果放在系统盘。容器和镜像太多，或者日志打印过大，会将系统盘堆满，所以需要切换container目录

官网链接：https://docs.docker.com/v1.11/engine/reference/commandline/daemon/#daemon-configuration-file








## 注意

目录切换完成后，下载个hello-world镜像进行测试

修改完docker root dir，重启后，下载镜像报错：
docker: Error response from daemon: containerd: container did not start before the specified timeout.
ERRO[0133] error getting events from daemon: context canceled
解决：重启服务器


## 创建daemon.json


### 切换root权限，创建/etc/docker/daemon.json:

	sudo su
	vim /etc/docker/daemon.json

### daemon.json内容：

graph:切换docker root dir

registry-mirrors：切换镜像下载的默认镜像私服

	{"graph":"/data/docker/container/","registry-mirrors": ["https://z8x8tdv4.mirror.aliyuncs.com"]}
 

### 创建保存的目录:

	mkdir -p /data/docker/container/
 

### 重启:

	sudo service docker restart
 
### 查询是否切换成功

docker info
 
	Containers: 18
	 Running: 12
	 Paused: 0
	 Stopped: 6
	Images: 14
	Server Version: 17.03.1-ce
	Storage Driver: aufs
	 Root Dir: /data/docker/container/aufs
	 Backing Filesystem: extfs
	 Dirs: 105
	 Dirperm1 Supported: true
	Logging Driver: json-file
	Cgroup Driver: cgroupfs
	Plugins: 
	 Volume: local
	 Network: bridge host macvlan null overlay
	Swarm: inactive
	Runtimes: runc
	Default Runtime: runc
	Init Binary: docker-init
	containerd version: 4ab9917febca54791c5f071a9d1f404867857fcc
	runc version: 54296cf40ad8143b62dbcaa1d90e520a2136ddfe
	init version: 949e6fa
	Security Options:
	 apparmor
	Kernel Version: 3.19.0-25-generic
	Operating System: Ubuntu 14.04.3 LTS
	OSType: linux
	Architecture: x86_64
	CPUs: 32
	Total Memory: 125.8 GiB
	Name: rz15831
	ID: VJQW:EZPK:FXAK:O7IE:SKAR:OGAG:TSVC:X6U2:EQTZ:DKZV:BMXZ:K6HA
	Docker Root Dir: /data/docker/container
	Debug Mode (client): false
	Debug Mode (server): false
	Registry: https://index.docker.io/v1/
	WARNING: No swap limit support
	Experimental: false
	Insecure Registries:
	 172.25.158.100
	 127.0.0.0/8
	Registry Mirrors:
	 https://z8x8tdv4.mirror.aliyuncs.com
	Live Restore Enabled: false

<font color="red">Docker Root Dir: /data/docker/container</font>

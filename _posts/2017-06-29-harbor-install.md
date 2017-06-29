---
layout: post
title:  "Harbor 安装和使用"
date:   2017-06-29
categories: docker harbor
tags: docker harbor
---

* content
{:toc}


Project Harbor是一个企业级registry服务器，用于存储和分发Docker镜像。 Harbor通过添加企业通常需要的功能（如安全性，身份和管理）来扩展开源Docker Distribution。作为企业私人registry机构，Harbor提供更好的性能和安全性。使registry更接近构建和运行环境，提高了镜像传输效率。 Harbor支持多个registry的设置，并在它们之间复制镜像。此外，Harbor还提供高级安全功能，例如用户管理，访问控制和活动审计。

 GitHub: https://github.com/vmware/harbor








## Harbor 安装

环境  Ubuntu14.04 

### 下载 harbor

	$ wget  https://github.com/vmware/harbor/releases/download/0.5.0/harbor-offline-installer-0.5.0.tgz
	$ tar xvf harbor-offline-installer-0.5.0.tgz
	$ vi harbor/harbor.cfg
	$ hostname = reg.mydomain.com   ##（reg.mydomain.com修改为自己的IP）其他默认即可


### docker-compose

harbor需要docker-compose支持


	$ sudo curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose   #下载docker-compose
	$ sudo chmod  +x /usr/local/bin/docker-compose  #添加docker-compose命令的执行权限
	$ docker-compose --version    #查看docker-compose版本
	 
	然后执行$ sudo bash harbor/install.sh
	 
	使用$ docker ps 看下，起六个容器
	CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS              PORTS                                      NAMES
	2f4305b4cd0f        nginx:1.11.5                     "nginx -g 'daemon ..."   12 seconds ago      Up 11 seconds       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   nginx
	f09c54d64dc9        vmware/harbor-jobservice:0.5.0   "/harbor/harbor_jo..."   12 seconds ago      Up 11 seconds                                                  harbor-jobservice
	8f86d0097f6c        vmware/harbor-ui:0.5.0           "/harbor/harbor_ui"      13 seconds ago      Up 12 seconds                                                  harbor-ui
	f7c143cca943        library/registry:2.5.0           "/entrypoint.sh se..."   13 seconds ago      Up 12 seconds       5000/tcp                                   registry
	614b3dfdfc76        vmware/harbor-db:0.5.0           "docker-entrypoint..."   13 seconds ago      Up 12 seconds       3306/tcp                                   harbor-db
	b0ec3a24f324        vmware/harbor-log:0.5.0          "/bin/sh -c 'crond..."   16 seconds ago      Up 13 seconds       0.0.0.0:1514->514/tcp                      harbor-log



### 查看效果

![harbor](/images/harbor/harbor.png)


## Harbor使用

### 登陆

harbor 默认登陆账号/密码:admin/Harbor12345

如果是在linux上登陆harbor

	docker login $IP


### 重启

	$ sudo docker-compose stop
	$ sudo docker-compose start

### 配置文件有变动

要更改Harbor的配置，首先停止现有的Harbor实例，更新harbor.cfg，然后再次运行install.sh：

	$ sudo docker-compose down 
	$ vim harbor.cfg
	$ sudo install.sh

### 删除Harbor的容器

#### 不删除镜像和Harbor的数据库文件

	$ sudo docker-compose down

#### 删除Harbor的数据库和镜像数据（用于干净的重新安装）：

	$ rm -r /data/database
	$ rm -r /data/registry

### docker节点添加harbor仓库为主仓库

两种方式:

#### 修改/etc/default/docker

	vim /etc/default/docker
	添加DOCKER_OPTS="--insecure-registry $IP"

#### 修改daemon.json

	sudo  vi /etc/docker/daemon.json

	{
	"insecure-registries": ["$IP"]
	}

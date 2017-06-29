---
layout: post
title:  "Rancher 安装和使用"
date:   2017-06-29
categories: docker rancher
tags: docker rancher
---

* content
{:toc}

Rancher是一个完整的，开源的平台，用于在生产环境中部署和管理容器。它包括Kubernetes，Mesos和Docker Swarm的商业支持发行版，使得在任何基础架构上轻松运行容器化应用程序。

官网： http://rancher.com/

docker hub ：https://hub.docker.com/




## rancher 安装和启动

rancher 的安装和启动很简单

	sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server

查询启动日志

	sudo docker logs -f $ID

 rancher集群参数

  --advertise-address  IP or Node

 rancher有内置的数据库，一般不需要连接外部数据
 如果需要连接外部数据库,参数:

 --db-host myhost.example.com --db-port 3306 --db-user username --db-pass password --db-name cattle


## rancher使用

 在rancher启动完成后，因为我们前面绑定的8080端口，我们可以通过http://ip:port访问到ui界面


![主页](/images/rancher/index.png)

### 添加主机

第一步一定要添加一台主机，要不然什么都做不了

先进入操作页面

![主页](/images/rancher/host.png)

复制命令，找到一台docker主机，直接运行命令

![主页](/images/rancher/addhost.png)

![主页](/images/rancher/addhostcmd.png)

等待几分钟，重新进入主机页面可以发现多了一台主机

![主页](/images/rancher/onehost.png)


### 添加我们第一个服务

![主页](/images/rancher/oneservice.png)

![主页](/images/rancher/addservice.png)

第一个服务启动完成

![主页](/images/rancher/startservice.png)


如果想要获取更多镜像可以访问docker hub上找心仪的镜像，或者直接制作

docker hub ：https://hub.docker.com/
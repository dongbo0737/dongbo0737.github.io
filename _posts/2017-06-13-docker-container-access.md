---
layout: post
title:  "docker容器垮主机访问"
date:   2017-06-13
categories: docker
tags: docker exception
---

* content
{:toc}


原生的docker容器跨主机相互访问（一共2种方法）

1.将容器暴露端口到在主机上（不单独介绍了，很简单）

2.新建1个网络，通过网卡来进行互相访问








1,安装1个键值数据，如：Consul,Etcd,Zookeeper等
安装consul:
 docker run -d -p 8500:8500 -h consul progrium/consul -server -bootstrap

2.找2台Docker主机配置参数，不要在有数据库这台，版本要在1.7.0+
DOCKER_OPTS="$DOCKER_OPTS --cluster-store=consul://$CONSUL_NODE_IP:8500  --cluster-advertise=网卡(如eth0):2376"

参数配置完成
sudo service docker restart

如果没有正常启动，查看日志
sudo tail -f /var/log/upstar/docker.log

3.创建网络
在2台docker主机上任意一台上执行命令，创建1个新的网络
docker network create -d overlay multi
overlay:驱动，可以理解为镜像
multi：名称

跑完之后，可以在2台docker主机上运行命令
docker network ls，可以发现多了1个新的网络 multi

4,查询网络的具体信息
docker network inspect multi

5.在2台docker主机上启容器，进行ping操作,可以启busybox容器进行测试，只有几M
docker run -it --name=c1 --net=multi busybox
docker run -it --name=c2 --net=multi busybox






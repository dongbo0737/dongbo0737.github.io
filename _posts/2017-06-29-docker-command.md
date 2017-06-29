---
layout: post
title:  "docker常用命令记录"
date:   2017-06-29
categories: docker
tags: docker
---

* content
{:toc}


记录一些docker常用命令，以便随时查询







## 命令

#### docker构建镜像，-t 打印日志到终端

	docker build -t 名称：tag 目录

	docker  tag  $镜像ID或者镜像名称   127.0.0.1/docker-imager/ubuntu-ule:14.04  #（打标记）

	docker  push  127.0.0.1/docker-imager/ubuntu-ule:14.0

#### 运行一个docker容器

	docker run -it 名称 

#### 删除1个docker容器

	docker -rm 名称/id

#### 启动一个nginx docker 并绑定port

	docker run --name webserver -d -p 80:80 nginx

####  进入启动的docker容器 bash进入交互式shell环境

	docker exec -it dockerID bash


  ● -it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
  ● --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
  ● bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。

####  将容器制作镜像

	docker commit -m 'Add a new file' -a 'dongbo' 容器id 镜像名称:tag
 
####  存储镜像为1个tar包

	docker save -o myubuntu_14.04.tar myubuntu:14.04

#### 载入镜像

	docker load --input myubuntu_14.04.tar

#### 将容器导出

	docker export -o timer_docker.tar 容器id

#### 导入容器,导入的容器会变成1个镜像

	docker export -o timer_docker.tar f20e52f1b593

docker import 和docker load的区别

docker load是导入镜像到本地镜像库，docker import 导入一个容器快照到本地镜像库，容器快照将丢弃所有的历史记录和元数据信息，镜像存储会保存完整记录，体积也更大

#### 在docker hub中查询ubuntu镜像

	docker search -s 5 ubuntu 

-s 指定星级，星级越高，镜像越好
5表示只查询5星以上的ubuntu镜像

##### 挂载主机上的/data/docker目录到/opt/webapp下

	docker run -d -p 8080:8080  --name test-volums -v /data/docker:/opt/webapp myubuntu:14.04

#### 创建1个数据卷容器

	docker run -it -v /dbdata --name dbdata myubuntu:14.04

#### 挂载数据卷容器

	docker run -it --volumes-from dbdata --name db2 myubuntu:14.04

#### 查询5星以上的spark镜像，星级越高，镜像越好

	docker search -s 5  spark


#### 批量停止docker 容器,慎用！！

	docker stop $(docker ps -q)

#### 批量删除docker 容器,慎用！！

	docker rm $(docker ps -qa)

#### 批量删除docker镜像,慎用！！

	docker rmi $(docker images -qa)

#### 停止一个docker 容器

	docker stop dockerID 

#### 开始一个docker容器

	docker start dockerID 

#### 拷贝本地数据到docker容器中

	docker cp containerId:/path/xxx /host/path/target/xxx

	docker cp /host/path/target/xxx containerId:/path/xxx
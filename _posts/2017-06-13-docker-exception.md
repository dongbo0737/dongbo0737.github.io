---
layout: post
title:  "docker遇到的异常和注意点"
date:   2017-06-13
categories: docker
tags: docker exception
---

* content
{:toc}


整理一些使用docker的时候，遇到的问题和解决办法





删除镜像时出现：Error response from daemon: conflict: unable to delete 95219df55354 (must be forced) - image is referenced in multiple repositories

可以通过 docker rmi 名称：tag 来删除，不用id删除

查询docker daemon启动日志
tail -f /var/log/upstart/docker.log

can't create unix socket /var/run/docker.sock: is a directory
rm -rf  /var/run/docker.sock 然后重新启动


Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
查看 ls -l /var/run/docker.sock，所在的组，（一般在docker组中）
将用户重新加入docker组中，并且重新登录即可（usermod -aG docker ${USER}）


状态为dead的容器删除报错
Error response from daemon: Driver aufs failed to remove root filesystem 0753373785c32e6dabd87396a63844711f4eab04841a0c3183b881d6d939c003: aufs: unmount error after retries: /var/lib/docker/aufs/mnt/aa87e99f1031d48daa66ce1abfb1550248918e4a4bfd737310913e903e4a3f83: device or resource busy
使用docker rm -fv 容器id 过几分钟后会自动删除

docker: Error response from daemon: driver failed programming external connectivity on endpoint quizzical_bell (f09acf9ddddc99be6397c5031f5c680088aedd6fe9593d86c4a38c21fdf29d4d):  (iptables failed: iptables --wait -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.17.0.2 --dport 8080 -j ACCEPT: iptables: No chain/target/match by that name.
关闭防火墙，重启docker daemo
sudo service docker restart


docker: Error response from daemon: driver failed programming external connectivity on endpoint kind_turing3 (78479c5356c5cd6aa003b1b97816610ab1c716288faddb48827ee1fdc7a87492): Bind for 0.0.0.0:80 failed: port is already allocated
容器端口冲突，更换宿主机绑定端口
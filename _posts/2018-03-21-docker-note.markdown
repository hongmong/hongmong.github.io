---
layout: post
title:  "Docker学习笔记"
date:   2018-03-21 08:08:08
categories: linux
tags: docker
mathjax: true
---

* content
{:toc}

### 安装
```bash
//移除之前的docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
//移除之前的yum安装docker仓库配置
sudo rm /etc/yum.repos.d/docker*.repo
//添加新的yum安装docker仓库配置
#http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum-config-manager --add-repo https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo
//更新yum软件包索引
sudo yum makecache fast
//安装最新的docker
sudo yum -y install docker-engine
#Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10
# sudo yum -y install docker-ce
//安装特定版本的docker
yum list docker-engine.x86_64 --showduplicates |sort -r
sudo yum -y install docker-engine-<VERSION_STRING>
//启动docker
sudo systemctl start docker
```

### 常用命令
```bash
#拉取镜像
docker pull mysql/mysql-server:5.7
#运行镜像，运行后会产生一个容器
docker run -d -p 3306:3306 -v /opt/mysql/my.cnf:/etc/my.cnf -v /opt/mysql/data:/var/lib/mysql -v /opt/mysql/log:/var/log -e MYSQL_ROOT_PASSWORD=123456 --name mymysql mysql/mysql-server:5.7
#查看所有的容器
docker ps -a
#正在运行中的容器
docker ps
#删除容器
docker rm #ID#
#进入到容器内部
docker exec -it #ID# bash
#查看容器日志
docker logs #ID#
#停止容器
docker stop #ID#
#启动容器
docker start #ID#
#重启容器
dokcer restart #ID#
```
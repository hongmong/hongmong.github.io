---
layout: post
title:  "CentOs Gitlab安装"
date:   2020-04-17 08:08:08
categories: linux
tags: linux git gitlab
excerpt: CentOs Gitlab安装及常用操作
mathjax: false
---

* content
{:toc}

# CentOS7

https://about.gitlab.com/install/?version=ce#centos-7

### 安装&配置需要的依赖
```
sudo yum install -y curl policycoreutils-python openssh-server perl
# 打开ssh访问和http、https端口
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

### 添加Gitlab仓库以及安装

#### 官方镜像
```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="http://192.168.200.130" yum install -y gitlab-ce
```

#### [国内镜像](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)

```
# vim /etc/yum.repos.d/gitlab-ce.repo
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

```
sudo yum makecache
sudo yum install gitlab-ce
```

```
gitlab-ctl reconfig
```

### 配置

> 更改Gitlab默认监听端口80

```
vim /var/opt/gitlab/nginx/conf/gitlab-http.conf
```

> 8080端口

```
vim /var/opt/gitlab/gitlab-rails/etc/puma.rb
```

### 常用命令


```
gitlab-ctl start
gitlab-ctl stop
```

### Runner

#### 安装

```
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permissions to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab CI user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

```
sudo gitlab-runner register --url http://192.168.200.131:10000/ --registration-token yQyjsMiKQzZvDULb44hx
```


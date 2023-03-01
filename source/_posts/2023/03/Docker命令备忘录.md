---
title: Docker命令备忘录
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-01 20:11:46
img:
coverImg:
password:
summary:
tags:
- command
categories: [Development & Operate,Docker]
---

收集一下不常用的命令待查  

# 通过容器生成镜像
```bash
docker commit -a "作者" -m "信息" containerId  imagename:version
```

### 通过root账号进入Docker 容器
```bash
docker exec -it --user root containerId bash
```

### Docker容器中的文件拷贝

```bash
docker cp CONTAINER ID:容器目录 本地目录
```

### centos 安装docker
```bash
curl -fsSL https://get.docker.com/ | sh
```

### 安装Docker compose
```bash
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.3.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```


### 查看Dockers磁盘占用
```bash
docker system df -v
```
---
title: Docker默认工作目录更改
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 22:24:01
img:
coverImg:
password:
summary:
tags:
- 环境配置
categories: [Development & Operate,Docker]
---


## 停掉docker服务

```bash
systemctl stop docker
```
## 创建新的数据目录 将旧数据拷贝至新的数据目录

```bash
mkdir /data/docker/lib
scp rp /var/lib/docker /data/docker/lib/
```
## 调整docker的数据目录路径

```bash
vim /etc/docker/daemon.json 
{
"registrymirrors": [
   "https://bxsfpjcb.mirror.aliyuncs.com"
],
"maxconcurrentdownloads": 10,
"logdriver": "jsonfile",
"loglevel": "warn",
"logopts": {
  "maxsize": "10m",
  "maxfile": "3"
  },
"insecureregistries":
	  ["127.0.0.1"],
"dataroot":"/data/docker/lib/docker"				#新路径
}
```
## 重启docker

```bash
systemctl disable docker
systemctl enable docker
systemctl daemonreload
systemctl restart docker
```
## 查看docker的数据目录是否调整成功

```bash
docker info
```


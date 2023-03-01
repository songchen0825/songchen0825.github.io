---
title: 解决 Failed to fetch binary-i386 Packages 404 Not Found 问题
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-01 20:56:11
img:
coverImg:
password:
summary:
tags:
- 疑难杂症
- Ubuntu
categories: [Development & Operate,Linux & Unit]
---

# 问题描述

关于局域网内Ubuntu客户端使用apt从局域网内Ubuntu镜像服务器拉取软件失败的问题。将Ubuntu客户端上/etc/apt/source.list中的链接改为局域网内镜像服务器的IP地址，但是在更新时系统报错。使用apt-mirror在[局域网]中配置了一台Ubuntu镜像服务器供局域网内其他设备使用，但是发现在Ubuntu终端上输入

```shell
Failed to fetch http://172.6.0.2/ubuntu/dists/jammy/main/binary-i386/Packages  404  Not Found [IP: 172.6.0.2 80]
```

# 修改统支持架构

## 1. 查看系统支持的其他架构

```shell
dpkg --print-foreign-architectures
---
i386 
```

## 2. 移除i386架构
```shell
  dpkg --remove-architecture i386
``` 

在移除过程中，终端可能会显示
```shell
dpkg: error: cannot remove architecture 'i386' currently in use by the database
```

这是因为有些软件是i386的，所以需要先移除i386架构的软件，然后再移除i386架构
```shell
apt-get remove .*:i386
dpkg --remove-architecture i386
```

移除完成之后，重新移除架构即可。
---
title: CentOS7防火墙操作方法
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-01 20:54:16
img:
coverImg:
password:
summary:
tags:
- 网络
- 环境配置
categories: [Development & Operate,Linux & Unit]
---


```bash
netstat -ntpl
```

我们可以用这个命令看下已经开放的端口，然后针对的是开放还是关闭。

## 第一、CentOS7 防火墙开启常见端口命令

1、安装Firewall命令：

```bash
yum install firewalld firewalld-config
```

2、Firewall开启常见端口命令

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --zone=public --add-port=22/tcp --permanent
firewall-cmd --zone=public --add-port=21/tcp --permanent
firewall-cmd --zone=public --add-port=53/udp --permanent
```

比如我们有单独自定义SSH端口或者服务器入口WEB端口，我们需要单独放行。

3、Firewall关闭常见端口命令

```bash
firewall-cmd --zone=public --remove-port=80/tcp --permanent
firewall-cmd --zone=public --remove-port=443/tcp --permanent
firewall-cmd --zone=public --remove-port=22/tcp --permanent
firewall-cmd --zone=public --remove-port=21/tcp --permanent
firewall-cmd --zone=public --remove-port=53/udp --permanent
```

我们也可以批量添加区间端口

```bash
firewall-cmd --zone=public --add-port=4400-4600/udp --permanent
firewall-cmd --zone=public --add-port=4400-4600/tcp --permanent
```

4、开启防火墙命令

```bash
systemctl start firewalld.service
```

5、重启防火墙命令

```bash
firewall-cmd --reload 或者 service firewalld restart
```

6、查看端口列表：

```bash
firewall-cmd --permanent --list-port
```

7、禁用防火墙

```bash
systemctl stop firewalld
```

8、查看状态

```bash
systemctl status firewalld或者 firewall-cmd --state
```

## 第二、如果还用的iptables

如果我们防火墙还是用的iptables，那命令稍微不同。

1、查看防火墙状态

```bash
service iptables status
```

2、暂时关闭防火墙

```bash
service iptables stop
```

3、永久关闭防火墙

```bash
chkconfig iptables off
```

4、重启防火墙

```bash
service iptables restart
```

5、开放指定端口

```bash
vi /etc/sysconfig/iptables
```

编辑文件，然后添加。

```bash
iptables -I INPUT -p tcp --dport 端口号 -j ACCEPT
```

保存配置

```bash
service iptables save
```

重启防火墙

```bash
service iptables restart
```

重启才可以生效。
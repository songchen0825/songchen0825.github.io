---
title: Kubernetes依靠NFS动态PV,PVC构建
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 20:32:27
img:
coverImg:
password:
summary:
tags:
- nfs
- pv
- pvc
categories: [Development & Operate,Kubernetes]
---


# 安装NFS 

## 所有节点安装nfs客户端

```shell
#所有节点安装nfs客户端
yum -y install nfs-utils
```

## 配置nfs 服务端 

```shell
# 安装nfs服务
yum install -y nfs-common nfs-utils rpcbind 


# 创建共享目录（nfs服务端）
mkdir /nfsdata{1..5}
chmod 777 /nfsdata{1..5}
chown nfsnobody /nfsdata{1..5}


# 添加共享配置
vim /etc/exports
#添加
/nfsdata1 *(rw,no_root_squash,no_all_squash,sync)
/nfsdata2 *(rw,no_root_squash,no_all_squash,sync)
/nfsdata3 *(rw,no_root_squash,no_all_squash,sync)
/nfsdata4 *(rw,no_root_squash,no_all_squash,sync)
/nfsdata5 *(rw,no_root_squash,no_all_squash,sync)


# 重启服务
systemctl restart rpcbind
systemctl restart nfs


# 查看共享
[root@k8s-node02 nfs]# showmount -e 192.168.1.22
Export list for 192.168.1.22

[root@minio2 centos]# exportfs -rv
exporting *:/home/data/nfs
```


# 通过 Helm 安装 nfs-provisioner

## 添加Helm存储库

```shell
helm repo add azure http://mirror.azure.cn/kubernetes/charts/
```
    
## 本地仓库搜

```shell
helm search repo nfs-client-provisioner
```
    
## 开始安装
    
```shell
helm install nfs-storage azure/nfs-client-provisioner \
--set nfs.server=192.168.0.98 \
--set nfs.path=/home/data/nfs \
--set storageClass.name=nfs-storage \
--set storageClass.defaultClass=true
```

说明：

-   nfs.server：nfs服务地址
-   nfs.path：nfs根目录
-   storageClass.name：存储类名称
-   storageClass.defaultClass：设为默认存储类

## 查看安装情况

```shell
kubectl get sc
```
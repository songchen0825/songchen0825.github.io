---
title: Helm安装Mongodb集群
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 19:24:55
img:
coverImg:
password:
summary:
tags:
- DevOps
- 虚拟化
- mongodb
categories: [Development & Operate,Kubernetes,Helm]
---

### 1. 添加 `bitnami` 的仓库

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 2. 查询 `MongoDB` 资源

```shell
$ helm repo update 
$ helm search repo mongodb
```

### 3. 拉取 `MongoDB chart` 到本地

```shell
$ mkdir /root/mongodb && cd /root/mongodb 
# 拉取 chart 到本地 /root/mongodb 目录 
$ helm pull bitnami/mongodb --version 12.1.11 $ tar -xvf mongodb-12.1.11.tgz 
$ cp mongodb/values.yaml ./values-test.yaml 

mongodb
├── Chart.lock
├── charts
│   └── common
├── Chart.yaml
├── README.md
├── templates
│   ├── arbiter
│   ├── common-scripts-cm.yaml
│   ├── configmap.yaml
│   ├── extra-list.yaml
│   ├── _helpers.tpl
│   ├── hidden
│   ├── initialization-configmap.yaml
│   ├── metrics-svc.yaml
│   ├── NOTES.txt
│   ├── prometheusrule.yaml
│   ├── psp.yaml
│   ├── replicaset
│   ├── rolebinding.yaml
│   ├── role.yaml
│   ├── secrets-ca.yaml
│   ├── secrets.yaml
│   ├── serviceaccount.yaml
│   ├── servicemonitor.yaml
│   └── standalone
├── values.schema.json
└── values.yaml
```

### 4. 对本地 `values-mogngo.yaml` 修改

```yaml
  

global:

  # 定义 storageClass 使用的类型

  storageClass: "nfs-storage"

  

# 定义 mongodb 集群为副本集模式

architecture: replicaset

  

# 启动集群认证功能，设置超级管理员账户密码

auth:

  enabled: true

  rootUser: root

  rootPassword: "root"

  

# 设置集群数量，3个

replicaCount: 3

  

# 定义 pod 的 nodeSelector

# nodeSelector: { "node": "middleware" }

  

# 启用持久化存储，使用 global.storageClass 自动创建 pvc

persistence:

  enabled: true

  size: 20Gi
```

### 5. 连接 `MongoDB 集群` 验证服务

```shell
mongosh admin --host "mongodb-cluster-0.mongodb-cluster-headless.middleware.svc.cluster.local:27017,mongodb-cluster-1.mongodb-cluster-headless.middleware.svc.cluster.local:27017,mongodb-cluster-2.mongodb-cluster-headless.middleware.svc.cluster.local:27017" --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD
```
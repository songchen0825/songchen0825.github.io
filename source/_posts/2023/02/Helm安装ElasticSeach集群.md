---
title: Helm安装ElasticSeach集群
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 21:06:01
img:
coverImg:
password:
summary:
tags:
- DevOps
- 虚拟化
- elasticsearch
- 全文检索
categories: [Development & Operate,Kubernetes,Helm]
---


# 添加 bitnami的仓库

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```


# 拉取bitnami/elasticsearch

```shell
helm pull bitnami/elasticsearch --version 18.2.9
```

# 本地value修改

```yaml
global:
  storageClass: "nfs-storage"
  imagePullSecrets: [regcred]
  elasticsearch:
    service:
      name: elasticsearch
      ports:
        restAPI: 9200
  kibanaEnabled: false
service:
  type: NodePort
master:
  replicaCount: 1
data:
  replicaCount: 3
coordinating:
  replicaCount: 1
ingest:
  replicaCount: 1
image:
  registry: 192.168.0.99:8084
  repository: yl-search-core
  pullPolicy: Always
  tag: 1.0.0
```

# 启动helm安装Elastic Search

```shell
helm install -f es-values.yaml es elasticsearch  --namespace middleware
```

---
title: Helm安装Minio集群
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 20:50:26
img:
coverImg:
password:
summary:
tags:
- DevOps
- 虚拟化
- minio
categories: [Development & Operate,Kubernetes,Helm]
---

1. 添加 helm 源：

```shell
helm repo add minio https://helm.min.io/
```
　　　　
2. 下载chart压缩包 ：

```shell
helm pull minio/minio
```
　　　　
3. 解压并按需创建values-minio.yaml

```shell
tar zxvf minio-8.0.10.tgz
vim values.yaml
```

编辑内容 

```yaml
resources:
  requests:
    memory: 1Gi
mode: distributed
```

4. 启动命令 

```shell
helm install minio --namespace middleware --create-namespace --set accessKey=minio,secretKey=minio123 --set mode=distributed --set service.type=NodePort --set persistence.enabled=true --set persistence.size=10Gi -f values-minio.yaml minio
```

---
title: Helm安装Zookeeper+Kafka
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 18:49:18
author:
img: 
coverImg: 
password:
summary:
tags:
- DevOps
- 虚拟化
- zookeeper
- kafka
categories: [Development & Operate,Kubernetes,Helm]
---


# 动态PV（后端为nfs存储）

```bash
[root@master ~]# kubectl get storageclass
NAME                    PROVISIONER   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-storage (default)   nfs-storage   Retain          Immediate           false                  31h
```



# 下载zookeeper ，kafka的helm包

添加bitnami仓库

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ali-incubator https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator/
```

列出添加的helm仓库

```bash
helm repo list
NAME         	URL                                                                      
bitnami      	https://charts.bitnami.com/bitnami                                       
ali-incubator	https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator/
```

查询helm包

```bash
helm search repo zookeeper
NAME             		CHART VERSION		APP VERSION	DESCRIPTION                                       
bitnami/zookeeper	6.0.0        			3.6.2      			A centralized service for maintaining configura...
bitnami/kafka    	12.2.4       			2.6.0      			Apache Kafka is a distributed streaming platform.
```

下载helm包到本地，在线安装可不用下载

```bash
[root@master zk]# helm pull bitnami/zookeeper
[root@master zk]# helm pull bitnami/kafka
[root@master zk]# clear
[root@master zk]# ls
kafka-12.2.4.tgz  zookeeper-6.0.0.tgz
```


# 安装zookeeper kafka

安装方法参考bitnami官网：https://docs.bitnami.com/tutorials/deploy-scalable-kafka-zookeeper-cluster-kubernetes/  

在线安装

```bash
# 安装zookeeper，可通过-n namaspace添加名称空间，因不暴露在公网，关闭了认证(--set auth.enabled=false)，并允许匿名访问，设置zookeeper副本为3
helm install zookeeper bitnami/zookeeper   --set replicaCount=3   --set auth.enabled=false   --set allowAnonymousLogin=true
# 安装kafka，取消自动创建zookeeper，使用刚刚创建的zookeeper，制定zookeeper的服务名称，
helm install kafka bitnami/kafka   --set zookeeper.enabled=false   --set replicaCount=3  --set externalZookeeper.servers=zookeeper
```


# 测试集群

进入kafka的pod创建一个topic

```bash
[root@master zk]# kubectl exec -it -n zk kafka-0 bash
I have no name!@kafka-0:/$ kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic testtopic
```

启动一个消费者

```bash
I have no name!@kafka-0:/$ kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic testtopic
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128175129821.png)  
新开一个窗口，进入kafka的pod，启动一个生产者，输入消息；在消费者端可以收到消息

```bash
[root@master zk]# kubectl exec -it -n zk kafka-0 bash 
I have no name!@kafka-0:/$ kafka-console-producer.sh --bootstrap-server kafka:9092 --topic mytopic
```
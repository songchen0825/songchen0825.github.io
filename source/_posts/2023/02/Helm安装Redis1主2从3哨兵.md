---
title: Helm安装Redis1主2从3哨兵
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 19:14:37
img:
coverImg:
password:
summary:
tags:
- DevOps
- 虚拟化
- redis
categories: [Development & Operate,Kubernetes,Helm]
---

# 添加 Helm 的仓库

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```


# 查询 `redis` 资源

```shell
$ helm repo update 
$ helm search repo redis 
NAME CHART VERSION APP VERSION DESCRIPTION 
bitnami/redis 16.11.2 6.2.7 Redis(R) is an open source, advanced key-value ... 
bitnami/redis-cluster 7.6.1 6.2.7 Redis(TM) is an open source, scalable, distribu...
```

# 拉取 `redis chart` 到本地

```shell
$ mkdir -p /root/redis/ && cd /root/redis/ 
# 拉取 chart 到本地 /root/redis/ 目录 
$ helm pull bitnami/redis --version 16.11.2 
$ tar -zxvf redis-16.11.2.tgz 
$ cp redis/values.yaml ./values-test.yaml 
# 查看当前目录层级 
$ tree -L 2
redis
├── Chart.lock
├── charts
│   └── common
├── Chart.yaml
├── img
│   ├── redis-cluster-topology.png
│   └── redis-topology.png
├── README.md
├── templates
│   ├── configmap.yaml
│   ├── extra-list.yaml
│   ├── headless-svc.yaml
│   ├── health-configmap.yaml
│   ├── _helpers.tpl
│   ├── master
│   ├── metrics-svc.yaml
│   ├── networkpolicy.yaml
│   ├── NOTES.txt
│   ├── pdb.yaml
│   ├── prometheusrule.yaml
│   ├── replicas
│   ├── rolebinding.yaml
│   ├── role.yaml
│   ├── scripts-configmap.yaml
│   ├── secret.yaml
│   ├── sentinel
│   ├── serviceaccount.yaml
│   ├── servicemonitor.yaml
│   └── tls-secret.yaml
├── values.schema.json
└── values.yaml
```

# 对本地 `values-test.yaml` 修改

修改配置 `values-test.yaml`

```yaml
global:
  # 全局定义 storageClass 使用 openebs
  storageClass: "openebs-jiva-default"
  # 定义 redis 集群认证密码
  redis:
    password: "redis123"


fullnameOverride: "redis"


# 定义集群的模式  Allowed values: `standalone` or `replication`
architecture: replication


# redis 服务配置定义
commonConfiguration: |-
  # Enable AOF https://redis.io/topics/persistence#append-only-file
  appendonly yes
  # Disable RDB persistence, AOF persistence already enabled.
  save ""


# master 节点配置信息
master:
  containerPorts:
    redis: 6379

  kind: StatefulSet

  persistence:
    enabled: true
    accessModes:
      - ReadWriteOnce
    size: 8Gi

  service:
    type: ClusterIP
    ports:
      redis: 6379


# replica 节点配置信息
replica:
  replicaCount: 3
  containerPorts:
    redis: 6379

  persistence:
    enabled: true
    storageClass: ""
    accessModes:
      - ReadWriteOnce
    size: 8Gi

  service:
    type: ClusterIP
    ports:
      redis: 6379


# sentinel 节点配置信息
sentinel:
  enabled: true
  containerPorts:
    sentinel: 26379

  persistence:
    enabled: true
    storageClass: ""
    accessModes:
      - ReadWriteOnce
    size: 100Mi

  service:
    type: ClusterIP
    ports:
      redis: 6379
      sentinel: 26379
```

# 安装 `redis` 集群

```shell
# 创建 test-middleware 名称空间 
$ kubectl create ns test-middleware 
# 安装 redis 集群 
$ helm -n test-middleware install redis redis -f values-test.yaml | tee test.log 
## helm -n NAMESAPCE install SERVER_NAME FILE_NAME -f CONFIG_FILE 
## -n 指定 kubernetes 集群名称空间 
## -f 指定使用的配置文件，文件中定义的配置可以覆盖 redis/values.yaml 文件中配置
```

# 查看部署的 `redis` 集群
```shell
$ helm -n test-middleware list 
NAME NAMESPACE REVISION UPDATED STATUS CHART APP VERSION 
redis test-middleware 1 2022-06-04 06:05:40.507941913 -0400 EDT deployed redis-16.11.2 6.2.7 

$ kubectl get pods --namespace=test-middleware 
NAME READY STATUS RESTARTS AGE 
redis-node-0 2/2 Running 0 3h45m 
redis-node-1 2/2 Running 0 3h44m 
redis-node-2 2/2 Running 0 3h42m
```

# 验证 redis 集群

```shell
# 登陆 redis 集群
$ export REDIS_PASSWORD=$(kubectl get secret --namespace test-middleware redis -o jsonpath="{.data.redis-password}" | base64 -d)

$ kubectl run --namespace test-middleware redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.7-debian-10-r23 --command -- sleep infinity

$ kubectl exec --tty -i redis-client --namespace test-middleware -- bash


# 查看 redis 主从集群
I have no name!@redis-client:/$ REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h redis -p 6379

redis:6379> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=redis-node-1.redis-headless.test-middleware.svc.cluster.local,port=6379,state=online,offset=4482538,lag=0
slave1:ip=redis-node-2.redis-headless.test-middleware.svc.cluster.local,port=6379,state=online,offset=4482538,lag=1
master_failover_state:no-failover
master_replid:8e48433d13ae59dbe2703a704504a98ecc61076a
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:4482538
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:3433963
repl_backlog_histlen:1048576


# 查看 Sentinel 节点
I have no name!@redis-client:/$ REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h redis-headless -p 26379

redis-headless:26379> info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=redis-node-0.redis-headless.test-middleware.svc.cluster.local:6379,slaves=2,sentinels=3
```
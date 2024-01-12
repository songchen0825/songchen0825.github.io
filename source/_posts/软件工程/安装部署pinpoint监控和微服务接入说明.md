---
title: 安装部署pinpoint监控和微服务接入说明
cover: false
date: 2024-01-05 16:06:18
tags:
categories:
top:
coverImg:
---
# 1. pinpoint是什么？

pinpoint是一款全链路分析工具，提供了无侵入式的调用链监控、方法执行详情查看、应用状态信息监控等功能。基于GoogleDapper论文进行的实现，与另一款开源的全链路分析工具[[Zipkin]]类似，但相比Zipkin提供了无侵入式、代码维度的监控等更多的特性。[[pinpoint]]支持的功能比较丰富，可以支持如下几种功能：

{% asset_img image.png 配置图 %}

{% asset_img image-1.png 配置图 %}


- 服务拓扑图：对整个系统中应用的调用关系进行了可视化的展示，单击某个服务节点，可以显示该节点的详细信息，比如当前节点状态、请求数量等•

- 实时活跃线程图：监控应用内活跃线程的执行情况，对应用的线程执行性能可以有比较直观的了解

- 请求响应散点图：以时间维度进行请求计数和响应时间的展示，拖过拖动图表可以选择对应的请求查看执行的详细情况

- 请求调用栈查看：对分布式环境中每个请求提供了代码维度的可见性，可以在页面中查看请求针对到代码维度的执行详情，帮助查找请求的瓶颈和故障原因。

- 应用状态、机器状态检查：通过这个功能可以查看相关应用程序的其他的一些详细信息，比如CPU使用情况，内存状态、垃圾收集状态，TPS和JVM信息等参数。
# 2. 架构组成

{% asset_img image-2.png 配置图 %}

Pinpoint 主要由 3 个组件外加 Hbase 数据库组成，三个组件分别为：Agent、Collector 和 Web UI。

- Agent组件：用于收集应用端监控数据，无侵入式，只需要在启动命令中加入部分参数即

- Collector组件：数据收集模块，接收Agent发送过来的监控数据，并存储到HBase

- [[WebUI]]：监控展示模块，展示系统调用关系、调用详情、应用状态等，并支持报警等功能

# 3. 安装与配置

根据公司开发环境进行部署，一台部署pinpoint的主程序，另外一台对微服务进行部署 ，配置环境：

|IP|OS|安装服务|描述|
|---|---|---|---|
|192.168.0.99|cenos7|pinpoint web，pinpoint collector，hbase|pinpoint web展示，逻辑控制，hbase存储|
|192.168.0.98|cenos7|micro-service，pinpoint-agent|微服务和代理|

## 3.1 安装Hbase

该服务可以使用[[docker]]快速构建和部署，以实现高效的测试和开发。通过使用本地磁盘进行数据存储，可以减少对额外硬件的依赖，并提高系统的灵活性。同时，使用内置的[[zookeeper]]服务可以更好地管理数据，加快数据访问速度并提高系统的可靠性。

在实际生产环境中，HDFS将被用于数据存储，因为它可以提供更高的数据容量和更好的容错能力。此外，在部署该服务时，还需要考虑到数据安全性和隐私保护等方面的问题，以确保系统的稳定性和可靠性。

### 3.1.1 搜索可用的hbase镜像

{% asset_img image-3.png 配置图 %}

### 3.1.2 启动hbase，需要暴露内置的zookeeper地址和数据采集地址

```shell
docker run -d -h myhbase --name pinpoint-hbase -v /tmp/spark:/tmp/spark -p 12181:2181 -p 19090:9090 -p 19095:9095 -p 16000:16000 -p 16010:16010 -p 16201:16201 -p 16301:16301 -p 16020:16020 -p 16030:16030 harisekhon/hbase
```

登录web，来查看[[HBase]]的数据是否初始化成功

{% asset_img image-4.png 配置图 %}

### 3.1.3 执行初始化脚本

{% asset_img image-5.png 配置图 %}

进入容器中执行解脚本

`/hbase/bin/hbase shell hbase-create.hbase`

## 3.2 安装pinpoint collector

从官方下载好启动程序，手动构建pinpoint collector镜像 ,需要修改hbase里zookeeper的位置地址

-Dpinpoint.zookeeper.address=192.168.0.99

-Dhbase.client.port=12181

```dockerfile
FROM openjdk:11

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN echo 'Asia/Shanghai' >/etc/timezone

ADD pinpoint-hbase2-collector-boot-2.5.2.jar app.jar

ENTRYPOINT ["java","-jar","-Dpinpoint.zookeeper.address=192.168.0.99","-Dhbase.client.port=12181","-Dserver.port=18888","/app.jar"]

EXPOSE 9991
EXPOSE 9992
EXPOSE 9993
EXPOSE 9994
EXPOSE 9995
EXPOSE 9996
EXPOSE 18888
```

> 由于hbase对zookeeper采用的是域名进行注册，所以启动的时候，需要把hbase的域名加到hosts里

```shell
docker build -t pinpoint-collector .
docker run -d --add-host=myhbase:192.168.0.99 -p 9991:9991 -p 9992:9992 -p 9993:9993 -p 9994:9994 -p 9995:9995 -p 9996:9996 -p 18888:18888 --name pinpoint-collector pinpoint-collector
```

## 3.3 安装pinpoint web

从官方下载好启动程序，手动构建pinpoint web镜像 ,需要修改hbase里zookeeper的位置地址

-Dpinpoint.zookeeper.address=192.168.0.99

-Dhbase.client.port=12181

```dockerfile
FROM openjdk:11

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN echo 'Asia/Shanghai' >/etc/timezone

ADD pinpoint-hbase2-web-boot-2.5.2.jar app.jar

ENTRYPOINT ["java","-jar","-Dpinpoint.zookeeper.address=192.168.0.99","-Dhbase.client.port=12181","-Dserver.port=18889","/app.jar"]

EXPOSE 9997
EXPOSE 18889
```

验证部署是否成功

{% asset_img image-6.png 配置图 %}

# 4. 应用接入，

本地镜像已经完成构建，仅需要修改项目中构建镜像的Dockerfile即可,对比过去仅需要修改两处

1. 依赖的构建镜像修改成micro-service-base
2. 添加启动参数,需要修改pinpoint.agentId，pinpoint.applicationName保持整个微服务集群的唯一性。

"-javaagent:/home/pinpoint-agent-2.5.2/pinpoint-bootstrap.jar","-Dprofiler.sampling.counting.sampling-rate=1","-Dpinpoint.agentId=yl-service-user-center","-Dpinpoint.applicationName=yl-service-user-center","-Dprofiler.transport.grpc.collector.ip=192.168.0.99"

```dockerfile
FROM micro-service-base

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN echo 'Asia/Shanghai' >/etc/timezone

COPY *-server/target/*-SNAPSHOT.jar app.jar

ENTRYPOINT ["java","-javaagent:/home/pinpoint-agent-2.5.2/pinpoint-bootstrap.jar","-Dprofiler.sampling.counting.sampling-rate=1","-Dpinpoint.agentId=yl-service-user-center","-Dpinpoint.applicationName=yl-service-user-center","-Dprofiler.transport.grpc.collector.ip=192.168.0.99","-jar","/app.jar"]
```

验证调用结果 ：

{% asset_img image-7.png 配置图 %}

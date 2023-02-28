---
title: 跨namespace访问SVC
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 19:34:48
img:
coverImg:
password:
summary:
tags:
- 网络
categories: [Development & Operate,Kubernetes]
---



在K8S中，同一个命名空间（namespace）下的服务之间调用，之间通过服务名（service name）调用即可。不过在更多时候，我们可能会将一些服务单独隔离在一个命名空间中（比如我们将中间件服务统一放在 middleware 命名空间中，将业务服务放在 business 命名空间中）。  
遇到这种情况，我们就需要跨命名空间访问，K8S 对service 提供了四种不同的类型，针对这个问题我们选用 `ExternalName` 类型的 service 即可。

> k8s service 分为四种类型，分别为：
> 
> -   ClusterIp（默认类型，每个Node分配一个集群内部的Ip，内部可以互相访问，外部无法访问集群内部）
> -   NodePort（基于ClusterIp，另外在每个Node上开放一个端口，可以从所有的位置访问这个地址）
> -   LoadBalance（基于NodePort，并且有云服务商在外部创建了一个负载均衡层，将流量导入到对应Port。要收费的，一般由云服务商提供，比如阿里云、AWS等均提供这种服务）
> -   ExternalName（将外部地址经过集群内部的再一次封装，实际上就是集群DNS服务器将CNAME解析到了外部地址上，实现了集群内部访问）

本文使用 `ExternalName` 实现我们的需求：  
1、首先创建一个 `ExternalName` 类型的 service  
2、然后通过 `{SERVICE_NAME}.{NAMESPACE_NAME}.svc.cluster.local` 这样的格式，访问目标 namespace 下的服务。

以文初所述的 middleware 和 business 为例：

-   `middleware` 命名空间下有一个服务 `middleware01`，middleware01 自己对外的 service_name 为 `svc-middleware01`
-   `business` 命名空间下有一个服务 `app01`，app01 自己对外的 service_name 为 `svc-app01`
-   在 `business` 命名空间下创建一个 `service`，名称为 `svc-middleware01-external`，设定 service 的 `type: ExternalName、externaleName: middleware01.middleware.svc.cluster.local`
-   那么，在 `app01` 中访问 `middleware01` 服务中的 `xxxApi` 地址则为：`http://svc-middleware01-external.middleware.svc.cluster.local:{PORT}/xxxApi`

最最后，我想告诉你的是：这个 `svc-middleware01-external` 你可以当做多此一举，完全可以直接使用 `{SERVICE_NAME}.{NAMESPACE_NAME}.svc.cluster.local` 跨命名空间访问。  
之所以做这一步，是因为一般来说其他命名空间的服务在当前我们自己命名空间中，我们希望名称统一。这样如果我们多个服务都使用同一个外部命名空间的服务时如果外部命名空间的服务名变更了，我们只需要修改一个地方即可。

---

下面再给一个同命名空间下 `Stateful Set` 通过内部 host 名称访问的例子（命名空间为 `demo`）：

1、创建了一个名称为 `eureka-server` 的 `Statefule Set`，数量设置3个副本，K8s 会自从创建3个Pod `eureka-server-0`、`eureka-server-1`、`eureka-server-2`。  
2、创建一个 `Headless` 类型的 Service，名称为 `svc-eureka-server`。  
3、则同命名空间下访问这3个 eureka-server Pod 的 host 如下：

```bash
http://eureka-server-0.svc-eureka-server:8761/eureka/
http://eureka-server-1.svc-eureka-server:8761/eureka/
http://eureka-server-2.svc-eureka-server:8761/eureka/
```

或者

```bash
http://eureka-server-0.svc-eureka-server.demo.svc.cluster.local:8761/eureka/
http://eureka-server-1.svc-eureka-server.demo.svc.cluster.local:8761/eureka/
http://eureka-server-2.svc-eureka-server.demo.svc.cluster.local:8761/eureka/
```

对比以上两种格式差异，下面一种多了一段 `demo.svc.cluster.local`，结合上文的内容，给出结论为 **“同一个命名空间中访问最后那一段可以省略”**。
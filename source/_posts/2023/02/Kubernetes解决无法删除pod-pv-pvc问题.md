---
title: Kubernetes解决无法删除pod,pv,pvc问题
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 20:39:49
img:
coverImg:
password:
summary:
tags:
- 疑难杂症
categories: [Development & Operate,Kubernetes]
---


# 删除PV 

```shell
kubectl patch pvc kafka-kafka-2 -p '{"metadata":{"finalizers":null}}' -n micro-service
```

# 删除pod

```shell 
kubectl delete pod xxx -n xxx --force --grace-period=0
```


如果强制删除还不行，设置 finalizers 为空
```shell
kubectl patch pod xxx -n xxx -p '{"metadata":{"finalizers":null}}'
```


尽管此操作很简单，但其他因素可能会干扰删除，包括 finalizers 和 owner references。

## K8s 中对象删除基本流程如下

（1）客户端提交删除请求到 API Server（可选传递 GracePeriodSeconds 参数）

（2）API Server 做 Graceful Deletion 检查（若对象实现了 RESTGracefulDeleteStrategy 接口，会调用对应的实现并返回是否需要进行 Graceful 删除）

（3）API Server 检查 Finalizers 并结合是否需要进行 graceful 删除，来决定是否立即删除对象（若对象需要进行 graceful 删除，更新 metadata.DeletionGracePeriodSecond 和metadata.DeletionTimestamp 字段，不从存储中删除对象；若对象不需要进行 Graceful 删除时：metadata.Finalizers 为空，直接删除。metadata.Finalizers 不为空，不删除，只更新 metadata.DeletionTimestamp。

## finalizers介绍

Finalizers 字段属于 Kubernetes GC 垃圾收集器，是一种删除拦截机制，能够让控制器实现异步的删除前（Pre-delete）回调。其存在于任何一个资源对象的 Meta中，在 k8s 源码中声明为 []string，该 Slice 的内容为需要执行的拦截器名称。

  

常见用途：

（1）控制器在对象刚创建时，在 metadata.finalizers 写入一个自定义字符串

（2）APIServer 在 metadata.finalizers 数组不为空时，不会删除对象。此逻辑是硬性，对所有对象生效。

（3）当有客户端删除对象时，控制器可以发现对象出于删除状态，然后执行相应的 pre-delete 逻辑。执行完成后，将之前写入的自定义字符串移除。

（4）当所有控制器都将各自写入到 metadata.finalizers 的字符串移除后。API Server 就自动将对象删除。

注：若对象同时实现了 graceful 删除策略，删除请求需要满足 GracefulPeriodSeconds = 0 条件

## Graceful Deletion

APIServer 在处理 Pod 删除请求时，会根据 pod 的 pod.Spec.TerminationGracePeriodSeconds 和 deleteOptions.GracefulPeriodSeconds 综合判断，是否进行 Graceful 删除。若是，并判断最终的 GracefulPeriodSeconds 该设置为多少。

（1）当删除请求选项中有设置 GracefulPeriodSeconds，以选项中为准。若没有，使用 pod.Spec.TerminationGracePeriodSeconds 。若 pod.Spec.TerminationGracePeriodSeconds 也没有设置，使用默认值 0 。

（2）当 Pod 没有调度，或者已经结束（无论成功还是失败）。GracePeriodSeconds 都重置为 0.

GracePeriodSeconds 为 0 表示不进行优雅删除。非 0 表示进行优雅删除。Pod 的默认优雅删除时间为 30 s ，在对象创建时配置在 pod.Spec.TerminationGracePeriodSeconds 字段。

优雅删除的目的是给予 Kubelet 一定时间对 Pod 实行优雅退出。在用户对 Pod 执行 Delete 操作时，Pod 对象不会立即从 API Server 删除，而只是进入 Termination 阶段。Kubelet 会对运行中的 Pod 的容器发送 TERM 信号，通知其退出。并执行用户配置的 preStop hooks 逻辑。当优雅时间过了之后，再开始使用 KILL 信号尝试强制杀容器。

当 kubelet 将 Pod 清理干净后，就会使用 GracefulPeriodSeconds==0 的参数执行删除操作。

因为 Pod 实现优雅删除目的是为了给予 Kubelet 时间做资源清理操作，所以这也是为什么在设置 GracePeriodSeconds 阶段，若 Pod 没有被调度或者已经退出，也就可以直接允许立即删除 (GracePeriodSeconds = 0)。

注： 理解该流程可以结合 Kubernetes 官网的 Pod Termination 说明：[https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/workloads/pods/pod/%23termination-of-pods) 对象第一次执行优雅删除操作时，会将当时 GracefulPeriodSeconds 配置在 metadata.DeletionGracefulPeriodSeconds 字段。 Deletion 操作在 API Server 是不可回退的操作。metadata.DeletionTimestamp 设置后不可更改，metadata.DeletionGracefulPeriodSeconds 只能减小，不能增加。

## 对象无法删除的原因

在了解以上机制后，对象无法删除无外乎以下两个原因：

对象存在 finalizers，关联的控制器故障未能执行或执行 finalizer 函数卡住

比如namespace控制器无法删除完空间内所有的对象，特别是在使用 aggregated apiserver 时，第三方 apiserver 服务故障导致无法删除其对象。此时，需要会恢复第三方 apiserver 服务或移除该 apiserver 的聚合，具体选择哪种方案需根据实际情况而定。

  

集群内安装的控制器给一些对象增加了自定义 finalizers ，未删除完 fianlizers 就下线了该控制器，导致这些 fianlizers 没有控制器来移除他们。此时，需要恢复该控制器会手动移除 finalizers，具体选择哪种方案根据实际情况而定。

对象需要优雅删除，但执行者不能完成删除。比如 Pod 因为 kubelet 无法下线节点上 node 容器、存储卷而无法删除。比较常见有以下原因：

-   kubelet 无法通过 container runtime 杀死进程。比如进程进入 D (Uninterruptible) 状态，container runtime 或操作内核遇到 bug 等。对于进程进入 D 状态，若能恢复照成 D 的故障，比如恢复关联的外设访问等，能解决问题。若不能，或者是因为后者内核 bug，一般并不能走正常流程让 kubelet 杀死进程。一般需要重启操作系统才能解决。
-   kubelet 进程停止或者 node 失去联系。 该情况下，并没有 kubelet 运行或者运行中的 kubelet 与 apiserver 已经断开，无法收到 pod 需要删除下线的消息。所以，没有执行者来进行优雅删除。该情况下，需要恢复节点以及 kubelet 的运行即可。

两种机制都有强制跳过的方案：

-   删除 finalizers ，让关联的逻辑不需要执行
-   kubelet delete --force --grace-period 0 直接删除

但应仅当关联清理工作已经不重要或已手动执行时，才可选择。不然容易照成数据、状态不一致等。
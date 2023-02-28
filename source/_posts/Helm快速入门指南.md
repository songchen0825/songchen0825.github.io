---
title: Helm快速入门指南
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 17:46:44
author:
img:
coverImg:
password:
summary:
tags:
- DevOps
- 虚拟化
categories: [Development & Operate,Kubernetes,Helm]
---

# 1. 快速安装

**Helm** 帮助您管理 Kubernetes 应用程序 ——Helm Charts 帮助您定义、安装和升级最复杂的 Kubernetes 应用程序。

在线文档 ： [Helm | Docs](https://helm.sh/zh/docs/)

## 1.1 先决条件

想成功和正确地使用[[Helm]]，需要以下前置条件。

1.  一个 Kubernetes 集群
2.  确定你安装版本的安全配置
3.  安装和配置Helm。

## 1.2 安装或者使用现有的Kubernetes集群

使用Helm，需要一个Kubernetes集群。对于Helm的最新版本，我们建议使用Kubernetes的最新稳定版， 在大多数情况下，它是倒数第二个次版本。

## 1.3 使用源码Source 安装 (Linux, macOS)

从源码构建Helm的工作要稍微多一点，但如果你想测试最新（预发布）的Helm版本，这是最好的方式。

您必须有可用的Go环境。

```shell
$ git clone https://github.com/helm/helm.git
$ cd helm
$ make
$ mv bin/helm /user/local/bin 
```

验证Helm 安装是否成功 
 
![[Pasted image 20230202171107.png]]


# 2.  入门 Chart

在本指南中我们会创建一个名为`mychart`的chart，然后会在chart中创建一些模板。

```console
$ helm create mychart
Creating mychart
```

快速查看 `mychart/templates/`

如果你看看 `mychart/templates/` 目录，会注意到一些文件已经存在了：

-   `NOTES.txt`: chart的"帮助文本"。这会在你的用户执行`helm install`时展示给他们。
-   `deployment.yaml`: 创建Kubernetes [工作负载](https://kubernetes.io/docs/user-guide/deployments/)的基本清单
-   `service.yaml`: 为你的工作负载创建一个 [service终端](https://kubernetes.io/docs/user-guide/services/)基本清单。
-   `_helpers.tpl`: 放置可以通过chart复用的模板辅助对象

然后我们要做的是... _把它们全部删掉！_ 这样我们就可以从头开始学习我们的教程。我们在开始时会创造自己的`NOTES.txt`和`_helpers.tpl`。

```console
$ rm -rf mychart/templates/*
```

编制生产环境级别的chart时，有这些chart的基础版本会很有用。因此在日常编写中，你可能不想删除它们。

## 2.1 第一个模板

第一个创建的模板是`ConfigMap`。Kubernetes中，配置映射只是用于存储配置数据的对象。其他组件，比如pod，可以访问配置映射中的数据。

因为配置映射是基础资源，对我们来说是很好的起点。

让我们以创建一个名为 `mychart/templates/configmap.yaml`的文件开始：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

**提示:** 模板名称不遵循严格的命名模式。但是建议以`.yaml`作为YAML文件的后缀，以`.tpl`作为helper文件的后缀。

上述YAML文件是一个简单的配置映射，构成了最小的必需字段。因为文件在 `mychart/templates/`目录中，它会通过模板引擎传递。

像这样将一个普通YAML文件放在`mychart/templates/`目录中是没问题的。当Helm读取这个模板时会按照原样传递给Kubernetes。

有了这个简单的模板，现在有一个可安装的chart了。现在安装如下：

```shell
$ helm install full-coral ./mychart
NAME: full-coral
LAST DEPLOYED: Tue Nov  1 17:36:01 2016
NAMESPACE: default
STATUS: DEPLOYED
REVISION: 1
TEST SUITE: None
```

## 2.2 验证脚本 

```shell
helm install --generate-name --dry-run --debug mychart
```

![[Pasted image 20230203093400.png]]


# 国内helm添加常用charts仓库
```shell
# 查看当前配置的仓库地址
    
$ helm repo list
    
# 删除默认仓库，默认在国外pull很慢
    
$ helm repo remove stable
    
# 添加几个常用的仓库,可自定义名字

$ helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
    
$ helm repo add kaiyuanshe http://mirror.kaiyuanshe.cn/kubernetes/charts
    
$ helm repo add azure http://mirror.azure.cn/kubernetes/charts
    
$ helm repo add dandydev https://dandydeveloper.github.io/charts
    
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```
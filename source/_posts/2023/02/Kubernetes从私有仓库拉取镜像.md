---
title: Kubernetes从私有仓库拉取镜像
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 20:21:45
img:
coverImg:
password:
summary:
tags:
- docker
- 环境配置
categories: [Development & Operate,Kubernetes]
---

## 准备开始

-   你必须拥有一个 Kubernetes 的集群，同时你的 Kubernetes 集群必须带有 kubectl 命令行工具。 建议在至少有两个节点的集群上运行本教程，且这些节点不作为控制平面主机。 如果你还没有集群，你可以通过 [Minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) 构建一个你自己的集群，或者你可以使用下面任意一个 Kubernetes 工具构建：
    
    -   [Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes)
    -   [玩转 Kubernetes](http://labs.play-with-k8s.com/)

-   要进行此练习，你需要 `docker` 命令行工具和一个知道密码的 [Docker ID](https://docs.docker.com/docker-id/)。
-   如果你要使用不同的私有的镜像仓库，你需要有对应镜像仓库的命令行工具和登录信息。

## 登录 Docker 镜像仓库

在个人电脑上，要想拉取私有镜像必须在镜像仓库上进行身份验证。

使用 `docker` 命令工具来登录到 Docker Hub。 更多详细信息，请查阅 [Docker ID accounts](https://docs.docker.com/docker-id/#log-in) 中的 **log in** 部分。

```shell
docker login
```

当出现提示时，输入你的 Docker ID 和登录凭证（访问令牌、 或 Docker ID 的密码）。

登录过程会创建或更新保存有授权令牌的 `config.json` 文件。 查看 [Kubernetes 中如何解析这个文件](https://kubernetes.io/zh-cn/docs/concepts/containers/images#config-json)。

查看 `config.json` 文件：

```shell
cat ~/.docker/config.json
```

输出结果包含类似于以下内容的部分：

```json
{
    "auths": {
        "https://index.docker.io/v1/": {
            "auth": "c3R...zE2"
        }
    }
}
```

**说明：**

如果使用 Docker 凭证仓库，则不会看到 `auth` 条目，看到的将是以仓库名称作为值的 `credsStore` 条目。 在这种情况下，你可以直接创建一个 Secret。请参阅[在命令行上提供凭证来创建 Secret](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)。

## 创建一个基于现有凭证的 Secret

Kubernetes 集群使用 `kubernetes.io/dockerconfigjson` 类型的 Secret 来通过镜像仓库的身份验证，进而提取私有镜像。

如果你已经运行了 `docker login` 命令，你可以复制该镜像仓库的凭证到 Kubernetes:

```shell
kubectl create secret generic regcred \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson
```

如果你需要更多的设置（例如，为新 Secret 设置名字空间或标签）， 则可以在存储 Secret 之前对它进行自定义。 请务必：

-   将 data 项中的名称设置为 `.dockerconfigjson`
-   使用 base64 编码方法对 Docker 配置文件进行编码，然后粘贴该字符串的内容，作为字段 `data[".dockerconfigjson"]` 的值
-   将 `type` 设置为 `kubernetes.io/dockerconfigjson`

示例：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
  namespace: awesomeapps
data:
  .dockerconfigjson: UmVhbGx5IHJlYWxseSByZWVlZWVlZWVlZWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGx5eXl5eXl5eXl5eXl5eXl5eXl5eSBsbGxsbGxsbGxsbGxsbG9vb29vb29vb29vb29vb29vb29vb29vb29vb25ubm5ubm5ubm5ubm5ubm5ubm5ubm5ubmdnZ2dnZ2dnZ2dnZ2dnZ2dnZ2cgYXV0aCBrZXlzCg==
type: kubernetes.io/dockerconfigjson
```

如果你收到错误消息：`error: no objects passed to create`， 这可能意味着 base64 编码的字符串是无效的。如果你收到类似 `Secret "myregistrykey" is invalid: data[.dockerconfigjson]: invalid value ...` 的错误消息，则表示数据中的 base64 编码字符串已成功解码， 但无法解析为 `.docker/config.json` 文件。

## 在命令行上提供凭证来创建 Secret

创建 Secret，命名为 `regcred`：

```shell
kubectl create secret docker-registry regcred \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>
```

在这里：

-   `<your-registry-server>` 是你的私有 Docker 仓库全限定域名（FQDN）。 DockerHub 使用 `https://index.docker.io/v1/`。
-   `<your-name>` 是你的 Docker 用户名。
-   `<your-pword>` 是你的 Docker 密码。
-   `<your-email>` 是你的 Docker 邮箱。

这样你就成功地将集群中的 Docker 凭证设置为名为 `regcred` 的 Secret。

**说明：** 在命令行上键入 Secret 可能会将它们存储在你的 shell 历史记录中而不受保护， 并且这些 Secret 信息也可能在 `kubectl` 运行期间对你 PC 上的其他用户可见。

## 检查 Secret `regcred`

要了解你创建的 `regcred` Secret 的内容，可以用 YAML 格式进行查看：

```shell
kubectl get secret regcred --output=yaml
```

输出和下面类似：

```yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJodHRwczovL2luZGV4L ... J0QUl6RTIifX0=
kind: Secret
metadata:
  ...
  name: regcred
  ...
data:
  .dockerconfigjson: eyJodHRwczovL2luZGV4L ... J0QUl6RTIifX0=
type: kubernetes.io/dockerconfigjson
```

`.dockerconfigjson` 字段的值是 Docker 凭证的 base64 表示。

要了解 `dockerconfigjson` 字段中的内容，请将 Secret 数据转换为可读格式：

```shell
kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
```

输出和下面类似：

```json
{"auths":{"your.private.registry.example.com":{"username":"janedoe","password":"xxxxxxxxxxx","email":"jdoe@example.com","auth":"c3R...zE2"}}}
```

要了解 `auth` 字段中的内容，请将 base64 编码过的数据转换为可读格式：

```shell
echo "c3R...zE2" | base64 --decode
```

输出结果中，用户名和密码用 `:` 链接，类似下面这样：

```none
janedoe:xxxxxxxxxxx
```

注意，Secret 数据包含与本地 `~/.docker/config.json` 文件类似的授权令牌。

这样你就已经成功地将 Docker 凭证设置为集群中的名为 `regcred` 的 Secret。

## 创建一个使用你的 Secret 的 Pod

下面是一个 Pod 配置清单示例，该示例中 Pod 需要访问你的 Docker 凭证 `regcred`：

[`pods/private-reg-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/pods/private-reg-pod.yaml) ![](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy pods/private-reg-pod.yaml to clipboard")

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred

```

将上述文件下载到你的计算机中：

```shell
curl -L -o my-private-reg-pod.yaml https://k8s.io/examples/pods/private-reg-pod.yaml
```

在 `my-private-reg-pod.yaml` 文件中，使用私有仓库的镜像路径替换 `<your-private-image>`，例如：

```none
your.private.registry.example.com/janedoe/jdoe-private:v1
```

要从私有仓库拉取镜像，Kubernetes 需要凭证。 配置文件中的 `imagePullSecrets` 字段表明 Kubernetes 应该通过名为 `regcred` 的 Secret 获取凭证。

创建使用了你的 Secret 的 Pod，并检查它是否正常运行：

```shell
kubectl apply -f my-private-reg-pod.yaml
kubectl get pod private-reg
```

---
title: Kubernetes修改NodePort的端口范围
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-02-28 20:38:06
img:
coverImg:
password:
summary:
tags:
- 网络
categories: [Development & Operate,Kubernetes]
---


在 [[Kubernetes]] 集群中，NodePort 默认范围是 30000-32767，某些情况下，因为您所在公司的网络策略限制，您可能需要修改 NodePort 的端口范围，本文描述了具体的操作方法。

下面的配置是基于 kubeadm 安装的集群  

# Step 1修改kube-apiserver.yaml文件

修改service-node-port-range=20000-22767

`vim /etc/kubernetes/manifests/kube-apiserver.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.0.42:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.0.42
    - --allow-privileged=true
    - --anonymous-auth=True
    - --apiserver-count=3
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --audit-log-path=/var/log/audit/kube-apiserver-audit.log
    - --audit-policy-file=/etc/kubernetes/audit-policy/apiserver-audit-policy.yaml
    - --authorization-mode=Node,RBAC
    - --bind-address=0.0.0.0
    - --client-ca-file=/etc/kubernetes/ssl/ca.crt
    - --default-not-ready-toleration-seconds=300
    - --default-unreachable-toleration-seconds=300
    - --enable-admission-plugins=NodeRestriction
    - --enable-aggregator-routing=False
    - --enable-bootstrap-token-auth=true
    - --endpoint-reconciler-type=lease
    - --etcd-cafile=/etc/ssl/etcd/ssl/ca.pem
    - --etcd-certfile=/etc/ssl/etcd/ssl/node-node1.pem
    - --etcd-keyfile=/etc/ssl/etcd/ssl/node-node1-key.pem
    - --etcd-servers=https://192.168.0.41:2379,https://192.168.0.42:2379,https://192.168.0.43:2379
    - --event-ttl=1h0m0s
    - --kubelet-client-certificate=/etc/kubernetes/ssl/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/ssl/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalDNS,InternalIP,Hostname,ExternalDNS,ExternalIP
    - --profiling=False
    - --proxy-client-cert-file=/etc/kubernetes/ssl/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/ssl/front-proxy-client.key
    - --request-timeout=1m0s
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/ssl/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/ssl/sa.pub
    - --service-account-lookup=True
    - --service-account-signing-key-file=/etc/kubernetes/ssl/sa.key
    - --service-cluster-ip-range=10.233.0.0/16
    - --service-node-port-range=30000-32767
    - --storage-backend=etcd3
    - --tls-cert-file=/etc/kubernetes/ssl/apiserver.crt
    - --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256
    - --tls-private-key-file=/etc/kubernetes/ssl/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.25.5
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.0.42
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.168.0.42
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 30
      httpGet:
        host: 192.168.0.42
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/log/audit
      name: audit-logs
    - mountPath: /etc/kubernetes/audit-policy
      name: audit-policy
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/ssl/etcd/ssl
      name: etcd-certs-0
      readOnly: true
    - mountPath: /etc/kubernetes/ssl
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /var/log/kubernetes/audit
      type: ""
    name: audit-logs
  - hostPath:
      path: /etc/kubernetes/audit-policy
      type: ""
    name: audit-policy
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/ssl/etcd/ssl
      type: DirectoryOrCreate
    name: etcd-certs-0
  - hostPath:
      path: /etc/kubernetes/ssl
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: ""
    name: usr-share-ca-certificates
status: {}
root@node2:~# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/ssl/etcd/ssl
      type: DirectoryOrCreate
    name: etcd-certs-0
  - hostPath:
      path: /etc/kubernetes/ssl
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: ""
    name: usr-share-ca-certificates
status: {}
```

# Step 2 重启apiserver

## 获得 apiserver 的 pod 名字

```shell
export apiserver_pods=$(kubectl get pods --selector=component=kube-apiserver -n kube-system --output=jsonpath={.items…metadata.name})
```

## 删除 apiserver 的 pod

```shell
kubectl delete pod $apiserver_pods -n kube-system
```

# Step 3 验证结果

执行以下命令，验证修改是否生效：
`kubectl describe pod $apiserver_pods -n kube-system`

输出结果如下所示：（此时，我们可以看到，apiserver 已经使用新的命令行参数启动）  

```shell
执行以下命令，验证修改是否生效：

kubectl describe pod $apiserver_pods -n kube-system
输出结果如下所示：（此时，我们可以看到，apiserver 已经使用新的命令行参数启动）

Host Port:
Command:
kube-apiserver
–advertise-address=172.17.216.80
–service-node-port-range=20000-22767
State: Running
Started: Mon, 11 Nov 2019 21:31:39 +0800
Ready: True
Restart Count: 0
Requests:
cpu: 250m
```

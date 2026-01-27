---
title: Kubernetes 容器运行节点指定与调度控制
date: 2025-11-13 10:25:01
tags: 
  - Docker
  - Kubernetes
  - Yaml
top: false
categories: 
  - 性能优化
  - K8s节点调度
img: /medias/featureimages/13.jpg

---


## 📅 日期

2025-11-13 02:23:24

## 📘 背景

在 Kubernetes 环境中，为了在集群中合理地调度 Pod，运维工程师往往需要**将某些容器限定运行在特定节点上**，例如：

- 把数据库 Pod 放在 SSD 存储节点；
- 把 GPU 任务放在具备 GPU 的节点；
- 把日志采集或监控服务放在边缘节点上。

Kubernetes 提供了多种方式实现这一目标。

---

## 🧩 一、使用 nodeSelector（最简单常用）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: nginx
    image: nginx:latest
  nodeSelector:
    kubernetes.io/hostname: k8s-node3
```

📘 **说明：**

- `nodeSelector` 用来指定 Pod 只能调度到匹配该标签的节点；
- 每个节点都有默认标签：`kubernetes.io/hostname=<节点主机名>`。

---

## 🧩 二、使用 nodeName（强制绑定）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  nodeName: k8s-node3
  containers:
  - name: nginx
    image: nginx
```

📘 **说明：**

- `nodeName` 是**硬绑定**，不会经过调度器。
- 如果节点不可用或资源不足，Pod 将一直处于 `Pending` 状态。

---

## 🧩 三、使用 Node Affinity（节点亲和性）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - k8s-node3
      containers:
      - name: nginx
        image: nginx:latest
```






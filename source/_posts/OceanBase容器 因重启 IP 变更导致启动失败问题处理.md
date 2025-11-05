---
title: OceanBase容器 因重启 IP 变更导致启动失败问题处理
date: 2025-09-11 14:27:22
tags: 
  - Docker
  - OceanBase
top: false
categories: 
  - 容器网络
img: /medias/featureimages/17.jpg
---
# OceanBase容器 因重启 IP 变更导致启动失败问题处理

## 🧩 报错信息

当执行 OceanBase-CE 容器启动命令时出现如下错误：

```
[ERROR] Traceback (most recent call last):
[ERROR]   File "core.py", line 2104, in start_cluster
[ERROR]   File "core.py", line 2148, in _start_cluster
[ERROR]   File "core.py", line 1011, in cluster_status_check
[ERROR]   File "core.py", line 194, in call_plugin
[ERROR]   File "core.py", line 260, in get_clients
[ERROR]   File "core.py", line 279, in get_clients_with_connect_status
[ERROR]   File "core.py", line 315, in ssh_clients_connect
[ERROR]   File "_stdio.py", line 956, in func_wrapper
[ERROR]   File "ssh.py", line 443, in connect
[ERROR]   File "_stdio.py", line 956, in func_wrapper
[ERROR]   File "ssh.py", line 400, in _login
[ERROR]   File "paramiko/client.py", line 368, in connect
[ERROR] paramiko.ssh_exception.NoValidConnectionsError: [Errno None] Unable to connect to port 22 on 172.18.0.6
[ERROR] 
[CRITICAL] [ERROR] OBD-1013: root@172.18.0.6 connect failed: time out
[INFO] [ERROR] OBD-1013: root@172.18.0.6 connect failed: time out
```

## 🧠 问题原因

* OceanBase-CE 容器重启后，原本分配的 IP（如 `172.18.0.6`）被其他容器占用。
* OceanBase 启动脚本尝试通过旧 IP 连接 SSH（端口 22），但由于 IP 变化，连接超时。

最终导致报错：

> `OBD-1013: root@172.18.0.6 connect failed: time out`

---

## ⚙️ 解决步骤

### 1️⃣ 查看容器网络与子网配置

执行以下命令查看容器所属网络：

```bash
docker network ls -q | xargs docker network inspect | grep -E '(Name|Subnet)'
```

确认 OceanBase 容器（如 `oceanbase_db`）属于哪个网络。

---

### 2️⃣ 查看目标网络下的 IP 占用情况

例如，OceanBase 属于 `root_default` 网络，则执行：

```bash
docker network inspect root_default
```

找到 `172.18.0.6` 当前被哪个容器占用。

---

### 3️⃣ 停止占用该 IP 的容器

```bash
docker stop <容器ID>
```

确认释放 IP 地址。

---

### 4️⃣ 停止 OceanBase 容器

```bash
docker stop oceanbase_db
```

---

### 5️⃣ 断开 OceanBase 容器与原网络的连接

```bash
docker network disconnect root_default oceanbase_db
```

---

### 6️⃣ 为 OceanBase 容器重新分配固定 IP

将 OceanBase 重新连接至网络并指定原来的 IP：

```bash
docker network connect --ip 172.18.0.6 root_default oceanbase_db
```

---

### 7️⃣ 启动容器

```bash
docker start oceanbase_db
```

验证容器启动正常，执行：

```bash
docker ps
```

查看容器状态应为 `Up`。

---

## ✅ 建议

1. **为 OceanBase 设置固定 IP 或自定义网络**：防止因容器重启导致 IP 漂移。
2. **使用 `docker-compose` 或自定义 `bridge` 网络**，可通过 `ipam` 配置静态分配。
3. **容器依赖关系启动顺序可通过 `depends_on` 指定**，避免网络初始化先后造成 IP 冲突。

---

**参考命令速览：**

```bash
# 查看容器网络
docker network ls -q | xargs docker network inspect | grep -E '(Name|Subnet)'

# 检查网络 IP 占用
docker network inspect root_default

# 停止冲突容器
docker stop <container_id>

# 断开 OceanBase 网络连接
docker network disconnect root_default oceanbase_db

# 为 OceanBase 指定 IP 并重新连接
docker network connect --ip 172.18.0.6 root_default oceanbase_db

# 启动 OceanBase 容器
docker start oceanbase_db
```

---

✅ **结论：**

> 该问题本质上是容器网络 IP 冲突导致的连接超时。通过释放原 IP、断开重连、固定 IP 分配，可彻底解决。

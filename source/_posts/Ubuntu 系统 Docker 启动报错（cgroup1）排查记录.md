---
title: Ubuntu 系统 Docker 启动报错（cgroup1）
date: 2025-11-11 13:55:02
tags: 
  - Linux
  - Ubantu
  - Docker
top: false
categories: 
  - 踩坑
  - 困难解决
img: /medias/featureimages/12.jpg

---

## 🧭 一、问题背景

在 Ubuntu 系统上安装 Docker 后执行启动命令：

```bash
sudo systemctl start docker
```

发现 Docker 无法启动，日志中出现以下报错：

```
failed to mount cgroup: cgroup1: not found
error while starting daemon: error initializing graphdriver
```

起初怀疑是 Docker 配置文件或内核兼容性问题。

---

## 🔍 二、问题分析

### 1️⃣ Docker 与 cgroup 的关系

Docker 依赖 Linux **cgroup（control group）** 机制来管理容器资源。  
目前 Linux 支持两种版本：

| 名称      | 说明             | 默认启用版本         |
| --------- | ---------------- | -------------------- |
| cgroup v1 | 传统分层管理     | Ubuntu 18.04 及以前  |
| cgroup v2 | systemd 统一管理 | Ubuntu 22.04 / 24.04 |

Ubuntu 22.04 及以上版本默认启用 **cgroup v2**，  
而部分旧版 Docker 默认仍尝试使用 **cgroup v1**，  
导致启动时出现 `cgroup1 not found` 错误。

---

## 🧩 三、问题复现

执行以下命令：

```bash
sudo systemctl status docker
```

得到输出：

```
dockerd: failed to start daemon: error while starting daemon: 
failed to mount cgroup: cgroup1 not found
```

再查看系统 cgroup 版本：

```bash
stat -fc %T /sys/fs/cgroup/
```

输出：

```
cgroup2fs
```

确认系统正在使用 **cgroup v2**。

---

## 🧰 四、解决方案

### ✅ 方案一：让 Docker 使用 systemd（推荐）

1️⃣ 编辑 Docker 配置文件：

```bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```

2️⃣ 写入以下配置：

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```

3️⃣ 重启 Docker：

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker
```

4️⃣ 验证启动结果：

```bash
sudo systemctl status docker
```

输出中出现：

```
Active: active (running)
```

✅ 表示 Docker 已成功启动。

---

### ✅ 方案二：回退到 cgroup v1（兼容旧系统）

适用于旧版 Kubernetes 或内核较老的环境。

1️⃣ 修改 GRUB 参数：

```bash
sudo nano /etc/default/grub
```

找到：

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

修改为：

```
GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0"
```

2️⃣ 更新 grub 并重启系统：

```bash
sudo update-grub
sudo reboot
```

3️⃣ 验证：

```bash
cat /proc/filesystems | grep cgroup
```

输出：

```
nodev   cgroup
```

✅ 表示已回退为 cgroup v1。

---

## 🧠 五、验证结果

执行：

```bash
docker info | grep -i cgroup
```

输出：

```
Cgroup Driver: systemd
Cgroup Version: 2
```

✅ 表示 Docker 已正确与系统 cgroup 机制对齐。

---

---

✍️ **结论：**

> 该问题属于 Ubuntu 22+ 与 Docker 之间的 cgroup 驱动不匹配问题。  
> 推荐使用 systemd 驱动统一管理，配置一次即可长期稳定运行。
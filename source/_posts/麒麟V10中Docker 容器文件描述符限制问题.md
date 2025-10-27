---
title: 麒麟V10中Docker 容器文件描述符限制问题
date: 2024-04-02 10:14:56
tags: 
  - Docker
  - 麒麟V10
  - Java
top: false
categories: 
  - 困难解决
  - 国产操作系统
img: /medias/featureimages/17.jpg

---
# 🧩 Docker 容器文件描述符限制问题排查解决
## 一、问题描述

在麒麟V10 版本中运行 Java 应用的 Docker 容器中，系统日志中出现以下异常或告警信息：

```
Too many open files
```

或在高并发场景下，应用无法正常建立新连接、文件读写失败、线程池异常退出。  
这是由于 **容器内文件描述符（ulimit nofile）限制过低** 导致的。

默认情况下，Docker 容器的 `nofile` 限制可能仅为 **1024**，远低于高并发 Java 应用所需的文件句柄数量。

---

## 二、问题分析

### 1️⃣ 文件描述符的作用

Linux 中每个打开的文件、网络套接字、管道、设备等都占用一个 **文件描述符（FD）**。  
Java 应用在高并发场景下（如 Web 服务、数据库连接池、日志写入等）会大量使用 FD。

当 FD 数量超过系统限制时，程序会抛出：

```
java.io.FileNotFoundException: Too many open files
```

或系统日志中出现：

```
EMFILE: Too many open files
```

---

### 2️⃣ 容器与宿主机的限制关系

Docker 容器继承宿主机的部分资源限制，但其内部的 ulimit 参数可以独立配置。

常见问题来源：

- Docker Daemon 未配置全局 ulimit；
- 容器启动时未显式声明；
- 容器内 `/etc/security/limits.conf` 无效；
- Compose 或 Kubernetes 未传递 ulimit 参数。

---

## 三、解决思路

1. **从根本上提高 Docker 容器可用文件描述符上限；**  
2. **确保所有 Java 应用容器在启动时均继承该配置。**

---

## 四、解决步骤

### ✅ 1. 修改 Docker Daemon 全局配置

编辑 `/etc/docker/daemon.json`：

```json
{
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 262144,
      "Soft": 262144
    }
  },
  "oom-score-adjust": -500
}
```

> 说明：
>
> - `default-ulimits`：设置所有容器的默认资源限制；
> - `nofile`：文件描述符数量；
> - `oom-score-adjust`：降低 Docker 守护进程被 OOM Killer 杀死的概率。

保存后执行：

```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

验证是否生效：

```bash
docker info | grep ulimits -A 3
```

---

### ✅ 2. 在 docker-compose.yml 中为 Java 容器单独设置限制

在对应服务中添加：

```yaml
services:
  my-java-app:
    image: my-java-app:latest
    container_name: java-app
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
```

> 说明：
>
> - `soft`：警告阈值，应用可动态调整；
> - `hard`：最大限制，应用无法超出；
> - 建议两者保持一致，确保最大性能。

重新启动容器：

```bash
docker-compose down
docker-compose up -d
```

验证：

```bash
docker exec -it java-app bash
ulimit -n
```

应显示：

```
262144
```

---

### ✅ 3. 针对单容器启动方式（docker run）

如果未使用 Compose，可直接在命令行中设置：

```bash
docker run -d   --name java-app   --ulimit nofile=262144:262144   my-java-app:latest
```

---

## 五、验证结果

执行以下命令查看容器内当前限制：

```bash
docker exec -it java-app bash
ulimit -a
```

输出应包含：

```
open files                      (-n) 262144
```

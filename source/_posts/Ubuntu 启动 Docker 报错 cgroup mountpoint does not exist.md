---
title: Ubuntu 启动 Docker 报错 cgroup mountpoint does not exist
date: 2025-10-27 10:58:49
tags: 
  - Linux
  - Ubantu
top: false
categories: 
  - 踩坑
  - 困难解决
img: /medias/featureimages/11.jpg

# Ubuntu 启动 Docker 报错 cgroup mountpoint does not exist


**系统环境：** Ubuntu 20.04+\
**问题现象：**\
执行 `docker-compose up` 或启动 PostgreSQL 容器时报错：Cannot start service postgreSQL: cgroups: cgroup mountpoint does not exist: unknown

------------------------------------------------------------------------

## 一、问题原因

出现该错误的根本原因是： \> **当前系统的 Docker
版本过低，无法正确识别或挂载 cgroups 控制组。**

在新版 Ubuntu（20、22、24 LTS）中，系统默认使用 **cgroup v2**，而旧版
Docker 对此支持不完善，会导致无法启动容器。

------------------------------------------------------------------------

## 二、解决方案

### ✅ 升级 Docker 到匹配版本

  Ubuntu 版本       推荐 Docker Engine 版本

----------------- -------------------------

  Ubuntu 20.X LTS   20.10.x 或更高版本
  Ubuntu 22.X LTS   20.10.x 或更高版本
  Ubuntu 24.X LTS   20.10.x 或 23.0.x

**升级命令示例：**

``` bash
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> 升级后重新启动 Docker：

``` bash
sudo systemctl restart docker
```

------------------------------------------------------------------------

## 三、验证修复

运行：

``` bash
docker run hello-world
```

若能正常输出欢迎信息，即表明 Docker 服务及 cgroup 挂载点已恢复正常。

------------------------------------------------------------------------

## 四、经验总结

1.  Ubuntu 系统升级后应同时关注 **Docker 版本兼容性**。\
2.  当出现 `cgroup mountpoint does not exist` 报错时，先确认 Docker
    版本是否支持当前内核的 cgroup 模式。\
3.  在生产环境中可提前规划 Docker
    与内核版本匹配，避免容器无法启动的问题。
---
title: Keepalived+Nginx 高可用负载均衡
date: 2025-07-19 19:10:11
tags: 
  - Nginx
  - Keepalived
top: false
categories: 
  - 高可用
  - 负载均衡
  - 反向代理
img: /medias/featureimages/19.jpg
---
# Nginx + Keepalived 高可用负载均衡详解

## 🧩 一、Nginx + Keepalived 的作用

### 🎯 目标：实现 **高可用反向代理 + 负载均衡**

单台 Nginx 即使做了负载均衡，但本身仍然是 **单点故障 (Single Point of Failure)**。
如果这台 Nginx 挂掉，整个网站仍然不可用。

于是，引入 **Keepalived** 来实现主备热切换：

```
用户请求 → 虚拟IP(VIP)
                ↓
        ┌───────────────┐
        │ Keepalived+Nginx 主节点 │ ← 正常工作
        └───────────────┘
                ↓（主机宕机时自动切换）
        ┌───────────────┐
        │ Keepalived+Nginx 备节点 │ ← 自动接管VIP
        └───────────────┘
```

### ✅ 效果

- 主节点宕机时，备节点**自动接管 IP**
- Nginx 服务**无感切换**
- 用户继续访问同一个 **VIP（虚拟 IP）**，不会中断

---

## ⚙️ 二、工作原理

Keepalived 基于 **VRRP（Virtual Router Redundancy Protocol）虚拟路由冗余协议** 实现：

- 多台服务器共享一个虚拟 IP（VIP）
- 只有 **Master 节点** 拥有该 IP
- 当 Master 掉线，**Backup 节点** 自动绑定 VIP，继续提供服务

Nginx 只需绑定在本机 IP 上服务即可，不需要感知切换。

---

## 🧱 三、核心配置讲解

### 🔹 Keepalived 主节点配置 `/etc/keepalived/keepalived.conf`

```bash
! Configuration File for keepalived

global_defs {
    router_id NGINX_MASTER
}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 3
    weight -5
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 123456
    }

    virtual_ipaddress {
        192.168.1.100
    }

    track_script {
        chk_nginx
    }
}
```

### 🔹 备节点配置修改

```bash
router_id NGINX_BACKUP
state BACKUP
priority 90
```

### 🔹 Nginx 配置

```nginx
upstream backend {
    server 192.168.1.10;
    server 192.168.1.11;
}

server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://backend;
    }
}
```

### 🔹 检查脚本 `/etc/keepalived/check_nginx.sh`

```bash
#!/bin/bash
nginx_status=$(ps -C nginx --no-header | wc -l)
if [ $nginx_status -eq 0 ]; then
    systemctl start nginx
    sleep 2
    nginx_status=$(ps -C nginx --no-header | wc -l)
    if [ $nginx_status -eq 0 ]; then
        systemctl stop keepalived
    fi
fi
```

---

## 🧰 四、常用命令

| 操作                     | 命令                                    |
| ------------------------ | --------------------------------------- |
| 检查 Keepalived 配置语法 | `keepalived -t`                         |
| 启动服务                 | `systemctl start keepalived`            |
| 停止服务                 | `systemctl stop keepalived`             |
| 查看日志                 | `journalctl -u keepalived -f`           |
| 查看 VIP                 | `ip addr show`                          |
| 模拟宕机                 | `systemctl stop nginx` 或 `ifdown eth0` |

---

## 🧪 五、典型应用场景

| 场景               | 描述                                     | 结果                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 🖥️ 网站前端高可用   | 两台 Nginx + Keepalived 组成主备         | 主机宕机，流量自动切到备机      |
| 🧮 API 网关高可用   | 多台 API Gateway 使用同一 VIP 出口       | 防止单节点 API 挂掉导致服务中断 |
| ☁️ 私有云出口       | Keepalived 维护浮动 IP，自动切换流量路由 | 提高云服务容灾能力              |
| 💾 数据节点反向代理 | Redis、Elasticsearch 前面加高可用代理层  | 读写请求统一入口                |

---

## 🧠 六、流程总结

```
[Client]
   ↓ 访问 VIP (192.168.1.100)
   ↓
[Keepalived Master] 持有 VIP
   ↓
[Nginx Master] 反向代理 + 负载均衡
   ↓
[后端集群] 多台应用服务器

(当 Master 掉线)
   ↓
[Keepalived Backup] 自动接管 VIP
   ↓
[Nginx Backup] 无缝继续服务
```

---

## ✅ 七、总结

> **Nginx 负责分流，Keepalived 负责切换。**
>
> - Nginx 提供 **反向代理 + 负载均衡**
> - Keepalived 提供 **高可用 + IP 漂移**
> - 二者结合，实现 **高可用负载均衡集群**
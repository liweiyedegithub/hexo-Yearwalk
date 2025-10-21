---
title: Keepalived 安装与双机高可用配置
date: 2024-06-09 10:23:32
tags: 
  - Linux
  - Keepalived
top: false
categories: 
  - 高可用
img: /medias/featureimages/14.jpg

---
# Keepalived 安装与双机高可用配置

## 一、Keepalived 简介

Keepalived 是一个基于 VRRP 协议实现的高可用解决方案，主要用于 IP 漂移和负载均衡。它能够检测服务器状态并在主备服务器之间自动切换虚拟 IP (VIP)。

## 二、环境准备

### 系统要求
- 两台 Linux 服务器（主备各一台）
- 相同子网内的静态 IP 地址
- root 或 sudo 权限

### 网络拓扑示例
```
主服务器 (Master): 192.168.1.100
备服务器 (Backup): 192.168.1.101
虚拟 IP (VIP): 192.168.1.200
```

## 三、安装 Keepalived

### 在 CentOS/RHEL 上安装

```bash
# 在两台服务器上执行
yum install -y keepalived
systemctl enable keepalived
```

### 在 Ubuntu/Debian 上安装

```bash
# 在两台服务器上执行
apt-get update
apt-get install -y keepalived
systemctl enable keepalived
```

## 四、配置 Keepalived

### 主服务器配置 (Master)

创建或编辑配置文件 `/etc/keepalived/keepalived.conf`：

```conf
! Configuration File for keepalived

global_defs {
    router_id LVS_MASTER  # 唯一标识，建议使用主机名
}

vrrp_instance VI_1 {
    state MASTER          # 主服务器设置为MASTER
    interface eth0        # 监听的网络接口
    virtual_router_id 51  # 虚拟路由ID，主备必须相同
    priority 100          # 优先级(1-255)，主服务器应高于备服务器
    
    advert_int 1          # 检查间隔(秒)
    
    authentication {
        auth_type PASS    # 认证类型
        auth_pass 1234    # 认证密码，主备必须相同
    }
    
    virtual_ipaddress {
        192.168.1.200/24  # 虚拟IP地址
    }
    
    # 可选：添加跟踪脚本检测应用状态
    track_script {
        chk_nginx
    }
}
```

### 备服务器配置 (Backup)

创建或编辑配置文件 `/etc/keepalived/keepalived.conf`：

```conf
! Configuration File for keepalived

global_defs {
    router_id LVS_BACKUP  # 唯一标识，建议使用主机名
}

vrrp_instance VI_1 {
    state BACKUP          # 备服务器设置为BACKUP
    interface eth0        # 监听的网络接口
    virtual_router_id 51  # 必须与主服务器相同
    priority 90           # 优先级低于主服务器
    
    advert_int 1          # 检查间隔(秒)
    
    authentication {
        auth_type PASS    # 认证类型
        auth_pass 1234    # 认证密码，与主服务器相同
    }
    
    virtual_ipaddress {
        192.168.1.200/24  # 虚拟IP地址
    }
    
    # 可选：添加跟踪脚本检测应用状态
    track_script {
        chk_nginx
    }
}
```

## 五、可选：配置健康检查

### 示例：Nginx 健康检查脚本

在两台服务器上创建检查脚本 `/etc/keepalived/check_nginx.sh`：

```bash
#!/bin/bash
if ! pgrep nginx > /dev/null; then
    exit 1
fi
exit 0
```

设置执行权限：

```bash
chmod +x /etc/keepalived/check_nginx.sh
```

在配置文件中添加跟踪脚本定义（主备都需要）：

```conf
vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2    # 检查间隔(秒)
    weight -20    # 检查失败时优先级降低值
    fall 2        # 连续失败多少次认为服务不可用
    rise 1        # 连续成功多少次认为服务恢复
}
```

## 六、启动服务

在两台服务器上执行：

```bash
systemctl start keepalived
systemctl status keepalived
```

## 七、验证配置

### 检查虚拟 IP

在主服务器上执行：

```bash
ip addr show eth0
```

应该能看到虚拟 IP `192.168.1.200` 绑定在主服务器的 `eth0` 接口上。

### 测试故障转移

1. 在主服务器上停止 keepalived 服务：
   ```bash
   systemctl stop keepalived
   ```

2. 在备服务器上检查是否获得了虚拟 IP：
   ```bash
   ip addr show eth0
   ```

3. 恢复主服务器后，虚拟 IP 应该会自动切换回来。

## 八、日志查看

查看 keepalived 日志：

```bash
journalctl -u keepalived -f
```

## 九、防火墙配置

如果启用了防火墙，需要允许 VRRP 协议：

```bash
# CentOS/RHEL
firewall-cmd --add-rich-rule='rule protocol value="vrrp" accept' --permanent
firewall-cmd --reload

# Ubuntu/Debian
ufw allow proto vrrp
```

## 十、常见问题排查

1. **虚拟 IP 不漂移**
   - 检查两台服务器的 `virtual_router_id` 是否相同
   - 检查认证密码是否一致
   - 检查网络是否互通，VRRP 多播是否被阻止

2. **服务频繁切换**
   - 检查网络延迟是否过高
   - 调整 `advert_int` 参数增加检查间隔

3. **日志中出现认证失败**
   - 确保主备服务器的 `auth_type` 和 `auth_pass` 完全一致

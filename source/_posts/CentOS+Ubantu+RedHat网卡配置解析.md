---
title: CentOS+Ubantu+RedHat网卡配置解析
date: 2023-11-17 09:58:44
tags: 
  - Linux
  - CentOS
  - Ubantu
  - RedHat
top: false
categories: 
  - 网卡配置
  - 网络接口配置
img: /medias/featureimages/11.jpg

---
# Linux 网络接口配置文件参数解析

以下是网络接口配置文件（如 `/etc/sysconfig/network-scripts/ifcfg-ens192`）中各参数的详细解释：

## 基本网络类型配置

| 参数 | 值 | 说明 |
|------|----|------|
| `TYPE` | `Ethernet` | 指定网络接口类型为以太网 |
| `PROXY_METHOD` | `none` | 不使用代理方法 |
| `BROWSER_ONLY` | `no` | 不仅限于浏览器使用 |

## IP地址配置

| 参数 | 值 | 说明 |
|------|----|------|
| `BOOTPROTO` | `none` | 禁用自动获取IP（`dhcp`为自动获取，`static`或`none`为静态IP） |
| `IPADDR` | `192.168.3.113` | 设置的静态IPv4地址 |
| `PREFIX` | `24` | 子网掩码位数（等同于`NETMASK=255.255.255.0`） |
| `GATEWAY` | `192.168.3.1` | 默认网关地址 |

## IPv4相关配置

| 参数 | 值 | 说明 |
|------|----|------|
| `DEFROUTE` | `yes` | 将此接口设置为默认路由 |
| `IPV4_FAILURE_FATAL` | `no` | IPv4配置失败不会导致接口禁用 |

## IPv6相关配置

| 参数 | 值 | 说明 |
|------|----|------|
| `IPV6INIT` | `yes` | 启用IPv6支持 |
| `IPV6_AUTOCONF` | `yes` | 启用IPv6自动配置 |
| `IPV6_DEFROUTE` | `yes` | 将此接口设置为IPv6默认路由 |
| `IPV6_FAILURE_FATAL` | `no` | IPv6配置失败不会导致接口禁用 |
| `IPV6_ADDR_GEN_MODE` | `stable-privacy` | IPv6地址生成模式（增强隐私保护） |
| `IPV6_PRIVACY` | `no` | 禁用IPv6隐私扩展 |

## 设备标识配置

| 参数 | 值 | 说明 |
|------|----|------|
| `NAME` | `ens192` | 网络接口名称（逻辑名） |
| `UUID` | `3f0c3cea-0b9a-477d-a985-1c7f38007f48` | 网络接口的唯一标识符 |
| `DEVICE` | `ens192` | 网络接口设备名（物理名） |

## 启动与DNS配置

| 参数 | 值 | 说明 |
|------|----|------|
| `ONBOOT` | `yes` | 系统启动时自动激活此接口 |
| `DNS1` | `192.168.3.1`<br>`8.8.8.8` | 主DNS服务器地址（注意此处有重复定义，实际生效的是最后一个） |
| `DNS2` | `114.114.114.114` | 备用DNS服务器地址 |

## 配置注意事项



1. **完整配置示例**：
   ```ini
   TYPE="Ethernet"
   PROXY_METHOD="none"
   BROWSER_ONLY="no"
   BOOTPROTO="none"
   DEFROUTE="yes"
   IPV4_FAILURE_FATAL="no"
   IPV6INIT="yes"
   IPV6_AUTOCONF="yes"
   IPV6_DEFROUTE="yes"
   IPV6_FAILURE_FATAL="no"
   IPV6_ADDR_GEN_MODE="stable-privacy"
   NAME="ens192"
   UUID="3f0c3cea-0b9a-477d-a985-1c7f38007f48"
   DEVICE="ens192"
   ONBOOT="yes"
   IPADDR="192.168.3.113"
   PREFIX="24"
   GATEWAY="192.168.3.1"
   DNS1="8.8.8.8"
   DNS2="114.114.114.114"
   IPV6_PRIVACY="no"
   ```
2. **修改配置后，需要重启网络生效**：
   ```ini
   systemctl restart network
   service network restart 
   ```
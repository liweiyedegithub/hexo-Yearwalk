---
title: tags
date: 2024-04-19 14:42:21
tags: 
  - Docker
  - OceanBase
top: false
categories: 
  - 数据库快速部署
---
# 使用Dokcer快速搭建OceanBase-CE版本
***安装前提需要已安装docker、docker-compose服务*** [docker、docker-compose在线、离线安装](https://blog.csdn.net/weixin_45494811)
## 在线加载镜像
``` bash
#新建OceanBase-CE的docker-compose.yml文件，添加以下内容
version: '3'
services:
  oceanbase:
    image: oceanbase/oceanbase-ce:latest
    container_name: oceanbase_db
    environment:
      - OB_ROOT_PASSWORD=123456789  # 设置 OceanBase 的 root 用户密码
    ports:
      - "2881:2881"  # 将 OceanBase 的默认 SQL 端口 2881 映射到主机
      - "2882:2882"  # 将 OceanBase 的默认 RPC 端口 2882 映射到主机
    volumes:
      - oceanbase_data:/root/ob  # 持久化 OceanBase 数据
    restart: always
volumes:
  oceanbase_data:

```
## 离线加载镜像  
需要将镜像手动加载
[天翼云盘-OceanBase-CE 镜像下载连接](https://cloud.189.cn/t/6VB7ZfZviMBv（访问码：rkc3）)
``` bash
#加载镜像
docker load -i oceanbase.tar
#为镜像打tag
docker tag d29ae3214fa3  mysql:8.0.27
```
## 镜像加载后启动
``` bash
#启动容器
docker-compose up -d 
#进入容器
docker exec -it oceanbase_db bash
obclient -uroot -h127.1 -P2881
ALTER USER 'root' IDENTIFIED BY '123456789';
```
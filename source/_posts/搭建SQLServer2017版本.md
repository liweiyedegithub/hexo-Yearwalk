---
title: tags
date: 2020-05-04 10:40:14
tags: 
  - Docker
  - SQLServer
top: false
categories: 
  - 数据库快速部署
---
# 使用Dokcer快速搭建SQLServer 2017版本
***安装前提需要已安装docker、docker-compose服务*** [docker、docker-compose在线、离线安装](https://blog.csdn.net/weixin_45494811)
## 在线加载镜像
``` bash
#新建SQLServer的docker-compose.yml文件，添加以下内容
version: '3'
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2017-latest
    container_name: sqlserver_db
    environment:
      ACCEPT_EULA: "Y"  # 必须设置为 "Y" 以接受 SQL Server 的许可条款
      SA_PASSWORD: "SQLServer@1001"  # 设置 SA 用户的密码
    ports:
      - "1433:1433"  # 将 SQL Server 的默认端口 1433 映射到主机
    volumes:
      - sqlserver_data:/var/opt/mssql  # 持久化 SQL Server 数据
    restart: always
volumes:
  sqlserver_data:
# SA/SQLServer@1001 连接
```
## 离线加载镜像  
需要将镜像手动加载
[天翼云盘-SQLServer 2017 镜像下载连接](https://cloud.189.cn/t/ZnqMj2zUZVza（访问码：w6zk）)
``` bash
#加载镜像
docker load -i SQLServer.tar
#为镜像打tag
docker tag fa4c9a7ae374  mcr.microsoft.com/mssql/server:2017-latest
```
## 镜像加载后启动
``` bash
#启动容器
docker-compose up -d 
# SA/SQLServer@1001 连接
```
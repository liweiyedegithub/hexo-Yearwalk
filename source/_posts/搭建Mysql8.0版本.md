---
title: 使用Dokcer快速搭建Mysql 8.0版本
date: 2024-03-18 16:22:11
tags: 
  - Docker
  - Mysql
top: false
categories: 
  - 数据库快速部署
---
# 使用Dokcer快速搭建Mysql 8.0版本
***安装前提需要已安装docker、docker-compose服务*** [docker、docker-compose在线、离线安装](https://blog.csdn.net/weixin_45494811)
## 在线加载镜像
``` bash
#新建Mysql的docker-compose.yml文件，添加以下内容
version: '3'
services:
  mysql:
    image: mysql:8.0.27
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: 123456789
      MYSQL_DATABASE: testdb
      MYSQL_USER: cquser
      MYSQL_PASSWORD: 123456789
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    restart: always
volumes:
  mysql_data:

```
## 离线加载镜像  
需要将镜像手动加载
[天翼云盘-Mysql8.0 镜像下载连接](https://cloud.189.cn/t/vumAryyyYjau（访问码：b1xp）)
``` bash
#加载镜像
docker load -i Mysql8.0.tar
#为镜像打tag
docker tag 3218b38490ce  mysql:8.0.27
```
## 镜像加载后启动
``` bash
#启动容器
docker-compose up -d 
# root 123456789 连接
```
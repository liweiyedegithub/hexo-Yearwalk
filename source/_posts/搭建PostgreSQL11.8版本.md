---
title: 使用Dokcer快速搭建PostgreSQL 11.8版本
date: 2024-03-18 17:32:11
tags: 
  - Docker
  - PostgreSQL
top: false
categories: 
  - 数据库快速部署
img: /medias/featureimages/5.jpg
---

# 使用Dokcer快速搭建PostgreSQL 11.8版本
***安装前提需要已安装docker、docker-compose服务*** [docker、docker-compose在线、离线安装](https://blog.csdn.net/weixin_45494811)
## 在线加载镜像
``` bash
#新建postgreSQL的docker-compose.yml文件，添加以下内容
version: '3'
services:
  postgreSQL:
    image: postgres:11.8
    restart: always
    container_name: postgreSQL
    environment:
      POSTGRES_PASSWORD: postgreSQL@
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - ./pg/data/pgdata:/var/lib/postgresql/data/pgdata
    ports:
     - "5432:5432"
    logging:
      driver: "json-file"
      options:
        max-size: "100m"

```
## 离线加载镜像  
需要将镜像手动加载
[天翼云盘-postgreSQL 11.8 镜像下载连接](https://cloud.189.cn/t/eUrqQzQ7Zni2（访问码：6dks）)
``` bash
#加载镜像
docker load -i postgres_118_x86.tar
#为镜像打tag
docker tag 31d22a1554df  postgres:11.8
```
## 镜像加载后启动
``` bash
#启动容器
docker-compose up -d 
# postgres/postgreSQL@ 连接
```
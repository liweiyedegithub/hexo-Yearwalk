---
title: docker安装oracle11g
date: 2020-05-04 10:40:14
tags: 
  - Docker
  - Oracle
  - 数据库
top: false
categories: 
  - 技术
  - 疑难杂症
---
# 使用Dokcer快速搭建DB2 V10.5版本
***安装前提需要已安装docker、docker-compose服务*** [docker、docker-compose在线、离线安装](https://blog.csdn.net/weixin_45494811)
## 在线加载镜像
``` bash
#新建oracle的docker-compose.yml文件，添加以下内容
version: '3'
services:
  oracle:
    image: registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g:latest
    container_name: oracle_db
    environment:
      ORACLE_PWD: "123456789"  # 设置 Oracle 的 SYS 用户密码
    ports:
      - "1521:1521"  # 将 Oracle 的默认端口 1521 映射到主机
    volumes:
      - oracle_data:/opt/oracle/oradata  # 持久化 Oracle 数据
    restart: always
volumes:
  oracle_data:

```
## 离线加载镜像
需要将registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g:latest镜像手动加载
``` bash
#加载镜像
docker load -i oracle_11g.tar
#为镜像打tag
docker tag obca  registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g:latest
```
## 镜像加载后启动
``` bash
#启动容器
docker-compose up -d 
#进入容器
docker exec -it oracle_db bash
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
export ORACLE_SID=helowin
sqlplus / as sysdba
ALTER USER SYS IDENTIFIED BY "123456789";
```

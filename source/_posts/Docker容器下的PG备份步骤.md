---
title: Docker容器下的PG备份步骤
date: 2024-10-18 10:44:18
tags: 
  - Docker
  - PostgreSQL
top: false
categories: 
  - 数据库备份与恢复
  - 定时备份
img: /medias/featureimages/16.jpg

---
# PostgreSQL逻辑备份操作指南

## 1. 文档目的

本文档主要说明如何对 CloudQuery Base 2.1.4（包含）及以上版本中的 PostgreSQL 和 LDAP 进行逻辑备份。该版本采用了 OpenLDAP 和 PostgreSQL 的存储机制。

## 2. 备份方案概述

本方案实现以下自动化备份功能：
- 每日 23:05 执行 PostgreSQL 数据库逻辑备份
- 每日 23:30 自动清理 30 天前的历史备份文件
- 支持备份文件管理和存储空间维护

## 3. 操作步骤

### 3.1. 环境准备

#### 创建备份目录
```bash
# 创建备份目录（可根据实际需求修改路径）
mkdir -p /opt/ldif
cd /opt/ldif
```

### 3.2. 备份脚本配置

#### 创建备份脚本 `cronbackup.sh`
```bash
vi /opt/ldif/cronbackup.sh
```

#### 脚本内容
```bash
#!/bin/bash
# CloudQuery 定时备份脚本
# 功能：备份 PostgreSQL 数据库

# 设置时间戳
currentTime=$(date "+%Y%m%d%H%M%S")

# PostgreSQL 数据库备份
docker exec docker-postgreSQL pg_dump -h localhost -U postgres postgres > /opt/ldif/pg_${currentTime}.sql

# 备份完成日志记录
echo "$(date '+%Y-%m-%d %H:%M:%S') - 备份完成: pg_${currentTime}.sql" >> /opt/ldif/backup.log
```

#### 设置脚本权限
```bash
chmod +x /opt/ldif/cronbackup.sh
```

### 3.3. 脚本验证测试

#### 手动执行测试
```bash
cd /opt/ldif
./cronbackup.sh
```

#### 验证备份文件
```bash
# 检查是否生成备份文件
ls -la /opt/ldif/*.sql

# 检查备份日志
tail -f /opt/ldif/backup.log
```

**预期结果**：应生成类似 `pg_20231201230501.sql` 的备份文件。

### 3.4. 配置定时任务

#### 编辑定时任务
```bash
crontab -e
```

#### 添加备份任务（每日 23:05 执行）
```bash
# CloudQuery 数据库备份任务（每天 23:05 执行）
05 23 * * * /bin/bash /opt/ldif/cronbackup.sh
```

#### 添加清理任务（每日 23:35 执行）
```bash
# 清理 30 天前的备份文件（每天 23:35 执行）
35 23 * * * find /opt/ldif \( -name "*.ldif" -o -name "*.sql" \) -mtime +30 -exec rm -rf {} \;
```



## 4. 故障排查

### 4.1. 常见问题检查

#### 检查定时任务状态
```bash
# 查看当前用户的定时任务
crontab -l

# 检查 cron 服务状态
systemctl status crond

# 查看 cron 执行日志
tail -f /var/log/cron
```

#### 检查脚本执行权限
```bash
ls -la /opt/ldif/cronbackup.sh
# 应显示：-rwxr-xr-x
```

#### 检查容器状态
```bash
# 确认 PostgreSQL 容器正常运行
docker ps | grep docker-postgreSQL

# 检查容器日志
docker logs docker-postgreSQL
```

### 4.2. 手动调试步骤

#### 测试数据库连接
```bash
# 进入容器测试数据库连接
docker exec -it docker-postgreSQL psql -h localhost -U postgres -d postgres -c "SELECT version();"
```

#### 测试备份命令
```bash
# 手动执行备份命令测试
docker exec docker-postgreSQL pg_dump -h localhost -U postgres postgres > /tmp/test_backup.sql
```

## 5. 恢复说明

### 5.1. PostgreSQL 数据库恢复
```bash
# 恢复备份文件（谨慎操作）
docker exec -i docker-postgreSQL psql -h localhost -U postgres -d postgres < /opt/ldif/pg_20231201000000.sql
```

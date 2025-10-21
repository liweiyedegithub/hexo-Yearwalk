---
title: DIY-Python容器
date: 2024-07-07 17:26:51
tags: 
  - Docker
  - Python
top: false
categories: 
  - DIY
  - 自定义构建容器
img: /medias/featureimages/12.jpg

---
# Python 自定义环境容器部署指南

## 一、获取镜像文件

### 百度网盘下载
- **下载链接**：https://pan.baidu.com/s/1PW0rcJdnV-tEM_EOK3abOQ  
- **提取码**：cwze
- **文件名**：`python_diy_3.6.8.tar`

## 二、镜像导入与验证

### 1. 导入Docker镜像
```bash
docker load -i python_diy_3.6.8.tar
```

### 2. 验证镜像导入
```bash
docker images | grep python_diy
```
**预期输出示例**：
```
python_diy   3.6.8    a1b2c3d4e5f6   2 weeks ago   1.2GB
```

## 三、容器启动配置

### 启动Python环境容器
```bash
docker run -d -it \
  --name python_diy_368 \
  -v /opt/py_script:/root \
  -v /etc/localtime:/etc/localtime:ro \
  -p 8868:8868 \
  python_diy:3.6.8 \
  /bin/bash
```

### 参数说明
| 参数 | 说明 |
|------|------|
| `-d` | 后台运行容器 |
| `-it` | 交互模式运行 |
| `--name python_diy_368` | 容器名称标识 |
| `-v /opt/py_script:/root` | 主机脚本目录挂载到容器内 |
| `-v /etc/localtime:ro` | 时间同步配置 |
| `-p 8868:8868` | 端口映射（主机:容器） |

### 目录配置建议
- **主机脚本目录**：`/opt/py_script`（建议与CloudQuery安装目录同级）
- **容器内目录**：`/root`（可根据实际情况调整）

## 四、运行状态检查

### 验证容器启动状态
```bash
docker ps | grep python_diy_368
```

**预期输出示例**：
```
CONTAINER ID  IMAGE              STATUS         PORTS                   NAMES
a1b2c3d4e5f6  python_diy:3.6.8  Up 5 minutes   0.0.0.0:8868->8868/tcp  python_diy_368
```

## 五、Python脚本部署与执行

### 1. 上传Python脚本
将需要执行的Python脚本（如`cqSend2other.py`）上传至主机目录：
```bash
# 确保脚本文件存在
ls -l /opt/py_script/cqSend2other.py
```

### 2. 创建执行脚本
创建执行包装脚本`cqSend2other.sh`：
```bash
vi /opt/cqSend2other.sh
```

**脚本内容**：
```bash
#!/bin/bash
docker exec -it python_diy_368 python3 /root/cqSend2other.py
```

### 3. 设置执行权限
```bash
chmod +x /opt/cqSend2other.sh
```

### 4. 测试脚本执行
```bash
cd /opt
./cqSend2other.sh >> cqSend2other.log
```

### 5. 验证执行结果
```bash
# 查看执行日志
tail -f cqSend2other.log

# 检查容器内Python进程
docker exec python_diy_368 ps aux | grep python
```

## 六、定时任务配置

### 配置Crontab定时任务
```bash
# 编辑当前用户的crontab
crontab -e
```

### 添加定时任务
```bash
# 每分钟执行一次Python脚本
* * * * * cd /opt && ./cqSend2other.sh >> cqSend2other.log 2>&1
```

### 其他定时方案示例
```bash
# 每5分钟执行一次
*/5 * * * * cd /opt && ./cqSend2other.sh >> cqSend2other.log 2>&1

# 每天凌晨2点执行
0 2 * * * cd /opt && ./cqSend2other.sh >> cqSend2other.log 2>&1

# 每小时执行一次
0 * * * * cd /opt && ./cqSend2other.sh >> cqSend2other.log 2>&1
```

### 定时任务管理命令
```bash
# 查看现有定时任务
crontab -l

# 重启cron服务（某些系统需要）
systemctl restart crond
```

## 七、故障排查

### 1. 容器状态检查
```bash
# 查看容器详细状态
docker inspect python_diy_368

# 查看容器日志
docker logs python_diy_368
```

### 2. 脚本执行调试
```bash
# 手动执行测试
docker exec -it python_diy_368 python3 /root/cqSend2other.py

# 进入容器内部调试
docker exec -it python_diy_368 /bin/bash
```

### 3. 定时任务调试
```bash
# 查看cron执行日志
tail -f /var/log/cron

# 检查脚本权限
ls -l /opt/cqSend2other.sh
```

## 八、注意事项

1. **目录权限**：确保`/opt/py_script`目录对容器可读可写
2. **资源监控**：定期检查容器资源使用情况
3. **日志轮转**：配置日志文件轮转，避免磁盘空间占满
4. **网络连通**：确保容器内Python脚本所需的网络访问权限
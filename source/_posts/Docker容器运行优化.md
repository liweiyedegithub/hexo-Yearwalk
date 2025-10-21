---
title: Docker容器运行优化
date: 2024-08-03 11:22:42
tags: 
  - Docker
top: false
categories: 
  - 性能优化
img: /medias/featureimages/13.jpg

---
# 服务器与容器优化配置指南

## 服务器基础优化配置

### 1. 关闭Swap分区（提升Docker容器IO性能）

```bash
# 临时关闭Swap
swapoff -a

# 永久禁用Swap（注释/etc/fstab中的Swap行）
sed -i '/swap/s/^/#/' /etc/fstab
```

**效果**：
- 提高Docker容器I/O速率
- 避免内存交换导致的性能下降

### 2. 防火墙配置优化

```bash
# 停止firewalld服务（临时）
systemctl stop firewalld

# 永久禁用firewalld
systemctl disable firewalld
```

**适用场景**：
- 内网环境或已有其他安全防护措施
- 需要提高网络吞吐量的场景

### 3. 磁盘格式化（需客户授权）

```bash
# 将磁盘格式化为XFS文件系统（高性能）
mkfs.xfs /dev/sdX
```

**优势**：
- XFS在大文件和高并发场景下性能优异
- 特别适合数据库和容器存储

### 4. 内核参数优化

编辑`/etc/sysctl.conf`文件：

```conf
# 增加文件描述符限制
fs.file-max = 1000000

# 提高脏页刷新阈值（写密集型应用）
vm.dirty_ratio = 20
vm.dirty_background_ratio = 10

# 减少交换倾向
vm.swappiness = 10
```

应用配置：
```bash
sysctl -p
```

## 容器性能优化配置

### 1. Java应用内存配置

```yaml
environment:
  - _JAVA_OPTIONS=-Xmx4g -Xms4g
```

**参数说明**：
- `-Xmx4g`：最大堆内存4GB
- `-Xms4g`：初始堆内存4GB
- 建议值根据实际物理内存调整

### 2. DNS解析优化

```bash
docker run --dns=8.8.8.8 --dns-search=. nginx
```

**优势**：
- 指定可靠DNS服务器
- 减少DNS解析延迟
- 与`--network=host`配合使用时效果更佳

### 3. 全局资源限制配置

编辑`/etc/docker/daemon.json`：

```json
{
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 262144,
      "Soft": 262144
    }
  },
  "oom-score-adjust": -500
}
```

**参数说明**：
- `nofile`：单个容器最大文件描述符数
- `oom-score-adjust`：降低容器被OOM Killer终止的概率

### 4. I/O性能优化

```yaml
environment:
  - innodb_flush_method=O_DIRECT
```

**适用场景**：
- 数据库容器（MySQL等）
- 高性能存储应用
- 需要绕过系统缓存直接读写磁盘的场景

## 最佳实践建议

1. **内存配置**：
   - 物理内存的70-80%分配给容器
   - 保留足够内存给宿主机系统进程

2. **监控调整**：
   ```bash
   # 监控容器资源使用
   docker stats
   
   # 监控系统资源
   top
   htop
   ```

3. **分层优化**：
   - 网络密集型应用：优化网络配置
   - 计算密集型应用：优化CPU分配
   - I/O密集型应用：优化存储配置

4. **安全注意事项**：
   - 禁用Swap后需确保物理内存充足
   - 禁用防火墙需配合其他安全措施
   - 生产环境变更前应在测试环境验证
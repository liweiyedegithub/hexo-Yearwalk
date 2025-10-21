---
title: nginx强制使用https访问
date: 2025-07-19 19:10:11
tags: 
  - Linux
  - Nginx
top: false
categories: 
  - https
img: /medias/featureimages/15.jpg
---
# Nginx 强制 HTTPS 访问解决方案

## 一、方案概述

本方案实现将所有 HTTP 请求自动重定向到 HTTPS，确保网站始终通过加密连接访问。适用于已有 SSL 证书配置的 Nginx 服务器。

## 二、前提条件

1. 已安装 Nginx
2. 已配置有效的 SSL 证书（包括 `.crt` 和 `.key` 文件）
3. 服务器已开放 443 端口（HTTPS）

## 三、完整实施步骤

### 1. 检查当前 Nginx 配置

```bash
nginx -t
```

### 2. 修改 Nginx 配置文件

编辑网站配置文件（通常位于 `/etc/nginx/conf.d/your_site.conf` 或 `/etc/nginx/sites-available/your_site`）：

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    # HTTP 强制跳转 HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # SSL 证书配置
    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;
    
    # SSL 优化配置
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    
    # HSTS 头（增强安全性）
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # 网站根目录配置
    root /var/www/html;
    index index.html index.htm;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # 其他配置...
}
```

### 3. 验证配置并重启 Nginx

```bash
# 测试配置语法
nginx -t

# 重新加载配置
systemctl reload nginx
# 或
service nginx reload
```

## 四、可选增强配置

### 1. 配置 HTTP/2（性能优化）

在 443 端口的 server 块中添加 `http2`：

```nginx
listen 443 ssl http2;
```

### 2. 启用 OCSP Stapling（提高 SSL 验证速度）

```nginx
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /path/to/ca_bundle.crt;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

### 3. 防止 SSL 剥离攻击

```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
```

## 五、验证 HTTPS 强制跳转

### 1. 使用 curl 测试

```bash
curl -I http://example.com
```

**预期输出**：
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Location: https://example.com/
```

### 2. 浏览器测试

1. 访问 `http://example.com`
2. 地址栏应自动跳转到 `https://example.com`
3. 检查浏览器地址栏是否有锁图标（表示安全连接）

## 六、常见问题解决

### 1. 出现重定向循环

**症状**：浏览器显示 "ERR_TOO_MANY_REDIRECTS"

**解决方案**：
- 检查是否有其他重定向规则冲突
- 确保 HTTPS 配置正确且证书有效
- 清除浏览器缓存后重试

### 2. 证书不受信任警告

**解决方案**：
- 确保证书链完整
- 使用权威 CA 机构颁发的证书
- 检查证书是否过期

### 3. 混合内容警告

**症状**：HTTPS 页面加载 HTTP 资源

**解决方案**：
- 将所有资源链接改为 HTTPS 或相对路径
- 添加 CSP 头：

```nginx
add_header Content-Security-Policy "upgrade-insecure-requests";
```

## 七、高级配置（多域名场景）

### 通配符证书配置

```nginx
server {
    listen 80;
    server_name *.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name *.example.com;
    
    ssl_certificate /path/to/wildcard.crt;
    ssl_certificate_key /path/to/wildcard.key;
    
    # 其他配置...
}
```

## 八、安全加固建议

1. **禁用旧版 TLS**：
   ```nginx
   ssl_protocols TLSv1.2 TLSv1.3;
   ```

2. **启用安全加密套件**：
   ```nginx
   ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
   ```

3. **启用 SSL 会话缓存**：
   ```nginx
   ssl_session_cache shared:SSL:10m;
   ssl_session_timeout 10m;
   ```

4. **定期更新证书**：
   - 设置证书到期提醒
   - 考虑使用 Let's Encrypt 自动续期

## 九、性能优化

1. **启用 SSL 会话票证**：
   ```nginx
   ssl_session_tickets on;
   ```

2. **调整缓冲区大小**：
   ```nginx
   ssl_buffer_size 4k;
   ```

3. **启用 TLS 1.3 0-RTT**（谨慎使用）：
   ```nginx
   ssl_early_data on;
   ```

通过以上配置，您的网站将强制使用 HTTPS，同时获得优化的安全性和性能表现。
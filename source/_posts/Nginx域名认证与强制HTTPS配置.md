---
title: Nginx 域名认证与强制 HTTPS 配置
date: 2025-06-11 09:54:22
tags: 
  - Linux
  - Nginx
top: false
categories: 
  - https
  - 域名认证
img: /medias/featureimages/16.jpg
---
# Nginx 域名认证与强制 HTTPS 配置指南

本文介绍如何在 Nginx 中配置域名绑定与强制 HTTP 跳转到 HTTPS。

---

## ✅ 一、完整示例配置

假设你的域名是 `example.com`：

```nginx
# --------------------------
# HTTP 部分（用于重定向到 HTTPS）
# --------------------------
server {
    listen 80;
    server_name example.com www.example.com;   # 绑定域名

    # 将所有 http 请求重定向到 https
    return 301 https://$host$request_uri;
}

# --------------------------
# HTTPS 部分（真正提供服务）
# --------------------------
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL 证书配置
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 网站根目录（改成你自己网站的路径）
    root /var/www/html;
    index index.html index.htm;

    # 如果是 Hexo 或静态博客
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 可选：为静态文件加缓存头
    location ~* \\.(jpg|jpeg|png|gif|ico|css|js|svg|woff2?)$ {
        expires 30d;
        access_log off;
    }

    # 可选：安全头部增强
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
}
```

---

## 🧭 二、配置说明

| 配置项                                     | 作用                                 |
| ------------------------------------------ | ------------------------------------ |
| `server_name example.com www.example.com;` | 指定要服务的域名                     |
| `listen 80;` + `return 301 https://...`    | 所有 HTTP 请求都跳转到 HTTPS         |
| `listen 443 ssl http2;`                    | 开启 HTTPS（使用 HTTP/2）            |
| `ssl_certificate` / `ssl_certificate_key`  | SSL 证书路径                         |
| `try_files $uri $uri/ /index.html;`        | 适用于 Hexo、React、Vue 这种静态站点 |
| `add_header ...`                           | 增强安全性，可选                     |

---

## 🚀 三、应用配置步骤

```bash
# 1️⃣ 检查 Nginx 配置是否正确
sudo nginx -t

# 2️⃣ 重新加载配置
sudo systemctl reload nginx
# 或
sudo nginx -s reload
```

---

## 🔒 四、验证是否生效

1. 在浏览器中访问 `http://example.com`
   → 会自动跳转到 `https://example.com`

2. 微信、手机浏览器访问
   → 地址栏显示小锁 🔒，说明 HTTPS 启用了。

---

## 💡 五、可选增强：HSTS

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

> 让浏览器在未来一年内自动把 HTTP 请求升级为 HTTPS。
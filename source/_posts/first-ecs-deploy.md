---
title: 第一次在阿里云 ECS 上部署 Web 应用
date: 2026-05-16 21:00:00
tags:
  - 运维
  - ECS
  - Linux
categories: 学习笔记
---

今天完成了人生第一次服务器部署，记录一下整个过程。

## 服务器信息

- 系统：Alibaba Cloud Linux 3
- CPU：2核
- 内存：1.8G
- 公网 IP：47.98.103.83

## 部署架构

```
用户浏览器
    ↓
阿里云安全组（放行 80 端口）
    ↓
Nginx（监听 80，反向代理）
    ↓
Node.js 应用（监听 3000）
    ↓
PM2（进程守护，崩溃自动重启）
```

## 关键步骤

### 1. SSH 登录服务器

```bash
ssh root@47.98.103.83
```

### 2. 安装 Node.js

```bash
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
dnf install nodejs -y
```

### 3. 用 PM2 守护进程

PM2 的作用：应用崩溃自动重启，服务器重启后自动拉起。

```bash
npm install -g pm2
pm2 start app.js --name myapp
pm2 save
pm2 startup systemd
```

### 4. Nginx 反向代理配置

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```

### 5. 安全组放行端口

在阿里云控制台 → 安全组 → 入方向 → 添加 HTTP/80 规则。

## 踩的坑

- GitHub 在国内访问不稳定，clone 项目要用镜像源
- 密码登录默认关闭，需要先在控制台重置密码
- 防火墙（firewalld）未运行，端口控制完全靠阿里云安全组

## 收获

第一次真正理解了"反向代理"的意思：用户访问 80 端口，Nginx 转发给内部 3000 端口，对外只暴露 80，更安全。

---
title: 🤖 CentOS 安装 OpenClaw 并打通飞书
published: 2026-03-11
updated: 2026-03-11
description: 'CentOS/Anolis OS 安装 OpenClaw 并集成飞书机器人的完整部署流程'
image: '/images/openclaw_feishu_setup.png'
tags: [openclaw, 飞书, AI]
category: '开发'
draft: false
---

## 系统要求

- CentOS 7+ / Anolis OS 8+
- 2核4G内存以上
- 公网IP或SSH隧道访问能力

## 系统准备

### 1. 安装Git

```bash
sudo yum install git -y
```

### 2. 安装Development Tools

```bash
sudo yum groupinstall "Development Tools" -y
```

### 3. 安装CMake

```bash
sudo yum install cmake -y
```

## Node.js环境

### 4. 安装NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
```

### 5. 安装Node.js

```bash
nvm install 20
nvm use 20
```

### 6. 验证Node.js

```bash
node -v
npm -v
```

### 7. 安装OpenClaw

```bash
npm install -g openclaw
```

## 飞书开放平台配置

### 8. 创建飞书应用

访问 [飞书开放平台](https://open.feishu.cn/app) 创建企业自建应用

### 9. 配置权限

在"权限管理"中添加以下权限:

```json
{
  "im:message": ["readonly", "readwrite"],
  "im:message.group_at_msg": ["readonly"],
  "im:message.p2p_msg": ["readonly"],
  "im:message.p2p_msg:send_as_bot": ["readwrite"],
  "im:chat": ["readonly", "readwrite"],
  "contact:user.base": ["readonly"],
  "contact:user.email": ["readonly"],
  "bitable:app": ["readonly", "readwrite"]
}
```

### 10. 获取凭证

记录以下信息:
- App ID
- App Secret

### 11. 配置事件订阅

在"事件订阅"中添加:
- 接收消息 v2.0
- 消息已读 v2.0

### 12. 配置机器人

在"机器人"中启用机器人功能

### 13. 发布版本

创建应用版本并发布到企业

### 14. 安装应用

在企业管理后台安装应用

## OpenClaw初始化

### 15. 运行onboard

```bash
openclaw onboard
```

### 16. 配置向导

按提示输入:
- 选择平台: Feishu
- App ID
- App Secret
- Webhook URL (暂时留空)

### 17. 测试连接

```bash
openclaw test
```

## 配置优化

### 18. 编辑配置文件

```bash
vi ~/.openclaw/config.json
```

### 19. 修改配置

```json
{
  "platform": "feishu",
  "appId": "your_app_id",
  "appSecret": "your_app_secret",
  "webhookUrl": "http://your_server_ip:3000/webhook",
  "port": 3000
}
```

### 20. 创建飞书群组

创建一个测试群组并将机器人添加进群

### 21. 配置事件订阅URL

在飞书开放平台"事件订阅"中配置:
```
http://your_server_ip:3000/webhook
```

### 22. 启动OpenClaw

```bash
openclaw start
```

### 23. 验证事件订阅

在飞书开放平台点击"请求配置URL"验证

### 24. 测试机器人

在群组中@机器人发送消息测试

## Web控制台访问

### 方案一: SSH隧道(本地开发)

```bash
ssh -L 3000:localhost:3000 user@your_server_ip
```

访问 `http://localhost:3000`

### 方案二: 公网访问(生产环境)

#### 配置防火墙

```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

#### 使用Nginx反向代理(推荐)

```bash
sudo yum install nginx -y
```

配置Nginx:

```nginx
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

启动Nginx:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 常见问题

### 权限不足

确保已添加所有必需的API权限并重新发布应用版本

### 事件订阅失败

检查服务器防火墙和安全组配置,确保3000端口可访问

### 机器人无响应

查看OpenClaw日志:

```bash
openclaw logs
```

## 参考资源

- [OpenClaw官方文档](https://github.com/openclaw/openclaw)
- [飞书开放平台](https://open.feishu.cn/)

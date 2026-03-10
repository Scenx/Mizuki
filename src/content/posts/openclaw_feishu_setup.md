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

本文档记录 CentOS/Anolis OS 8.10 安装 OpenClaw 并集成飞书机器人的完整流程。

> 系统要求: 8GB 内存,确保机器同时能在全球含中国大陆畅通

## 系统准备

### 1. 安装 Git

```bash
dnf install -y git
```

### 2. 安装开发工具

```bash
dnf groupinstall "Development Tools" -y
```

### 3. 安装 CMake

```bash
dnf install -y cmake
```

> Ubuntu 用户可跳过步骤 1-3,根据需要自行安装对应工具

## Node.js 环境

### 4. 安装 NVM

访问 https://www.nvmnode.com/guide/installation.html#nvm-install-for-linux-macos 拷贝安装命令

### 5. 刷新环境变量

关闭 SSH 会话重新打开,或执行:

```bash
source ~/.bashrc
```

### 6. 安装 Node.js

访问 https://nodejs.org/zh-cn/download 查看最新 LTS 版本(如 24.14.0)

```bash
nvm install 24.14.0
```

### 7. 安装 OpenClaw

参考 https://github.com/openclaw/openclaw

```bash
npm install -g openclaw@latest
```

## 飞书开放平台配置

### 8. 创建企业自建应用

访问 https://open.feishu.cn/app

### 9. 获取凭证

点击左侧"凭证与基础信息",记录 App ID 和 App Secret

### 10. 添加机器人能力

点击左侧"添加应用能力",添加机器人能力

### 11. 导入权限配置

点击左侧"权限管理" → "批量导入权限",导入以下 JSON:

```json
{
  "scopes": {
    "tenant": [
      "base:app:copy", "base:app:create", "base:app:read", "base:app:update",
      "base:collaborator:create", "base:collaborator:delete", "base:collaborator:read",
      "base:dashboard:copy", "base:dashboard:read",
      "base:field:create", "base:field:delete", "base:field:read", "base:field:update",
      "base:form:read", "base:form:update",
      "base:record:create", "base:record:delete", "base:record:read", "base:record:retrieve", "base:record:update",
      "base:role:create", "base:role:delete", "base:role:read", "base:role:update",
      "base:table:create", "base:table:delete", "base:table:read", "base:table:update",
      "base:view:read", "base:view:write_only",
      "base:workflow:read", "base:workflow:write",
      "bitable:app", "bitable:app:readonly",
      "board:whiteboard:node:create", "board:whiteboard:node:delete", "board:whiteboard:node:read", "board:whiteboard:node:update",
      "contact:contact", "contact:contact.base:readonly", "contact:contact:update_department_id", "contact:contact:update_user_id",
      "contact:department.base:readonly", "contact:department.hrbp:readonly", "contact:department.organize:readonly",
      "contact:functional_role", "contact:functional_role:readonly",
      "contact:group", "contact:group:readonly",
      "contact:job_family", "contact:job_family:readonly",
      "contact:job_level", "contact:job_level:readonly",
      "contact:job_title:readonly", "contact:role:readonly",
      "contact:unit", "contact:unit:readonly",
      "contact:user.assign_info:read", "contact:user.base:readonly", "contact:user.department:readonly",
      "contact:user.dotted_line_leader_info.read", "contact:user.email:readonly", "contact:user.employee:readonly",
      "contact:user.employee_id:readonly", "contact:user.employee_number:read", "contact:user.gender:readonly",
      "contact:user.id:readonly", "contact:user.job_family:readonly", "contact:user.job_level:readonly",
      "contact:user.phone:readonly", "contact:user.subscription_ids:write", "contact:user.user_geo",
      "contact:work_city:readonly",
      "docs:doc", "docs:doc:readonly",
      "docs:document.comment:create", "docs:document.comment:read", "docs:document.comment:update", "docs:document.comment:write_only",
      "docs:document.content:read", "docs:document.media:download", "docs:document.media:upload",
      "docs:document.subscription", "docs:document.subscription:read",
      "docs:document:copy", "docs:document:export", "docs:document:import",
      "docs:event.document_deleted:read", "docs:event.document_edited:read", "docs:event.document_opened:read",
      "docs:event:subscribe",
      "docs:permission.member", "docs:permission.member:auth", "docs:permission.member:create",
      "docs:permission.member:delete", "docs:permission.member:readonly", "docs:permission.member:retrieve",
      "docs:permission.member:transfer", "docs:permission.member:update",
      "docs:permission.setting", "docs:permission.setting:read", "docs:permission.setting:readonly", "docs:permission.setting:write_only",
      "docx:document", "docx:document.block:convert", "docx:document:create", "docx:document:readonly", "docx:document:write_only",
      "drive:drive", "drive:drive.metadata:readonly", "drive:drive.search:readonly", "drive:drive:readonly",
      "drive:drive:version", "drive:drive:version:readonly", "drive:export:readonly",
      "drive:file", "drive:file.like:readonly", "drive:file.meta.sec_label.read_only",
      "drive:file:download", "drive:file:readonly", "drive:file:upload", "drive:file:view_record:readonly",
      "im:app_feed_card:write", "im:biz_entity_tag_relation:read", "im:biz_entity_tag_relation:write",
      "im:chat", "im:chat.access_event.bot_p2p_chat:read", "im:chat.announcement:read", "im:chat.announcement:write_only",
      "im:chat.chat_pins:read", "im:chat.chat_pins:write_only", "im:chat.collab_plugins:read", "im:chat.collab_plugins:write_only",
      "im:chat.managers:write_only", "im:chat.members:bot_access", "im:chat.members:read", "im:chat.members:write_only",
      "im:chat.menu_tree:read", "im:chat.menu_tree:write_only", "im:chat.moderation:read",
      "im:chat.tabs:read", "im:chat.tabs:write_only", "im:chat.top_notice:write_only",
      "im:chat.widgets:read", "im:chat.widgets:write_only",
      "im:chat:create", "im:chat:delete", "im:chat:moderation:write_only", "im:chat:operate_as_owner",
      "im:chat:read", "im:chat:readonly", "im:chat:update",
      "im:datasync.feed_card.time_sensitive:write",
      "im:message", "im:message.group_at_msg:readonly", "im:message.group_msg", "im:message.p2p_msg:readonly",
      "im:message.pins:read", "im:message.pins:write_only", "im:message.reactions:read", "im:message.reactions:write_only",
      "im:message.urgent", "im:message.urgent.status:write", "im:message.urgent:phone", "im:message.urgent:sms",
      "im:message:readonly", "im:message:recall", "im:message:send_as_bot", "im:message:send_multi_depts",
      "im:message:send_multi_users", "im:message:send_sys_msg", "im:message:update",
      "im:resource", "im:tag:read", "im:tag:write", "im:url_preview.update", "im:user_agent:read",
      "sheets:spreadsheet", "sheets:spreadsheet.meta:read", "sheets:spreadsheet.meta:write_only",
      "sheets:spreadsheet:create", "sheets:spreadsheet:read", "sheets:spreadsheet:readonly", "sheets:spreadsheet:write_only",
      "slides:presentation:create", "slides:presentation:read", "slides:presentation:update", "slides:presentation:write_only",
      "space:document.event:read", "space:document:delete", "space:document:move", "space:document:retrieve", "space:document:shortcut",
      "space:folder:create",
      "tenant:tenant.product_assign_info:read",
      "wiki:member:create", "wiki:member:retrieve", "wiki:member:update",
      "wiki:node:copy", "wiki:node:create", "wiki:node:move", "wiki:node:read", "wiki:node:retrieve", "wiki:node:update",
      "wiki:setting:read", "wiki:setting:write_only",
      "wiki:space:read", "wiki:space:retrieve", "wiki:space:write_only",
      "wiki:wiki", "wiki:wiki:readonly"
    ],
    "user": []
  }
}
```

### 12. 申请权限开通

点击"申请开通" → "确认"

### 13. 创建应用版本

点击上方提示"创建版本":
- 应用版本号: 1.0.0
- 更新说明: 随意填写
- 可用范围: 编辑改为"全部成员"
- 点击"保存"

### 14. 发布应用

确认提交发布申请 → 点击"确认发布"(免审自动通过)

## OpenClaw 初始化

### 15. 启动 Onboarding

```bash
openclaw onboard --install-daemon
```

### 16. 配置向导

按照提示依次选择/输入:

```
Continue? → yes
Onboarding mode → QuickStart
Model/auth provider → Custom Provider
API Base URL → 填写你的 API 地址
How to provide API key? → Paste API key now
API Key → 填写你的 API Key
Endpoint compatibility → OpenAI-compatible
Model ID → 填写模型 ID (如 claude-opus-4-6)
Endpoint ID → 直接回车
Model alias → 直接回车
Select channel → Feishu/Lark (飞书)
Install Feishu plugin? → Use local plugin path
App Secret → 填写飞书 App Secret
Feishu App ID → 填写飞书 App ID
Connection mode → WebSocket (default)
Feishu domain → Feishu (feishu.cn) - China
Group chat policy → Open - respond in all groups (requires mention)
Search provider → Skip for now
Configure skills now? → yes
Install missing dependencies → Skip for now
Set GOOGLE_PLACES_API_KEY → No
Set GEMINI_API_KEY → No
Set NOTION_API_KEY → No
Set OPENAI_API_KEY (image-gen) → No
Set OPENAI_API_KEY (whisper) → No
Set ELEVENLABS_API_KEY → No
Enable hooks? → Skip for now
How to hatch? → Hatch in TUI (recommended)
```

### 17. 测试连接

在对话框输入"说中文 你好",验证机器人响应

## 配置优化

### 18. 编辑配置文件

```bash
vi /root/.openclaw/openclaw.json
```

修改以下配置:
- contextWindow: 900000
- maxTokens: 900000
- 在 channels.feishu 中添加: "requireMention": false

示例:
```json
{
  "channels": {
    "feishu": {
    "enabled": true,
    "appId": "xxx",
    "appSecret": "xxx",
    "connectionMode": "websocket",
    "domain": "feishu",
    "requireMention": false,
    "groupPolicy": "open"
  }
  }
```

### 19. 创建飞书群组

飞书建群 → 添加群机器人 → 选择你创建的企业自建应用

### 20. 配置事件订阅

回到飞书开放平台:
- 点击"事件与回调" → "事件配置"
- 订阅方式: "使用长连接接收事件" → 保存

### 21. 添加事件

点击"添加事件":
- 选择"应用身份订阅"
- 展开"消息与群组"
- 全选所有事件 → 添加

### 22. 发布新版本

创建版本:
- 应用版本号: 1.0.1
- 更新说明: 随意填写
- 保存 → 确认发布(免审)

### 23. 重启服务

```bash
# 关闭 SSH 会话,重新打开
openclaw gateway restart
```

### 24. 验证功能

在群里直接发送消息(无需 @),机器人回复即成功

## Web 控制台访问

### 方案一: SSH 隧道

本地浏览器通过 SSH 隧道访问

### 方案二: 公网访问

Nginx 反向代理 + HTTPS + OpenClaw 安全策略配置 allow origin




---
title: 🔊 幽冥传奇客户端没有声音传奇客户端没声音传奇没声音的解决方法
published: 2025-12-11
description: '传奇客户端没声音？反编译后检查 apktool.yml 配置即可解决'
image: '/images/legend_client_no_sound.png'
tags: [传奇, 逆向, Android]
category: '逆向'
draft: false
---

> 传奇客户端回编译后没有声音？问题出在 apktool.yml 配置文件。

## 解决方法

### 步骤一：反编译客户端

使用反编译工具反编译客户端，比如 ApkToolAid。

### 步骤二：检查 apktool.yml 配置

查看反编译后目录下的 `apktool.yml` 配置文件，检查 `doNotCompress` 中是否有 `mp3` 这一项：

![apktool.yml 配置](/images/legend_client_no_sound_apktool.png)

如果没有，手动添加：

```yaml
doNotCompress:
  - arsc
  - png
  - mp3
```

### 步骤三：回编译

直接回编译，出来的包就有声音了。
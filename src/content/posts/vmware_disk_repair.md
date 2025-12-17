---
title: 🔧 vmware虚拟机意外断电导致磁盘损坏centos无法开机解决方案
published: 2025-12-17
description: 'VMware 虚拟机断电后 CentOS 无法启动，使用 xfs_repair 修复磁盘'
image: '/images/vmware_disk_repair.png'
tags: [vmware]
category: '运维'
draft: false
---

> VMware 虚拟机意外断电后 CentOS 无法开机？XFS 文件系统损坏导致，修复即可。

## 解决方法

```bash
xfs_repair -L /dev/mapper/ao-root
```

如果提示设备忙，先卸载：

```bash
umount /dev/mapper/ao-root
xfs_repair -L /dev/mapper/ao-root
```
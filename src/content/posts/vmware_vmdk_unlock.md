---
title: 🔧 VMware 断电后打不开磁盘 vmdk 或其所依赖的快照磁盘的解决方案
published: 2026-02-20
updated: 2026-02-20
description: 'VMware 因断电等意外情况导致无法打开 vmdk 磁盘文件，通过删除 .lck 锁文件或 vmware-vdiskmanager 修复'
image: '/images/vmware_vmdk_unlock.png'
tags: [vmware]
category: '运维'
draft: false
---

> 断电或意外关机后，VMware 启动虚拟机报错：打不开磁盘 xxx.vmdk 或它所依赖的某个快照磁盘。

![报错截图](/images/vmware_vmdk_unlock_error.webp)

## 方法一：删除 .lck 锁文件

关闭 VMware，进入虚拟机所在目录，删除所有 `.lck` 文件夹，然后重新打开 VMware 启动虚拟机。

## 方法二：使用 vmware-vdiskmanager 修复

如果方法一无效，关闭 VMware，打开命令提示符，对虚拟机目录下的每个 vmdk 文件都执行一次修复：

```bash
cd /d "C:\Program Files (x86)\VMware\VMware Workstation"
vmware-vdiskmanager.exe -R "D:\你的虚拟机路径\xxx.vmdk"
vmware-vdiskmanager.exe -R "D:\你的虚拟机路径\xxx-000001.vmdk"
vmware-vdiskmanager.exe -R "D:\你的虚拟机路径\xxx-000002.vmdk"
```

没问题的文件执行后不会有输出，修复成功的会提示 `The virtual disk was repaired successfully`。全部执行完后重新启动虚拟机即可。

---
title: 🖥️ VMware 虚拟机死机无法关机的快速解决办法
published: 2026-01-17
description: 'VMware 虚拟机卡死后无法正常关机时，通过查找进程 PID 强制结束的解决方案'
image: '/images/vmware_vm_freeze.png'
tags: [vmware]
category: '运维'
draft: false
---

> 当 VMware 中的虚拟机死机，且无法通过正常方式关闭时，可以通过查找虚拟机进程的 PID 来强制结束。

## Step 1：打开虚拟机目录

在 VMware 中右键单击此台虚拟机，并选择 **"打开虚拟机目录"**。

![打开虚拟机目录](/images/vmware_vm_freeze_step1.png)

## Step 2：找到 vmware.log 文件

在打开的目录中，找到 `vmware.log` 文件。

![找到 vmware.log](/images/vmware_vm_freeze_step2.png)

## Step 3：查找 PID

双击打开 `vmware.log` 文件，在第一行找到 `pid` 值。

![查找 PID](/images/vmware_vm_freeze_step3.png)

## Step 4：结束进程

打开物理机的 **"任务管理器"**，点击 **"详细信息"** 选项卡，找到 PID 为上一步中获取的进程号，右键单击选择 **"结束任务"** 即可。

![结束进程](/images/vmware_vm_freeze_step4.png)

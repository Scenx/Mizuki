---
title: 🔧 VMware 虚拟机磁盘被撑大、删文件后主机空间不释放的压缩方法
published: 2026-06-23
updated: 2026-06-23
description: 'VMware 精简磁盘只增不减，虚拟机里删了文件主机硬盘照样被占满，界面"压缩磁盘"也没用，正确做法是 vmware-toolbox-cmd disk shrink。'
image: '/images/vmware_disk_shrink.png'
tags: [vmware, 运维]
category: '运维'
draft: false
---

> VMware 的精简（thin）磁盘**只增不减**：虚拟机里写过的数据把 vmdk 撑大后，哪怕你在操作系统里把文件删干净，主机硬盘上的 vmdk 文件**还是那么大**，空间一点不还。这篇记录正确的压缩姿势，以及一个必踩的坑。

## 现象

- 虚拟机磁盘曾经被写满（日志、临时文件、数据库等）；
- 进虚拟机操作系统里 `rm` 删掉一大堆文件、`df` 显示空闲一大片；
- 回到主机一看，那个 `.vmdk` 文件**纹丝不动**，主机硬盘照样被占着。

## 坑：界面"压缩磁盘"没用，必须命令行 shrink

很多人第一反应是用 VMware 界面的 **"压缩磁盘 / Compact"**——对运行中的 Linux 虚拟机，这个**基本没用**。

根因是两层：

1. **精简磁盘只增不减**：vmdk 随写入增长，但删除文件不会自动收缩。
2. **Linux 删文件不清零数据块**：删除只是把块标记为"可覆盖"，块里的旧数据还在。从主机/ESXi 的视角看，那些块**依旧有数据**，所以无从回收。

也就是说，**得先把空闲块清零、再压缩**，主机才知道哪些块是真的空。VMware Tools 的 `disk shrink` 正是把这两步一起做了，而界面那个"压缩"在这种场景下使不上劲。

## 正确做法：vmware-toolbox-cmd disk shrink

### 1. 先装 VMware Tools / open-vm-tools

`vmware-toolbox-cmd` 来自 VMware Tools，**没装就没有这个命令**：

```bash
# Debian/Ubuntu
sudo apt install open-vm-tools

# CentOS/RHEL/Anolis
sudo yum install open-vm-tools
```

### 2. 关掉磁盘读写频繁的进程

shrink 过程会**锁定虚拟机内存**（quiesce），且会把空闲空间全部清零、磁盘 IO 极重。**务必先停掉 MySQL、Redis 这类高频读写的进程**，否则数据可能不一致、过程也会被反复打断：

```bash
sudo systemctl stop mysqld   # 按实际情况停掉数据库等高 IO 服务
```

### 3. 执行压缩

```bash
# 看哪些挂载点可压缩
sudo vmware-toolbox-cmd disk list

# 压缩根分区（按需换成你的挂载点）
sudo vmware-toolbox-cmd disk shrink /
```

![空闲块先被清零、再整体压缩](/images/vmware_disk_shrink_process.png)

## 关键的坑：过程中磁盘"看起来爆满"，别慌

执行 shrink 时，你会看到**虚拟机操作系统里的磁盘逐步被写满、直到 100%**——这是正常的，它在往空闲块里写零。

**这个"爆满"的过程并不会占用主机的磁盘空间**（写的是会被回收的零块），所以：

- **不要中途打断**、不要被吓到去删东西；
- 安心等它跑完。结束后 vmdk 会真正缩小，主机硬盘空间随之释放。

## 结论

VMware 精简磁盘只增不减，虚拟机里删文件主机空间不还，**界面"压缩磁盘"对运行中的 Linux 没用**。正确做法是装好 VMware Tools，停掉数据库等高 IO 进程，跑 `vmware-toolbox-cmd disk shrink /`；过程中磁盘看似爆满是清零阶段、不占主机空间，等它跑完即可回收。

---

**参考链接：**
- [Defragmenting and shrinking VMware virtual machine disks](https://knowledge.broadcom.com/external/article/315631/defragmenting-and-shrinking-vmware-works.html)
- [Growing, thinning, and shrinking virtual disks in ESXi](https://knowledge.broadcom.com/external/article/341657/growing-thinning-and-shrinking-virtual-d.html)
- [Howto Shrink a Thin Provisioned Virtual Disk (VMDK)](https://www.virten.net/2014/11/howto-shrink-a-thin-provisioned-virtual-disk-vmdk/)

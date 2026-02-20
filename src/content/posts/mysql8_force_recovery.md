---
title: 🔧 MySQL 8.0+ 断电导致无法启动的数据恢复方案
published: 2026-02-20
updated: 2026-02-20
description: 'MySQL 8.0+ 断电后无法启动，通过 innodb_force_recovery 强制启动并备份数据，重装后导入恢复'
image: '/images/mysql8_force_recovery.png'
tags: [mysql, 数据库]
category: '数据库'
draft: false
---

> MySQL 8.0+ 断电后无法启动？通过 `innodb_force_recovery` 强制启动，备份数据后重装恢复。

## Step 1：修改配置强制启动

编辑 MySQL 配置文件，在 `[mysqld]` 下添加：

```ini
[mysqld]
innodb_force_recovery = 4
```

然后尝试启动 MySQL：

```bash
sudo systemctl start mysql
```

如果启动失败，依次尝试将值改为 `5`、`6`，直到能启动为止。数值越大跳过的检查越多，优先用最小的值。

## Step 2：备份业务数据库

启动成功后马上用 Navicat、mysqldump 等工具把自己的业务库逐个导出为 `.sql` 文件，确认文件大小正常、内容完整。

## Step 3：卸载干净并重装 MySQL

备份确认无误后，彻底卸载 MySQL 并清理数据目录，重新安装。

## Step 4：导入数据

将之前备份的 `.sql` 文件逐个导入新的 MySQL，验证数据正常即可。

> ⚠️ **注意：** `innodb_force_recovery` 模式下 MySQL 是只读的，不要尝试写入操作。备份完成后记得把配置文件中的 `innodb_force_recovery` 删掉，否则重装后也会进入恢复模式。

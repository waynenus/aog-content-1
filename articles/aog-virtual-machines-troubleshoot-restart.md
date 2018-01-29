---
title: '文件系统损坏导致虚拟机无法正常启动的问题及解决方法'
description: '文件系统损坏导致虚拟机无法正常启动的问题及解决方法'
author: chpaadmin
resourceTags: 'Virtual Machines Linux'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: chpa
ms.date: 01/29/2018
wacn.date: 10/11/2016
---

# 文件系统损坏导致虚拟机无法正常启动的问题及解决方法

## 简介

计算机的[文件系统]((https://zh.wikipedia.org/zh-cn/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F))是一种存储和组织计算机数据的方法，它使得对其访问和查找变得容易，文件系统使用文件和树形目录的抽象逻辑概念代替了硬盘和光盘等物理设备使用数据块的概念，用户使用文件系统来保存数据不必关心数据实际保存在硬盘（或者光盘）的地址为多少的数据块上，只需要记住这个文件的所属目录和文件名。

在使用中, 会遇到文件系统损坏的故障, 直接导致Azure平台的虚拟机无法正常启动和访问, 以下是关于此类问题的描述及解决方法.

> [!NOTE]
> 本文档讨论的文件系统以 CentOS 作为示例, 其他版本的 Linux 略有不同, 请注意差别。

## 文件系统损坏常见问题

### 问题一：root 文件系统损坏

```
Checking all file systems.
[/sbin/fsck.ext4 (1) -- /] fsck.ext4 -a /dev/sda1
/dev/sda1 contains a file system with errors, check forced .
/dev/sda1: Inodes that were part of a corrupted orphan linked list found.
/dev/sda1: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY
```

### 问题二：常规文件系统损坏

```
EXT4-fs (sda1): INFO: recovery required on readonly filesystem
EXT4-fs (sda1): write access will be enabled during recovery
EXT4-fs warning (device sda1): ext4_clear_journal_err:4531: Filesystem error recorded from previous mount: IO failure
EXT4-fs warning (device sda1): ext4_clear_journal_err:4532: Making fs in need of filesystem check .
```

## 解决方案

设定以下场景：
- **虚拟机 A** ：文件系统故障所在的虚拟机。
- **虚拟机 B** ：临时创建的虚拟机。

### 问题一：root 文件系统损坏

1. 在 Azure 门户上停止运行虚拟机 A。
2. 创建一台临时Linux虚拟机B。
3. 删除虚拟机 A, 但是选择保留磁盘。
4. 选择虚拟机B->附加磁盘->选择虚拟机A的系统磁盘。
5. 以管理员身份登陆虚拟机 B。
6. 执行: `# fdisk -l`。
7. 确认虚拟机A的系统磁盘作为新的磁盘设备附加在虚拟机B上.假定虚拟机A的系统磁盘为/dev/sdc, root文件系统为/dev/sdc1。
8. 执行以下步骤, 进行备份文件系统信息:

    ```
    # fdisk -l /dev/sdc > /var/tmp/fdisk_before.log
    # dumpe2fs /dev/sdc1 > /var/tmp/dumpe2fs_before.log
    # tune2fs -l /dev/sdc1 > /var/tmp/tune2fs_before.log
    # e2fsck -n /dev/sdc1 > /var/tmp/e2fsck._beforelog
    ```

9. 执行以下命令, 进行文件系统修复:

    ```
    # fsck -yM /dev/sdc1
    ```

### 问题二：常规文件系统损坏

1. 在 Azure 门户上停止运行虚拟机 A。
2. 创建一台临时Linux虚拟机B。
3. 删除虚拟机 A, 但是选择保留磁盘。
4. 选择虚拟机B->附加磁盘->选择虚拟机A的系统磁盘。
5. 以管理员身份登陆虚拟机 B。
6. 执行: `# fdisk -l`。
7. 确认虚拟机 A 的系统磁盘作为新的磁盘设备附加在虚拟机 B 上.假定虚拟机 A 的系统磁盘为 /dev/sdc, root 文件系统为 /dev/sdc1。
8. 执行如下命令,将虚拟机A的系统磁盘挂载到临时虚拟机上:

    ```
    # mkdir /mnt/temp_fs
    # mount /dev/sdc1 /mnt/temp_fs
    # cp /mnt/temp_fs/etc/fstab /mnt/temp_fs/etc/fstab.org
    # vi /mnt/temp_fs/etc/fstab
    // 将文件系统损坏的条目注释掉,保存修改, 退出vi.
    # umount /dev/sdc1
    ```

9. 分离虚拟机 A 的系统磁盘。
10. 基于虚拟机 A 的系统磁盘, 重建虚拟机 A。
11. 以管理员身份登录虚拟机 A。
12. 执行以下命令, 进行文件系统修复:

    ```
    # fsck -yM <file system>;
    ```

13. 文件系统修复完毕以后, 恢复 /etc/fstab 被注释的对应条目, 重启虚拟机。
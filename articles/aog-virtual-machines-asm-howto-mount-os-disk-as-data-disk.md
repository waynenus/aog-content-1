---
title: '挂载 ASM 虚机 OS 盘为数据磁盘'
description: '挂载 ASM 虚机 OS 盘为数据磁盘'
author: StellaZhou64 
resourceTags: 'Virtual Machines, ASM, OS Disk, Data Disk'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: jieying.zhou
ms.date: 08/28/2018
wacn.date: 08/28/2018
---

# 挂载 ASM 虚机 OS 盘为数据磁盘

本文以 CentOS 为例，说明如何挂载 ASM 虚机 OS 盘为其他虚机的数据磁盘。

1. 删除 VM，保留磁盘。

    ![01](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/01.png)

2. 新建一台同类型 VM（新），将原 VM 的 OS 盘作为数据盘挂载。

    **VM（新）** -> **磁盘** -> **附加现有磁盘** -> **位置** -> **选择存储账户** -> **vhds** -> **原 VM OS 磁盘**

    > [!NOTE]
    > 挂载的 OS 盘必须以 .vhd 结尾，若不是 .vhd 结尾可以在 Storage Explorer 中重命名。

    挂载完成后效果如下，数据磁盘处显示原 VM OS 盘：

    ![02](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/02.png)

3. Putty 登陆 VM（新）。首先使用 `dmesg` 来查找磁盘（用于发现新磁盘的方法可能各不相同）。以下示例使用 `dmesg` 来筛选 SCSI 磁盘：

    ![03](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/03.png)

    此处，`sdc` 是我们需要的磁盘。

4. 新建文件夹用于挂载磁盘。此处新建文件夹 `/datadrive`。使用以下命令挂载：

    ![04](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/04.png)

5. 进入 `/datadrive` 查看原虚机 OS 盘内容。

    ![05](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/05.png)

6. 若要重新使用 OS 盘创建虚机，首先 `umount disk`。

    ![06](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/06.png)

7. 将原 VM OS 磁盘分离。

    ![07](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/07.png)

8. 在磁盘（经典）中找到原 VM OS 磁盘。点击上方 “**创建 VM**”。

    ![08](media/aog-virtual-machines-asm-howto-mount-os-disk-as-data-disk/08.png)

9. 按照提示创建 VM。
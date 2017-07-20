---
title: 如何解决 Linux 虚拟机磁盘设备名不一致的问题
description: 如何解决 Linux 虚拟机磁盘设备名不一致的问题
service: ''
resource: Virtual Machines Linux
author: chpaadmin
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'Virtual Machines, Linux, Disk'

ms.service: virtual-machines-linux
wacn.topic: aog
ms.topic: article
ms.author: chpa
ms.date: 07/20/2017
wacn.date: 07/20/2017
---

# 如何解决 Linux 虚拟机磁盘设备名不一致的问题

## 问题描述

在 Linux 虚拟机内，将附加的多块数据磁盘以设备名（/dev/sdxx）的方式创建文件系统，并将之写入 /etc/fstab 文件中实现启动自动挂载功能。但是在虚拟机重启之后，会随机出现设备名和实际的文件系统不一致的问题。

## 问题分析

由于 Azure 虚拟机在启动过程中，随机分配数据磁盘的 SCSI 地址，导致了数据磁盘在附加后，获取的 SCSI 地址会随机变化。比如原来的数据磁盘 A，初始的设备名为 /dev/sdc ，在重启之后，会随机的被分配为 /dev/sdd ，原来的数据磁盘 B，初始的设备名为 /dev/sdd，在重启之后，被分配为 /dev/sdc。这样的话，如果客户通过设备进行自动挂载的话，那么会看到挂载点下的实际数据是不一致的。

## 解决方案

为了避免上述的设计导致自动挂载时出现故障，建议使用 UUID 来代替设备名。不同文件系统的 UUID 是不会随着重启而改变的。这样，就可以确保每次自动挂载都能将正确的文件系统挂载到正确的挂载点。

1. 按照以下命令获取文件系统的 UUID :

```
# blkid
/dev/sdb1: UUID="f788cc09-fad5-4df9-9360-ffe39d82****" TYPE="ext4"
/dev/sda1: UUID="9bb6e11f-4697-476a-9e71-0ebfff61****" TYPE="xfs"
/dev/sda2: UUID="445d96a9-aeb1-4623-a2db-be133bdf****" TYPE="xfs"
```

2. 按照以下格式写入 /etc/fstab 文件 :

```
# cat /etc/fstab
…
…
UUID=445d96a9-aeb1-4623-a2db-be133bdf**** /                       xfs     defaults        0 0
UUID=9bb6e11f-4697-476a-9e71-0ebfff61**** /boot                   xfs     defaults        0 0
```
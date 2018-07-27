---
title: '如何解决 VMware 克隆虚拟机在 Azure Site Recovery 中的复制状态异常问题'
description: '如何解决 VMware 克隆虚拟机在 Azure Site Recovery 中的复制状态异常问题'
author: lickkylee
resourceTags: 'Site Recovery, VMware'
ms.service: site-recovery
wacn.topic: aog
ms.topic: article
ms.author: lqi
ms.date: 07/18/2018
wacn.date: 07/18/2018
---

# 如何解决 VMware 克隆虚拟机在 Azure Site Recovery 中的复制状态异常问题

## 问题描述

在 VMware 环境中的虚拟机 A 已经被 Azure Site Recovery(以下简称：ASR) 进行了容灾保护，虚拟机 A 在 ASR 中的状态正常。现将其克隆后新建了另一台虚拟机 B。在对虚拟机 B 启用复制后，可能会遇到虚拟机 A、B 的复制状态出现异常，如复制进度卡住等现象。

## 问题分析

VMware 环境中对虚拟机进行复制时，会在虚拟机中安装 Mobility Agent。该 Agent 在安装过程中会生成一个 ID，称为 `Host ID` 或 `AgentGeneratedId`，在复制过程中，该 ID 会唯一标识一台虚拟机，被记录在配置服务器的数据库中。同时，虚拟机的复制数据也会写入进程服务器的以该 ID 命名的文件夹下面。

虚拟机被克隆并启用复制后，该 ID 不会改变，会重新写入数据库，并复用原有进程服务器中的文件夹，因此会造成配置服务器中虚拟机记录的不正确，进程服务器中的复制数据处理异常，导致整个复制状态异常。

## 解决方法

基于这种情况，建议用户清理这两台虚拟机的复制环境，重新进行复制。具体过程如下：

1. 清理这两台虚拟机现有的信息和环境

    1. 在 Azure ASR 中，禁用两台虚拟机的复制。

        登录 [Azure 门户](https://portal.azure.cn)，选中您的 Recovery Services 保管库，在左侧导航栏中，点击 “**复制的项**”，在右侧标签页下，右键单击计算机，再单击 “**禁用复制**”。在 “**禁用复制**” 页中，选择 **禁用复制并删除(推荐)**。

    2. 在配置服务器上，检查两台虚拟机信息是否正确：

        以管理员身份打开 mysql 的命令行，执行命令查看所有注册的虚拟机的信息：

        ```sql
        mysql> use svsdb1;
        mysql> select id as hostid, name, ipaddress, ostype as operatingsystem, from_unixtime(lasthostupdatetime) as heartbeat from hosts where name!='InMageProfiler'\G;
        ```

        注意查看虚拟机的信息是否正确。若不正确，请按下面步骤进行清理:

        在命令行中切换到 `C:\home\svsystems\bin`（根据安装情况，盘符可能不同）。<br>
        运行以下命令注销：<br>
        语法：`Unregister-ASRComponent.pl -IPAddress <IP_ADDRESS_OF_MACHINE_TO_UNREGISTER> -Component <Source/ PS / MT>`<br>
        例如，若需要取消虚拟机 "OnPrem-VM01"（ 10.0.0.4）的注册，则运行:<br>
        `Unregister-ASRComponent.pl -IPAddress 10.0.0.4 -Component Source`

        参考文档：[如何清理重复项](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx)。

    3. 删除两台虚拟机上的 Mobility Agent。

        以 root 用户身份在 Linux 服务器上登录。<br>
        在终端中转到 `/user/local/ASR`，请运行以下命令卸载移动服务：<br>
        `uninstall.sh -Y`

        参考文档：[安装移动服务](https://docs.azure.cn/zh-cn/site-recovery/vmware-azure-install-mobility-service#uninstall-mobility-service-on-a-linux-computer)。

2. 重新为两台虚拟机启用复制。

> [!NOTE]
> 如果客户由克隆虚拟机并使用 ASR 复制的需求，建议在克隆之前先停止 ASR 复制，删除 Mobility Agent，在克隆后再启用复制。这样来避免其他潜在的由克隆引起的问题。

---
title: "跨资源组移动 MySQL 实例的替代方案"
description: "跨资源组移动 MySQL 实例的替代方案"
author: kustbilla
resourceTags: 'MySQL Database On Azure, Migration, PowerShell'
ms.service: mysql
wacn.topic: aog
ms.topic: article
ms.author: weihuan
ms.date: 5/30/2018
wacn.date: 5/30/2018
---

# 跨资源组移动 MySQL 实例的替代方案

## 问题描述

客户在 Azure 门户上移动 MySQL 数据库到其他资源组时出错。报错信息如下：

```
无法移动资源“/subscriptions/<sub_id>/resourceGroups/<rg_name>/providers/Microsoft.MySql/servers/<server_name>”。跟踪 ID 是“eba0cd27-5920-473f-a050-8ee9a05fa510” (代码: ResourceMoveFailed) The move resources request is not supported. (代码: MoveResourcesNotSupported)
```

## 问题分析

目前 Azure 尚不支持对 MySQL 实例的跨资源组移动，因此客户在尝试移动资源时会出现这个报错。
参考官方文档：[无法移动的服务](https://docs.azure.cn/zh-cn/azure-resource-manager/resource-group-move-resources#services-that-cannot-be-moved)。

## 替代方案

此需求可以用 PowerShell 的替代方案来解决，方法如下：

1. 假设现 MySQL 实例 A 位于资源组 1 下，现在需要将实例 A 移动到资源组 2 下；
2. 用 PowerShell 在资源组 2 下创建实例 A 的从属实例 B(该操作只能用 PowerShell 完成，因为只有在 PowerShell 环境中才能选择从属实例所属的资源组) ；
3. 当从属实例创建完毕后，等待两个实例间的数据同步至最小；
4. 停止复制关系使得实例 B 获得读写权限；
5. 删除资源组 1 下的实例 A；
6. 在资源组 2 下为实例 B 创建名为 A 的从属实例；
7. 待数据同步完成停止复制关系，这样便可获得在资源组 2 下的有读写权限的实例 A。

参考 PowerShell 命令如下所示：

```powershell
New-AzureRmResource -ResourceType "Microsoft.MySql/servers" -ResourceName <slave MySQL service name>  -ApiVersion 2015-09-01 -ResourceGroupName <slave resource group> -Location <Location of slave> -SkuObject @{name='MS3'} -Properties @{replicationMode='AzureSlave'; creationSource=@{server='<master MySQL service name>';region='<location of master>'}; version = '<MySQL Version>';}
```

> [!NOTE]
> 以上步骤仅为实现跨资源组移动 MySQL 实例的替代方案，在资源组 1 下删除实例 A 和资源组 2 上创建实例 A 期间，系统无法访问该实例名。
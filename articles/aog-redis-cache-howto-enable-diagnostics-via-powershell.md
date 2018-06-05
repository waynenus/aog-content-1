---
title: "如何使用 PowerShell 开启 Redis 诊断功能"
description: "如何使用 PowerShell 开启 Redis 诊断功能"
author: chenrui1988
resourceTags: 'Redis Cache, Diagnostics, PowerShell'
ms.service: redis-cache
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 5/16/2018
wacn.date: 5/16/2018
---

# 如何使用 PowerShell 开启 Redis 诊断功能

Azure Redis 提供了很多对 Redis 监视功能，并能够将监视数据存储于存储账号，Azure Redis 默认是开启监视功能，如果在使用过程中发现，没有开启该功能，请尝试使用 PowerShell 或者直接在管理界面来操作开启。在生产环境下，我们也建议开启监视功能。

本主要介绍如何使用 PowerShell 开启 Redis 的诊断功能，开启诊断使用 `Set-AzureRmRedisCacheDiagnostics` 命令，使用方式如下：

```powershell
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
Set-AzureRmRedisCacheDiagnostics -ResourceGroupName geogroup -Name georedis -StorageAccountId "/subscriptions/xxx/resourceGroups/poddev/providers/Microsoft.Storage/storageAccounts/storageName"
```

> [!NOTE]
> 参数说明：
> `StorageAccountId` 需要指向诊断数据的存储账号资源 ID，我们可以在存储账号的属性中看到此值，该存储账号必须与 Redis 在同一个区域。
> ![01](media/aog-redis-cache-howto-enable-diagnostics-via-powershell/01.png)
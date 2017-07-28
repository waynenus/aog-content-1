---
title: 解决由于 Key Vault 未启用导致 VM 无法启动的问题
description: 解决由于 Key Vault 未启用导致 VM 无法启动的问题
services: Key Vault
documentationCenter: ''
author: garfield0
manager: ''
editor: ''
tags: ''

ms.service: key-vault
wacn.topic: aog
ms.topic: article
ms.author: tonglei.wang
ms.date: 06/28/2017
wacn.date: 06/28/2017
---

# 解决由于 Key Vault 未启用导致 VM 无法启动的问题

## 现象描述

用户将虚拟机从 ASM 模式迁移到 ARM 模式后，有时会出现虚拟机启动时报错，提示信息如下：

```
Provisioning failed. Key Vault https://XXXX has not been enabled for deployment or the vault id provided. /subscriptions/XXXX/resourceGroups/demo-Migrated/providers/Microsoft.KeyVault/vaults/demo, does not match the Key Vault's true resource id. KeyVaultAccessForbidden
```

## 解决方案

1. 首先，我们需要使用下面的命令来获取受影响的虚拟机所在订阅下的 Key Vault 信息：

    ```PowerShell
    #登陆到受影响虚拟机所在的订阅下
    PS C:\windows\system32> Login-AzureRmAccount –Environment AzureChinaCloud

    Environment           : AzureChinaCloud
    Account               : XXX@mcpod.partner.onmschina.cn
    TenantId              : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    SubscriptionId        : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    SubscriptionName      : <订阅名称>
    CurrentStorageAccount :

    #查询并找到虚拟机所在资源组对应的 Key Vault 信息
    PS C:\windows\system32> Get-AzureRmKeyVault


    Vault Name          : <Key Vault 名称>
    Resource Group Name : <资源组名称>
    Location            : chinaeast
    Resource ID         : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /resourceGroups/<资源组名称>/providers/Microsoft.KeyVault/vaults/<Key Vault 名称>
    Tags                :
    ```

    > [!NOTE]
    > 如果您在结果中无法找到虚拟机所在的资源组对应的 Key Vault 信息，则无法使用该解决方案，请联系技术支持寻求帮助。

2. 使用下述命令来启用虚拟机对应资源组的 Key Vault：

    ```PowerShell
    #启用 Key Vault
    Set-AzureRmKeyVaultAccessPolicy -VaultName <Key Vault 名称> -ResourceGroupName <资源组名称> -EnabledForDeployment
    ```

3. 重新启动受影响的虚拟机，虚拟机可以正常启动。
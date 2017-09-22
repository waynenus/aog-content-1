---
title: 关于托管磁盘的复制
description: 关于托管磁盘的复制
service: ''
resource: Virtual Machines
author: garfield0
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'Virtual Machines, Managed Disks, Replication'
cloudEnvironments: MoonCake

ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: tonglei.wang
ms.date: 09/20/2017
wacn.date: 09/20/2017
---
# 关于托管磁盘的复制

## 适用场景

想把托盘磁盘在当前资源组里复制一份？想把托盘磁盘复制到其他资源组里？想把托盘磁盘复制到其他订阅里？在 Azure 门户上发现没有这些功能？你可以在这篇文章里可以找到答案。

> [!NOTE]
> 本文仅适用于使用托管磁盘的虚拟机。

## 解决方案

1. 将托盘磁盘在当前资源组里复制一份。

    ```
    #读取托盘磁盘信息
    PS C:\windows\system32> $managedDisk= Get-AzureRMDisk -ResourceGroupName "<资源组名称>" -DiskName "<托管磁盘名称>"
    #准备托盘磁盘配置信息
    PS C:\windows\system32> $diskConfig = New-AzureRmDiskConfig -SourceResourceId $managedDisk.Id -Location $managedDisk.Location -CreateOption Copy
    #创建托盘磁盘，注意磁盘名称不要重复
    PS C:\windows\system32> New-AzureRmDisk -Disk $diskConfig -DiskName "<托管磁盘名称>" -ResourceGroupName "<资源组名称>"
    AccountType        : StandardLRS
    TimeCreated        : 8/29/2017 1:36:52 PM
    OsType             : Linux
    CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
    DiskSizeGB         : 128
    EncryptionSettings :
    OwnerId            :
    ProvisioningState  : Succeeded
    Id                 : /subscriptions/订阅 ID/resourceGroups/资源组名称/providers/Microsoft.Compute/disks/托盘磁盘名称
    Name               : 托盘磁盘名称
    Type               : Microsoft.Compute/disks
    Location           : chinaeast
    Tags               :
    ```

2. 将托管磁盘复制到其他资源组里。

    ```
    #读取托盘磁盘信息
    PS C:\windows\system32> $managedDisk= Get-AzureRMDisk -ResourceGroupName "<资源组名称>" -DiskName "<托管磁盘名称>"
    #准备托盘磁盘配置信息
    PS C:\windows\system32> $diskConfig = New-AzureRmDiskConfig -SourceResourceId $managedDisk.Id -Location $managedDisk.Location -CreateOption Copy
    #创建托盘磁盘，注意磁盘名称不要重复
    PS C:\windows\system32> New-AzureRmDisk -Disk $diskConfig -DiskName "<托管磁盘名称>" -ResourceGroupName "<新资源组名称>"
    AccountType        : StandardLRS
    TimeCreated        : 8/29/2017 1:36:52 PM
    OsType             : Linux
    CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
    DiskSizeGB         : 128
    EncryptionSettings :
    OwnerId            :
    ProvisioningState  : Succeeded
    Id                 : /subscriptions/订阅 ID/resourceGroups/新资源组名称/providers/Microsoft.Compute/disks/托盘磁盘名称
    Name               : 托盘磁盘名称
    Type               : Microsoft.Compute/disks
    Location           : chinaeast
    Tags               :
    ```

3. 将创建好的托管磁盘挂载到虚拟机上。请注意托管磁盘只能挂载到使用托管磁盘的虚拟机上。

    ```
    #在当前订阅下读取托盘磁盘信息
    PS C:\windows\system32> $managedDisk= Get-AzureRMDisk -ResourceGroupName "<资源组名称>" -DiskName "<托管磁盘名称>"

    #登陆需要复制到的订阅
    PS C:\windows\system32> Login-AzureRmAccount -Environment AzureChinaCloud

    Environment           : AzureChinaCloud
    Account               : tonwan@mcpod.partner.onmschina.cn
    TenantId              : 954ddad8-66d7-47a8-8f9f-1316152d9587
    SubscriptionId        : d1fc1ce1-589b-47af-b2a8-0a5e860f0560
    SubscriptionName      : Tonglei.Wang
    CurrentStorageAccount :

    #选择需要复制到的订阅
    PS C:\windows\system32> Select-AzureRmSubscription -SubscriptionId d1fc1ce1-589b-47af-b2a8-0a5e860f0560


    Environment           : AzureChinaCloud
    Account               : tonwan@mcpod.partner.onmschina.cn
    TenantId              : 954ddad8-66d7-47a8-8f9f-1316152d9587
    SubscriptionId        : d1fc1ce1-589b-47af-b2a8-0a5e860f0560
    SubscriptionName      : Tonglei.Wang
    CurrentStorageAccount :

    #准备托盘磁盘配置信息
    PS C:\windows\system32> $diskConfig = New-AzureRmDiskConfig -SourceResourceId $managedDisk.Id -Location $managedDisk.Location -CreateOption Copy

    #创建托盘磁盘，注意磁盘名称不要重复
    PS C:\windows\system32> New-AzureRmDisk -Disk $diskConfig -DiskName "<托管磁盘名称>" -ResourceGroupName "<资源组名称>"
    AccountType        : StandardLRS
    TimeCreated        : 8/29/2017 1:36:52 PM
    OsType             : Linux
    CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
    DiskSizeGB         : 128
    EncryptionSettings :
    OwnerId            :
    ProvisioningState  : Succeeded
    Id                 : /subscriptions/ 需要复制到的订阅 ID/resourceGroups/ 资源组名称 /providers/Microsoft.Compute/disks/ 托盘磁盘名称
    Name               : 托盘磁盘名称
    Type               : Microsoft.Compute/disks
    Location           : chinaeast
    Tags               :
    ```
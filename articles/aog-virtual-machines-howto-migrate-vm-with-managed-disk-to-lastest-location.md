---
title: '如何将托管磁盘虚拟机迁移至新区(中国东部 2 和中国北部 2)'
description: '如何将托管磁盘虚拟机迁移至新区(中国东部 2 和中国北部 2)'
author: EmiliaHuang
resourceTags: 'Virtual Machines, Migration, PowerShell'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: Xuzhou.Huang
ms.date: 10/15/2018
wacn.date: 10/15/2018
---

# 如何将托管磁盘虚拟机迁移至新区(中国东部 2 和中国北部 2)

## 问题描述

随着中国东部 2 和中国北部 2 数据中心的正式商用，不少用户计划将中国东部或中国北部的虚拟机迁移至新区以获取更多资源。除了可以使用 ASR（Azure Site Recovery）进行资源迁移，用户也可以通过以下方式对托管磁盘虚拟机进行迁移。

## 解决方法

考虑到 127GiB 的非托管磁盘直接转换成托管磁盘时会增加 1GiB 的容量，托管磁盘定价层会因此由 S10 升级为 S15。为此我们将快照作为转换介质，规避磁盘容量上的变化。

如下为迁移一台 OS disk 大小为 127GiB 的托管虚拟机的基本操作步骤：

1. 为原虚机托管磁盘 A 创建快照 B；
2. 将快照 B 拷贝为新数据中心中的 VHD C；
3. 为 VHD C 创建新区域中的快照 D;
4. 通过快照 D 创建新区域中的托管磁盘 E。

```powershell
# To login to Azure Resource Manager
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
$SubscriptionID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Select-AzureRmSubscription -SubscriptionID $SubscriptionID

$ResourceGroupName = 'myResourceGroup'
$vmName = 'sourceVM'
$oldlocation = 'old region'
$newlocation = "new region"

# Get the VM object #
$sourcevm = Get-AzureRmVM -Name $vmName `
   -ResourceGroupName $resourceGroupName

# Get the OS disk name #
$disk = Get-AzureRmDisk -ResourceGroupName $resourceGroupName `
  -DiskName $sourcevm.StorageProfile.OsDisk.Name

### a) Create a snapshot B of the original disk A in the old region.
# Create the snapshot configuration #
# Please choose Windows or Linux according to your VM #
$SnapshotConfig =  New-AzureRmSnapshotConfig `
  -SourceUri $disk.Id `
  -OsType Windows `
  -CreateOption Copy `
  -Location $oldlocation

# Take the snapshot #
$SnapshotName = 'mySnapshot'
$Snapshot = New-AzureRmSnapshot `
   -Snapshot $SnapshotConfig `
   -SnapshotName $SnapshotName `
   -ResourceGroupName $ResourceGroupName

### b) Copy managed snapshots as VHD C to a storage account in the new region.
#Provide Shared Access Signature (SAS) expiry duration in seconds e.g. 3600.
#Know more about SAS here: https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-shared-access-signature-part-1
$sasExpiryDuration = "3600"

#Generate the SAS for the snapshot
$sas = Grant-AzureRmSnapshotAccess -ResourceGroupName $ResourceGroupName -SnapshotName $SnapshotName  -DurationInSecond $sasExpiryDuration -Access Read

# Create storage account in new region #
$NewResourceGroupName = "NewResourceGroupName"
New-AzureRmResourceGroup -Name $NewResourceGroupName -Location $newlocation

$storageAccountName = "yourstorageaccountName"
$storageType = 'Standard_LRS'
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $NewResourceGroupName `
  -Name $storageAccountName `
  -Location $newlocation `
  -SkuName $storageType
$ctx = $storageAccount.Context

#Name of the storage container where the downloaded snapshot will be stored
$storageContainerName = "storagecontainername"
New-AzureStorageContainer -Name $storageContainerName -Context $ctx -Permission blob

#Provide the key of the storage account where you want to copy snapshot.
$storageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $NewResourceGroupName -AccountName $storageAccountName)[0].Value

#Provide the name of the VHD file to which snapshot will be copied.
$destinationVHDFileName = "vhdfilename"

#Create the context for the storage account which will be used to copy snapshot to the storage account
$destinationContext = New-AzureStorageContext –StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

#Copy the snapshot to the storage account
$job = Start-AzureStorageBlobCopy -AbsoluteUri $sas.AccessSAS -DestContainer $storageContainerName -DestContext $destinationContext -DestBlob $destinationVHDFileName
Wait-Job $job

### c) Create another snapshot D from the copied unmanaged disk C.
#Provide the URI of the VHD file (page blob) in a storage account. Please not that this is NOT the SAS URI of the storage container where VHD file is stored.
#e.g. https://contosostorageaccount1.blob.core.windows.net/vhds/contosovhd123.vhd
#Note: VHD file can be deleted as soon as Managed Disk is created.
$sourceVHDURI = (Get-AzureStorageBlob -Container $storageContainerName -Context $ctx -Blob "$destinationVHDFileName").ICloudBlob.Uri.AbsoluteUri

#Provide the resource Id of the storage account where VHD file is stored.
#e.g. /subscriptions/6582b1g7-e212-446b-b509-314e17e1efb0/resourceGroups/MDDemo/providers/Microsoft.Storage/storageAccounts/contosostorageaccount1
#This is an optional parameter if you are creating snapshot in the same subscription
$storageAccountId = (Get-azurermstorageaccount -ResourceGroupName $NewResourceGroupName -Name $storageAccountName).Id

$snapshotConfigNew = New-AzureRmSnapshotConfig -AccountType $storageType -Location $newlocation -CreateOption Import -StorageAccountId $storageAccountId -SourceUri $sourceVHDURI

#Provide the name of the snapshot
$snapshotNameNew = 'mysnapshotNew'
$SnapshotNew = New-AzureRmSnapshot -Snapshot $snapshotConfigNew -ResourceGroupName $NewResourceGroupName -SnapshotName $snapshotNameNew

### d) Create the target managed disk E from snapshot D.
# Create managed disk in the new region #
$newDisk = New-AzureRmDisk -DiskName ($sourcevm.StorageProfile.OsDisk.Name) -Disk `
    (New-AzureRmDiskConfig  -Location $newlocation -CreateOption Copy `
    -SourceResourceId $SnapshotNew.Id) `
    -ResourceGroupName $NewResourceGroupName

# Create the new VM#
# Create the subNet and vNet #
$subnetName = 'mySubNet'
$singleSubnet = New-AzureRmVirtualNetworkSubnetConfig `
   -Name $subnetName `
   -AddressPrefix 10.0.0.0/24

$vnetName = "myVnetName"
$vnet = New-AzureRmVirtualNetwork `
   -Name $vnetName -ResourceGroupName $NewResourceGroupName `
   -Location $newlocation `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $singleSubnet

# Create the network security group and an RDP rule #
$nsgName = "myNsg"

$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzureRmNetworkSecurityGroup `
   -ResourceGroupName $NewResourceGroupName `
   -Location $newlocation `
   -Name $nsgName -SecurityRules $rdpRule

# Create a public IP address and NIC #
$ipName = "myIP"
$pip = New-AzureRmPublicIpAddress `
   -Name $ipName -ResourceGroupName $NewResourceGroupName `
   -Location $newlocation `
   -AllocationMethod Dynamic

$nicName = "myNicName"
$nic = New-AzureRmNetworkInterface -Name $nicName `
   -ResourceGroupName $NewResourceGroupName `
   -Location $newlocation -SubnetId $vnet.Subnets[0].Id `
   -PublicIpAddressId $pip.Id `
   -NetworkSecurityGroupId $nsg.Id

# Set the VM name and size #
$vmName = "myVM"
$vmSize = "standard_A1_v2"
$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize

# Add the NIC #
$vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Add the OS disk #
# Please choose Windows or Linux according to your VM #
$vm = Set-AzureRmVMOSDisk -VM $vm -ManagedDiskId $newDisk.Id -StorageAccountType Standard_LRS `
    -DiskSizeInGB 127 -CreateOption Attach -Windows

# Complete the VM #
New-AzureRmVM -ResourceGroupName $NewResourceGroupName -Location $newlocation -VM $vm
```

> [!NOTE]
> 1. 请在运行以上命令前将Powershell AzureRM模块更新至最新版（https://docs.microsoft.com/zh-cn/powershell/azure/install-azurerm-ps?view=azurermps-6.12.0#update-the-azure-powershell-module）
> 2. 虚机迁移后 IP 地址会发生变更。
> 3. 本脚本通过创建快照的方式为托管磁盘拷贝成非托管磁盘，是为了在原虚机不停机的条件下进行迁移工作。为了更好的保证数据一致性和完整性，我们仍旧建议用户可以先停机再进行拷贝。

---
title: "解决由 PowerShell 复制正在运行的虚拟机导致的 HostName 异常问题"
description: "本文用于解决由 PowerShell 复制正在运行的虚拟机导致的 HostName 异常问题"
author: LiheZhang
resourceTags: 'Virtual Machines, HostName, PowerShell'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 02/05/2018
wacn.date: 01/31/2018
---

# 解决由 PowerShell 复制正在运行的虚拟机导致的 HostName 异常问题

## 问题描述

用户通过复制正在运行的虚拟机的 VHD 磁盘创建新的虚拟机后，出现网络通信异常的现象，应用无法通过 HostName 正常解析到对应的虚拟机设备，从而导致众多应用无法正常安装。

## 问题分析

在 Azure 平台，我们并不推荐在不做一般化操作的前提下，通过直接复制正在运行的虚拟机的 VHD 磁盘的方式创建虚拟机。Azure 平台官方推荐对虚拟机执行一般化操作后，再通过[捕获镜像的方式创建新的虚拟机](/virtual-machines/linux/tutorial-custom-images)。

因为通过复制正在运行的虚拟机的 VHD 磁盘创建出新的虚拟机，和原来正在运行的虚拟机所有的配置（包含 HostName）都是一致的。这将导致相同 HostName 的不同虚拟机拥有不同的 IP 地址，Azure 平台 DNS 服务器会将 HostName 对应的 IP 地址更新为最晚重启的虚拟机的内网IP。当用户的应用配置为使用 HostName 的方式建立网络通信时，一旦有虚拟机重启、或继续用这个 VHD 创建新的虚拟机时，Azure 平台可能会根据域名解析到不正确的目标虚拟机，导致业务的网络通信混乱或异常。

新创建的虚拟机中的网卡配置文件同源虚拟机的网卡配置文件是完全一致的，所以新虚拟机网卡配置中的 `DHCP_HOSTNAME` 与源虚拟机也一致，示例如下：源虚拟机 **CraneCenos** 和新建的虚拟机 **CranevmfromVHD** 的网卡配置文件均相同：
```
[crane@CraneCentos ~]$ cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=dhcp
TYPE=Ethernet
USERCTL=no
PEERDNS=yes
IPV6INIT=no
DHCP_HOSTNAME=CraneCentos
```
此时在两台虚拟机（源虚拟机和新建虚拟机）内部运行 `host <HostName>` 命令结果是相同的，HostName 的结果均为新创建虚拟机（**cranevmfromVHD**）的内网 IP，而非源虚拟机的内网 IP。这是因为新虚拟机创建后，Azure 平台的 DNS 会将自己 DNS 中 HostName 记录更新为新创建虚拟机的 IP 地址 **172.10.1.20** ，来替代源虚拟机的内网 IP 地址 **172.10.1.4**:
```
[crane@CraneCentos ~]$ host CraneCentos
CraneCentos.0w2jigzwqggenpfyzenup22n0b.ax.internal.chinacloudapp.cn has address 172.10.1.20
```
用 Azure 门户上显示的新建虚拟机的 HostName 则会提示无法找到（新虚拟机实际 HostName 为 CraneCentos）：
```
[crane@CraneCentos ~]$ host cranevmfromVHD
Host cranevmfromVHD not found: 3(NXDOMAIN)
```
在实际应用中，如果同一个 VNET 下想通过 HostName 方式访问其它虚拟机，那么在此种情况下就会出现：通过 HostName 的方式原本访问到的应该是 **172.10.1.4**，但现在通过 HostName 的方式访问到的则为另外一台 IP 地址为 **172.10.1.20** 的虚拟机，此种情况则是导致该 HostName 解析或 HostName 通信异常的原因。

## 解决方法

针对如上问题，参考解决方案如下：

> [!NOTE]
> 以下步骤中请使用您的实际项目的 HostName 进行替换。

- 以 Linux CentOS 操作系统为例：

    1. 确保 /etc/waagent.conf 文件中的如下选项为”y”：`Provisioning.MoninorHostName=y`
    2. 通过命令 `vi /etc/sysconfig/network-scripts/ifcfg-eth0` 修改新创建虚拟机的 HostName。
    3. 重启 Waagent 服务：`service waagent restart`
    4. 修改虚拟机内部的显示名称：`sysctl kernel.hostname=<HOSTNAME>`
    5. 重启虚拟机。

- 以 Linux Ubuntu 操作系统为例:

    1. 确保 /etc/waagent.conf 文件中的如下选项为”y”：`Provisioning.MoninorHostName=y`
    2. 通过命令 `sudo /bin/hostname <HOSTNAME>` 修改新创建虚拟机的 HostName。
    3. 重启 Waagent 服务：`service WaLinuxAgent restart`
    4. 修改虚拟机内部的显示名称：`sysctl kernel.hostname=<HOSTNAME>`
    5. 重启虚拟机。
    
通过以上步骤，在虚拟机重启后，我们可以看到虚拟机的 HostName 以及虚拟机内部显示的文件名均已与虚拟机的名称一致了。

## 更多信息

使用 Azure PowerShell 复制正在运行的虚拟机 "**CraneCentos**" 的 VHD 并新建虚拟机 "**cranevmfromVHD**" 的详细操作步骤如下：

```powershell
Login-AzureRmAccount -EnvironmentName AzureChinaCloud   #登录订阅
Get-AzureRmSubscription  #获取订阅
Select-AzureSubscription -SubscriptionId xxx  #选择当前订阅
#复制正在运行的虚拟机的 VHD，请提前将待复制的 VHD 所在的存储容器改为允许公网访问的“容器”选项
Get-AzureRmStorageAccount -ResourceGroupName cranetest -Name cranetestdiag549
$storageAccountName = "cranenorthtest"
$storageContainerName = "vhds"
$storageAccountKey =“xxx”'
$destinationVHDFileName = "Testcranecentos69.vhd"
$destinationContext = New-AzureStorageContext –StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
start-AzureStorageBlobCopy -AbsoluteUri "https://xxx.blob.core.chinacloudapi.cn/vhds/CraneCentos20170724112854.vhd" -DestContainer $storageContainerName -DestContext $destinationContext -DestBlob $destinationVHDFileName
#获取 VNET & Subnet（本例中使用已经存在的 VNET）
$vnet=Get-AzureRmVirtualNetwork -Name cranevnetnorth -ResourceGroupName crane
$Subnet1=Get-AzureRmVirtualNetworkSubnetConfig -Name subnet1 -VirtualNetwork $vnet

#创建一个公网 IP 地址：
$publicIP=New-AzureRmPublicIpAddress -Name cranetestfromVHD -ResourceGroupName crane -Location "China North" -AllocationMethod Dynamic -IpAddressVersion IPv4 -Force  #新建
Get-AzureRmPublicIpAddress -Name cranetestfromVHD -ResourceGroupName crane #查看刚刚创建的公网 IP 信息
#创建可用性集：
$Availabilityset = New-AzureRmAvailabilitySet -Name cranetestAV -ResourceGroupName crane -Location "China North"
#创建网卡：
$NIC= New-AzureRmNetworkInterface -Name cranetestvmfromVHDNIC -ResourceGroupName crane -Location "China North" -SubnetId $Subnet1.Id -PublicIpAddressId $publicIP.Id -PrivateIpAddress 172.10.1.20
$NIC= Get-AzureRmNetworkInterface -Name cranetestvmfromVHDNIC -ResourceGroupName crane
#配置要使用的存储账号以及系统盘名称：
$storage = Get-AzureRmStorageAccount -Name cranenorthtest -ResourceGroupName crane
$vmname = "cranevmfromVHD"
$osdiskname = $vmname + "_OSDisk"
$osdiskurl = ”https://cranenorthtest.blob.core.chinacloudapi.cn/vhds/Testcranecentos69.vhd“
#生成虚拟机的配置，将新建的虚拟机放在虚拟网络及可用性集中：
$vmconfig = New-AzureRmVMConfig -VMName $vmname -VMSize Standard_A1 -AvailabilitySetId $Availabilityset.Id | Set-AzureRmVMOSDisk -Name $osdiskname -VhdUri $osdiskurl  -CreateOption Attach -Linux | Add-AzureRmVMNetworkInterface -Id $NIC.Id -Primary
#创建虚拟机：
New-AzureRmVM -ResourceGroupName crane -Location "China North" -VM $vmconfig  
```

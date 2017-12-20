---
title: "如何快速定位本地到 Azure 端 vpn 隧道连通性问题"
description: "如何快速定位本地到 Azure 端 vpn 隧道连通性问题"
author: Bingyu83
resourceTags: 'VPN Gateway, Connectivity'
ms.service: vpn-gateway
wacn.topic: aog
ms.topic: article
ms.author: biyu
ms.date: 12/20/2017
wacn.date: 12/20/2017
---

# 如何快速定位本地到 Azure 端 vpn 隧道连通性问题

VPN 隧道的连通性通常与 VPN 网关配置本身以及网关之间的运营商网络稳定性有关。

VPN 故障排查过程通常比较复杂，而客户在本地 vpn 设备做抓包或是联系 Azure 技术支持再做抓包通常需要较长的时间做准备，而且问题往往在开始抓包之前消失。

本文主要针对本地和 Azure 端正确配置了 VPN 相关参数，但仍然出现 VPN 隧道连通不稳定的情况，介绍了几种快速及时定位问题的方法，从而为快速排除故障提供了有力的支持。

## 使用 NMAP 工具

由于 Azure 使用 IKE 协议做 IPSec 通信之前的协商，使用的不是 TCP 协议端口，故不能使用简单的 telnet 或是 psping/paping 工具。这里为大家介绍的是 nmap 工具。

Linux 操作系统系统安装 nmap 方法：

- CentOS: `yum install nmap`<br>
- Ubuntu: `sudo apt-get install nmap`<br>
- Windows 操作系统: 在[此处](https://nmap.org/download.html)下载并安装。

由于 IKE 协商是基于 UDP 端口 500 或 4500（启用了 NAT-T），请参考如下方法在位于 Azure 上的与 Azure VPN 网关在同一个虚拟网络的虚拟机中使用 nmap 工具测试本地 vpn 设备协商用的公网 IP 的端口，以下以 Linux 虚拟机为例：

先探测此 IP 的 TCP 端口侦听情况：

### TCP:

```
E:~$ sudo nmap -sT 203.xxx.xxx.xxx

Starting Nmap 7.40 ( https://nmap.org ) at 2017-10-16 17:17 CST
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.04 seconds
```

可以看到此 IP 没有任何 TCP 端口处于侦听状态。

再确认此 IP 的所有 UDP 端口状态：

### UDP:

```
E:~$ sudo nmap -sU 203.xxx..xxx.xxx
 
Starting Nmap 7.40 ( https://nmap.org ) at 2017-10-16 17:22 CST
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.04 seconds
```

发现没有探测到这个 IP 的任何 UDP 端口是 UP 状态。因此判断这是 vpn 隧道断开的直接原因。问题的根本原因可以根据此现象做进一步定位。

### 对比正常的情况（举例测试）：

```
E :~$ sudo  nmap -sU 139.xxx.xxx.xxx
 
Starting Nmap 7.40 ( https://nmap.org ) at 2017-10-16 17:28 CST
Nmap scan report for 139.xxx.xxx.xxx
Host is up (0.0011s latency).
Not shown: 999 open|filtered ports
PORT    STATE SERVICE
500/udp open  isakmp
 
Nmap done: 1 IP address (1 host up) scanned in 16.67 seconds
```

可以看到，这个 IP 的 UDP 端口 500 即 IKE 所使用的协议端口是 UP 的。

此方法也同样适用于从本地网络网关到 Azure 的 VPN 网关的 IP 的 UDP 端口扫描，从而得出双向的连通性状态。

## 收集 VPN 网关的诊断日志

除了以上基本的端口连通性测试外，客户也可以通过 PowerShell 打开针对 Azure vpn 的诊断日志，具体步骤如下：

```PowerShell
# VNET Resource Group and Name
$rgName = "your resource name"
$vnetGwName = "your GW name"
$timestamp = get-date -uFormat "%d%m%y@%H%M%S"

# Details of existing Storage Account that will be used to collect the logs
$storageAccountName = "storage account name"
$storageAccountKey = "storage account key"
$captureDuration = 60
$storageContainer = "vpnlogs"
$logDownloadPath = "D:\vpnlogs (create the folder first)"
$Logfilename = "VPNDiagLog_" + $vnetGwName + "_" + $timestamp + ".txt"

# Set Storage Context and VNET Gateway ID
$storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

# NOTE: This is an Azure Service Manager cmdlet and so no AzureRM on this one.  AzureRM will not work as we don’t get the gatewayID with it.
$vnetGws = Get-AzureVirtualNetworkGateway

# Added check for only provisioned gateways as older deleted gateways of same name can also appear in results and capture will fail
$vnetGwId = ($vnetGws | ? GatewayName -eq $vnetGwName | ? state -EQ "provisioned").GatewayID

# Start Azure VNET Gateway logging
Start-AzureVirtualNetworkGatewayDiagnostics  `
    -GatewayId $vnetGwId `
    -CaptureDurationInSeconds $captureDuration `
    -StorageContext $storageContext `
    -ContainerName $storageContainer

# Optional – Test VNET gateway connection to another server across the tunnel 
# Only use this if you are connected to the local network you are connecting to FROM Azure. Otherwise create some traffic across the link from on prem.
# Test-NetConnection -ComputerName 10.0.0.4 -CommonTCPPort RDP

# Wait for diagnostics capturing to complete
Sleep -Seconds $captureDuration

# Step 6 – Download VNET gateway diagnostics log
$logUrl = ( Get-AzureVirtualNetworkGatewayDiagnostics -GatewayId $vnetGwId).DiagnosticsUrl
$logContent = (Invoke-WebRequest -Uri $logUrl).RawContent
$logContent | Out-File -FilePath $logDownloadPath\$Logfilename
```

通过下载出来的诊断日志分析定位问题，也可以将此针对日志提交到 Azure 技术支持案例中，由工程师做进一步分析。

以上两种方法为问题发生时提供了及时有效的排查手段，有利于问题的快速定位和解决。
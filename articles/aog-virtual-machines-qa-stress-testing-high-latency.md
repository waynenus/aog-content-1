---
title: 虚拟机压力测试延迟高的可能原因及 ILPIP 配置 / 查询脚本
description: 虚拟机压力测试延迟高的可能原因及 ILPIP 配置 / 查询脚本
services: Virtual Machines
documentationCenter: ''
author: JoiGE
manager: ''
editor: ''
tags: 'Virtual Machines, SNAT, ILPIP'

ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: yunjia.ge
ms.date: 08/01/2017
wacn.date: 08/01/2017
---

# 虚拟机压力测试延迟高的可能原因及 ILPIP 配置 / 查询脚本

## 问题描述

用户使用 Azure 虚拟机访问其他资源，做压力 / 负载测试（并发数非常高）时，可能会出现以下情况 ：

测试初期 Client VM 的延迟结果正常；
测试后期 Client VM 的延迟偶尔突增/连接失败，越后期超高延迟（比如 30 秒）出现越多；

## 问题分析

造成这一现象的根本原因很可能是 SNAT（Source NAT）数量过高。

### 什么是 SNAT 呢？

对于 ASM 虚拟机，出于可拓展性的考虑，云服务中每台虚拟机都隐藏在云服务面向公网的 VIP 之后，各自有自己的 DIP。

对于 ARM 虚拟机，默认会分配一个面向公网的 VIP 和一个面向内部的 DIP。用户可以自定义设置，不为 ARM 虚拟机分配公网 IP。

当这些未部署特定公网 IP 的虚拟机访问公网资源时，出站流量将通过 NAT 层（更改虚拟机的 DIP 和端口为面向公网的 VIP 和对应端口，以便流量被路由，否则虚拟机单凭 DIP 无法在公网中被路由）。

这个过程即 SNAT：更改源信息（源 IP 地址和源端口）。

### SNAT 数量建议上限是多少？

出于整体性能考虑，Azure 为每个虚拟机预分配了 160 个源端口。虚拟机访问公网资源时， Azure 会为该会话分配这 160 个中可用的端口。如果同时有 160 个请求正在运行，第 161 个及之后的请求需等 Azure 分配了新的端口才能使用。

当用户的并发非常高时，Azure 分配新端口的速度可能不足以保障每个请求都能及时获得 Azure 分配的 SNAT 端口。未被分配到端口的请求就可能出现出站连接尝试失败 / 延迟高的情况。


## 解决方法

- 资源管理器模式

    对于 ARM 部署的虚拟机，可以为它的网卡 NIC 配置公网 IP
    通过 Portal 或 PowerShell 创建公网 IP，具体操作步骤可以参考官网：[创建公共 IP 地址](https://docs.azure.cn/zh-cn/virtual-network/virtual-network-public-ip-address#create)。

    在 Portal 上虚拟机对应的 "**网络接口**" 内， 点击 **IP 配置**，配置对应的公共 IP 地址 即可。

- 经典模式

    一般对此的解决方法是为这台高负载的机器配置独有的公网 IP —— 实例层级公共 IP（ILPIP）。

    对于 ASM 部署的虚拟机，将创建的公网 IP 配置为虚拟机的 ILPIP：

    ILPIP（Instance Level Public IP） 为用户虚拟机提供了一个独有的公网 IP，于是用户可用此 IP 直接访问虚拟机，出站流量也无需作 SNAT。

    具体说明可参考官网：[实例层级公共 IP（经典）概述](https://www.azure.cn/documentation/articles/virtual-networks-instance-level-public-ip)。

    具体配置方法如下，具体参数请根据实际情况修改。

    1. 如何检索虚拟机的 ILPIP 信息：

        若要查看虚拟机的 ILPIP 信息，请运行以下 PowerShell 命令，然后观察 PublicIPAddress 和 PublicIPName 的值：

        ```PowerShell
        Get-AzureVM -Name FTPInstance -ServiceName FTPService
        ```

        预期输出：

        ```
        DeploymentName	: FTPService
        Name				: FTPInstance
        Label				: 
        VM					: Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
        InstanceStatus				: ReadyRole
        IpAddress                   	: 100.74.118.91
        InstanceStateDetails    	: 
        PowerState                  	: Started
        InstanceErrorCode         	: 
        InstanceFaultDomain       	: 0
        InstanceName               	: FTPInstance
        InstanceUpgradeDomain     	: 0
        InstanceSize              	: Small
        HostName                  	: FTPInstance
        AvailabilitySetName     	: 
        DNSName                   	: http://ftpservice888.chinacloudapp.cn/
        Status                     	: ReadyRole
        GuestAgentStatus           	:     Microsoft.WindowsAzure.Commands.ServiceManagement.Model.GuestAgentStatus
        ResourceExtensionStatusList	: {Microsoft.Compute.BGInfo}
        PublicIPAddress         		: 104.43.142.188
        PublicIPName             	: ftpip
        NetworkInterfaces         	: {}
        ServiceName                	: FTPService
        OperationDescription       	: Get-AzureVM
        OperationId               	: 568d88d2be7c98f4bbb875e4d823718e
        OperationStatus            	: OK
        ```

    2. 如何向现有的虚拟机配置 ILPIP

        运行以下 PowerShell 命令（仅支持使用 Powershell 配置 ILPIP），下面的示例为 FTPInstance 这台虚拟机新建了一个名为 ftpip2 的 ILPIP：

        ```
        Get-AzureVM -ServiceName FTPService -Name FTPInstance | Set-AzurePublicIP -PublicIPName ftpip2 | Update-AzureVM
        ```

    ### 配置 ILPIP 是否会造成虚拟机重启？

    用户可能担心此操作是否会造成虚拟机重启、业务中断等等。

    配置 ILPIP 不会导致虚拟机重启，配置成功后 Cloud Service IP 和 ILPIP 都可以用来连接虚拟机。

    ### ILPIP 能否被保留 / 设置为静态？

    ILPIP 不可设置为静态。如有需要建议部署 ARM 的虚拟机，并设置虚拟机的 Public IP（VIP）为 Reserved IP。

    ### 统计现有的 ILPIP

    ILPIP 单独计价，具体价格可以参考：[IP 地址价格详情](https://www.azure.cn/pricing/details/reserved-ip-addresses/)。

    每个订阅最多可以分配 5 个 ILPIP 地址。然而，查看订阅下已经使用了多少个 ILPIP 只能通过查询每一台虚拟机看是否有 PublicIP Address 项，过程比较费时。

    具体查询的 PowerShell 命令如下：

    ```PowerShell
    $vm = Get-AzureVM -Name xxx -ServiceName xxx
    ```

    其中 `$vm.publicIPAddress` 即 ILPIP。
    为便于查询某订阅下的所有 ILPIP，我们在 GitHub 中提供[自动化获取订阅下所有 ILPIP 的脚本](https://github.com/wacn/AOG-CodeSample/blob/master/VirtualMachines/PowerShell/ILPIPconfig.ps1)供用户使用，直接下载后在 PowerShell 中运行即可。
    
    ### ILPIP 带来的安全问题

    配置 ILPIP 后相当于直接将虚拟机暴露在公网，安全方面建议在虚拟机内部配置防火墙。
    更多详细的安全策略解释可以参考：[Microsoft 云服务和网络安全性](https://docs.microsoft.com/zh-cn/azure/best-practices-network-security)，这里也提到了外围网络的安全概念和具体示例。


---
title: 在 Microsoft Azure(IaaS) 中部署 SQL Server AlwaysOn 可用性组的建议及最佳实践
description: 在 Microsoft Azure(IaaS) 中部署 SQL Server AlwaysOn 可用性组的建议及最佳实践
documentationCenter: ''
author: v-DavidTang
manager: ''
editor: ''
tags: 'SQL Server, AlwaysOn, AlwaysOn Availability Groups, Transactions'

ms.service: virtual-machines-windows
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 08/15/2017
wacn.date: 08/15/2017
---
# 在 Microsoft Azure(IaaS) 中部署 SQL Server AlwaysOn 可用性组的建议及最佳实践

![wsfc-cluster](./media/aog-virtual-machines-windows-sql-server-translate-recommendations-and-best-practices-when-deploying-sql-server-alwayson-availability-groups-in-microsoft-azure-iaas/wsfc-cluster.png)

## 简介

Microsoft Azure SQL Server 虚拟机（VMs）可帮助降低数据库高可用性和灾难恢复（HADR）解决方案的成本。Azure 虚拟机支持大多数 SQL Server HADR 解决方案，包括 **Azure-only** 和 **混合解决方案**。
在 IaaS 环境中成功部署 AlwaysOn 可用性组有许多重要的注意事项和特殊配置。此博客列出了在 Windows Azure 中部署可用性组时应解决的关键注意事项。

## IaaS 中 Windows 和 Cluster 的最佳实践

### 心跳检测建议

[**IaaS 中的 SQL AlwaysOn  - 调整故障转移集群网络阈值**](http://blogs.msdn.com/b/alwaysonpro/archive/2014/06/02/iass-with-sql-alwayson-tuning-failover-cluster-network-thresholds.aspx)

在 IaaS 中使用 SQL Server AlwaysOn 运行 Windows 故障转移集群时，建议将集群设置更改为更松懈的监视状态。 除此之外的集群设置是被限制的，以避免导致不必要的中断。 默认设置是专为高度优化的内部网络而设计的，并没有考虑由多租户环境（如 Windows Azure（IaaS））而引起的诱发延迟的可能性。

> [!NOTE]
> 将集群心跳设置重置为默认设置时会出现一个已知问题。要解决此问题，请应用以下修补程序：

- Windows 2012 R2 和 Windows 2008 SP2 版本

    [将运行在的 Windows Server 2012 R2 或 Windows Server 2008 SP2 上的集群节点的属性恢复为默认值](https://support.microsoft.com/zh-cn/help/2898118/changed-cluster-properties-revert-to-default-values-on-cluster-nodes-t)

- Windows 2012 版本

    [将运行在的 Windows Server 2012 上的集群节点的属性恢复为默认值](https://support.microsoft.com/zh-cn/help/2935773/changed-cluster-properties-revert-to-default-values-on-cluster-nodes-t)

### 仲裁模式建议

[WSFC 仲裁模式和投票配置 (SQL Server)](https://docs.microsoft.com/zh-cn/sql/sql-server/failover-clusters/windows/wsfc-quorum-modes-and-voting-configuration-sql-server)

SQL Server AlwaysOn 可用性组利用 Windows Server 故障转移集群 (WSFC) 来作为平台技术。 WSFC 使用一种基于仲裁的方法来监视集群的整体运行状况，并且最大限度地提高节点级别的容错能力。 WSFC 仲裁模式和节点投票配置对于了解 Always On 高可用性和灾难恢复解决方案的设计、操作及故障排除的非常重要。

[AlwaysOn 可用性组中的仲裁投票配置检查](https://blogs.msdn.microsoft.com/sqlalwayson/2012/03/13/quorum-vote-configuration-check-in-alwayson-availability-group-wizards-andy-jing/)

在本博客中，我们深入了解调整 Windows Server 故障转移集群（WSFC）中可用性组的仲裁投票的准则，并以具体示例解释其背后的原因。

[故障转移集群和 AlwaysOn 可用性组 (SQL Server)](https://docs.microsoft.com/zh-cn/sql/database-engine/availability-groups/windows/failover-clustering-and-always-on-availability-groups-sql-server)

AlwaysOn 可用性组（ SQL Server 2012 中引入的高可用性和灾难恢复解决方案）需要 Windows Server 故障转移集群 (WSFC)。

WSFC 集群的总体运行状况是由集群中节点的仲裁投票决定的。 如果 WSFC 集群由于计划外的灾难而导致脱机，或由于持续的硬件或通信故障，则需要管理员手动干预。 Windows Server 或 WSFC 集群管理员将需要“强制仲裁”，并在非容错配置中将仍有效的集群节点变为联机状态。

### DIP/VIP 设置静态 IP

[Windows Azure 虚拟网络中的设备网络隔离选项]( http://azure.microsoft.com/blog/2014/03/28/network-isolation-options-for-machines-in-windows-azure-virtual-networks/)

应用程序隔离是企业环境中的一个重要问题，因为企业客户希望保护各种环境免受未经授权或不必要的访问。这包括经典的前端和后端场景，其中特定的后端网络或子网络中的机器只能允许某些客户端，其他计算机则基于 IP 地址白名单连接到特定的端点。这些场景可以轻而易举的在 Microsoft Azure 中实现，在 Azure 环境中客户端应用程序可以通过互联网访问虚拟机应用程序服务器，或者也可以通过 VPN 连接从内部网络进行访问。

## 存储账户

[存储空间-性能设计](https://social.technet.microsoft.com/wiki/contents/articles/15200.storage-spaces-designing-for-performance.aspx)

当使用最佳实践进行准备和配置时，存储空间可以在 Azure 中提供高性能存储。 但是，有些变量需要注意，这些变量可能会影响部署在存储空间中的磁盘的性能。

例如，如果标准层虚拟机中频繁使用的 VHD 数量接近 40，则性能可能会降低。有关详细信息，请参阅[配置 Azure 虚拟机以获得最佳存储性能](https://blogs.msdn.microsoft.com/mast/2014/10/14/configuring-azure-virtual-machines-for-optimal-storage-performance/)。

同样应该参考 SQL Server 最佳实践通过部署多个磁盘来优化存储和提高 IOPS 性能。有关更多信息，请参阅“[Azure 虚拟机中的 SQL Server 性能指南](https://docs.azure.cn/zh-cn/virtual-machines/windows/sql/virtual-machines-windows-sql-performance)” 中的 “ Windows Azure 虚拟机磁盘和缓存设置 ” 和 “ 数据磁盘性能选项和注意事项 ” 部分。同样还可以查看 [Azure 虚拟机中 SQL Server 性能最佳实践](https://docs.azure.cn/zh-cn/virtual-machines/windows/sql/virtual-machines-windows-sql-performance#io) 中的 “ I/O 指导原则 ” 部分。

此外，请考虑使用此脚本：[自动创建预先配置为最大存储性能的 Azure VM](https://gallery.technet.microsoft.com/Automate-the-creation-of-e41aecf6) 创建针对最大存储性能进行了优化的 Microsoft Azure 虚拟机。

## IaaS 中的 SQL Server 最佳实践

### 在 Windows Azure 中部署 SQL Server AlwaysOn 可用性组

提供以下范围从分步配置到部署 AlwaysOn 可用性组到完全自动部署。
使用新版  [SQL Server 2014 AlwaysOn](https://portal.azure.cn/)库。

- 教程：[Azure AlwaysOn 可用性组（PowerShell）](https://docs.azure.cn/zh-cn/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups) 
- 教程：[Azure AlwaysOn 可用性组（GUI）](https://docs.azure.cn/zh-cn/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups) 

## IaaS 中的 SQL Server 最佳实践

[Azure 虚拟机中 SQL Server 的性能最佳实践](https://docs.azure.cn/zh-cn/virtual-machines/windows/sql/virtual-machines-windows-sql-performance)

参见最佳实践的**快速检查列表**，以确保在 Microsoft Azure（IaaS） 环境中运行时您实现了 SQL Server 优化选项。
[Azure 虚拟机中 SQL Server 的性能指南](http://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/Performance%20Guidance%20for%20SQL%20Server%20in%20Windows%20Azure%20Virtual%20Machines.docx) 

参见性能指南白皮书，进行深入分析和建议以便在 Microsoft Azure（IaaS）环境中运行时优化 SQL Server 性能。
 
在部署 AlwaysOn 可用性组时，建议使用 [Azure 虚拟机中 SQL Server 的性能指南白皮书](http://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/Performance%20Guidance%20for%20SQL%20Server%20in%20Windows%20Azure%20Virtual%20Machines.docx) 和 [Azure 虚拟机中 SQL Server 的性能最佳实践](https://docs.azure.cn/zh-cn/virtual-machines/windows/sql/virtual-machines-windows-sql-performance)部署 SQL Server，以最大限度地提高 AlwaysOn 可用性组运行的性能。

## 创建 Azure 可用性组侦听器

[教程：AlwaysOn 可用性组的侦听器配置](https://docs.azure.cn/zh-cn/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener) 

AlwaysOn 可用性组监听器需要独特的配置步骤。由于 Microsoft Azure 虚拟机具有静态 IP 地址限制，必须配置可用性组侦听器才能实现云服务 IP 地址，以连接到可用性组数据库。

请使用此链接[教程：AlwaysOn 可用性组的侦听器配置](https://docs.azure.cn/zh-cn/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener)，了解有关配置 Azure 可用性组侦听器的分步说明。

## 测试 Azure 可用性组侦听器

正确配置后，可以从互联网上的任何地方连接 Azure 可用性群组侦听器。

> [!IMPORTANT]
> 在位于同一云服务中托管主副本 SQL Server 的工作站或服务器上测试 Azure 侦听器并不是有效的，将会失败。 Azure 侦听器是专为从不同云服务运行或通过互联网连接的应用程序而设计的。

例如，在与可用性组不同的云服务上运行的工作站，以及安装了 SQL Server 客户端工具时，请使用以下命令行来进行测试可用性组主副本的 SQL Server 连接：

```
sqlcmd –S "<CloudServiceDNSName>,<EndpointPort>" -d "<DatabaseName>" -Q "select @@servername, db_name()" -l 15
```

可以通过使用 Azure 门户查看托管 AlwaysOn 可用性组的云服务仪表板来查找 DNS 名称。

终结点端口是指在[教程：AlwaysOn 可用性组的侦听器配置](https://docs.azure.cn/zh-cn/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener) 中 Azure 侦听器设置的第八步中指定的公共端口。

## Azure AlwaysOn 可用性组侦听器中使用 ReadIntent 路由

[Azure AlwaysOn 可用性组侦听器中使用 ReadIntent 路由](https://blogs.msdn.microsoft.com/alwaysonpro/2014/03/31/use-readintent-routing-with-azure-alwayson-availability-group-listener/) 

Azure 侦听器可以配置为 ReadIntent 路由连接请求。当部署在 Microsoft Azure 中时，它需要独特的配置步骤。有关如何使用 Azure 侦听器配置 ReadIntent 路由的更多信息和分步说明，请查看[Azure AlwaysOn 可用性组侦听器中使用 ReadIntent 路由](https://blogs.msdn.microsoft.com/alwaysonpro/2014/03/31/use-readintent-routing-with-azure-alwayson-availability-group-listener/)以正确配置 Azure 侦听器进行只读路由。
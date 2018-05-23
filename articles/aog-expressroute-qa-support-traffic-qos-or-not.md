
---
title: "ExpressRoute 是否支持网络流量的 QOS"
description: "本文主要针对用户本地网络通过 ExpressRoute 专线连通到 Azure 相应的虚拟网络内部资源后，在网络传输中是否支持 QOS 做说明。"
author: Bingyu83
resourceTags: 'ExpressRoute'
ms.service: expressroute
wacn.topic: aog
ms.topic: article
ms.author: biyu
ms.date: 4/30/2018
wacn.date: 4/30/2018
---

# ExpressRoute 是否支持网络流量的 QOS

本文主要针对用户本地网络通过 ExpressRoute 专线连通到 Azure 相应的虚拟网络内部资源后，在网络传输中是否支持 QOS 做说明。

## 问题描述

本地网络到 Azure 虚拟网络的流量在高峰时段可能存在拥塞，即流量达到甚至超出 ExpressRoute 专线带宽。

客户本地网络边界设备，即 Azure MSEE 路由设备的对端 - BGP peer，是否可以配置基于 DSCP 标记的 QOS 策略。

## 问题分析

目前（截止到本文撰写完成时）针对这种场景，网络传输的双方是本地网络和 Azure 的虚拟网络，这是我们所说的 Azure 专用对等互联（Azure private peering），ExpressRoute 的 MSEE 路由设备**不识别**专用对等互连中传输的网络包中的 DSCP 标记。

即带有 DSCP 标记的网络包到达 Azure MSEE 后，DSCP 标记会被被重置为 0，即 Best effort。<br>
因此本地网络到 Azure 虚拟网络的流量在 ExpressRoute 的 MSEE 路由设备不支持 QOS。

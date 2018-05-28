---
title: "Azure 计费常见问题 - 数据传输"
description: "Azure 计费常见问题 - 数据传输"
author: v-DavidTang
resourceTags: 'Azure Billing, Data Transfer, Virtual Network, FAQ'
ms.service: billing
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 5/28/2018
wacn.date: 3/31/2018
---

# Azure 计费常见问题 - 数据传输

- [问题一：Windows Azure 能提供的流量带宽是多少？数据传输 Data Transfer 是如何收费的？](#section1)
- [问题二：Azure 上两台虚拟机之间通过公网 IP（VIP）通信如何收费？](#section2)
- [问题三：一台虚拟机通过公网 IP 访问另一台虚拟机时，数据传输如何计费？](#section3)
- [问题四：哪些服务会产生标准数据流量？](#section4)
- [问题五：S2SVPN 流量如何计费？](#section5)
- [问题六：Virtual Network 如何计费？](#section6)

## <a id="section1"></a>问题一：Windows Azure 能提供的流量带宽是多少？数据传输 Data Transfer 是如何收费的？

1. 由世纪互联运营的 Microsoft Azure (以下简称为 “中国区 Azure”)提供的是 BGP 3 线带宽，此 3 线宽带包括移动、电信和联通。
2. 中国区 Azure 提供的带宽是免费的共享带宽。
3. 关于带宽大小，由于涉及世纪互联和微软的商业运营，此处不便提供相关信息，你可以通过试用账号自行测试。
4. 关于免费流量，目前我们每月提供的免费流量信息如下：1 元试用账户每月 50GB（传入传出各 50GB），标准预付费账户每月 1TB（传入），企业用户 20TB（传入）。超过部分按照 0.67/GB 开始计费。

## <a id="section2"></a>问题二：Azure 上两台虚拟机之间通过公网 IP（VIP）通信如何收费？

1. 两台虚拟机在同一数据中心时（都在中国东部或都在中国北部），VM1 访问 VM2 公网 IP 不经过 Internet，没有出数据中心，不产生流量费用。
2. 两台虚拟机在不同的数据中心时，VM1 访问 VM2 公网 IP 跨数据中心，必须要通过 Internet，会产生流量费用。

## <a id="section3"></a>问题三：一台虚拟机通过公网 IP 访问另一台虚拟机时，数据传输如何计费？

在 Azure 中，可以在一个云服务下部署多台虚拟机，云服务会分配一个公网 IP，如 42.159.0.1；云服务下的虚拟机会分配不同的内部 IP（DIP），如 10.0.0.4 和 10.0.0.5；同一云服务下的虚拟机互相访问会直接走内部 IP；当您用云服务 40.50.60.7 下的虚拟机去访问另一个云服务 42.159.0.2 下的虚拟机时，才会走公网 IP，只要这两个云服务在同一个数据中心，就不收取流量费用。

## <a id="section4"></a>问题四：哪些服务会产生标准数据流量？

除虚拟机、存储访问和网站等常用服务，P2S VPN 出站以及 CND 回源请求等官网明确写明会产生普通流量的服务。

## <a id="section5"></a>问题五：S2S VPN 流量如何计费？

只要通过 Internet 的都要按照正常的数据流量的费用来收，所以没有特别写出来。

1. S2S 的 VPN 只要发生流量，就会单独计费。
2. 计费标准是和 P2S 一样，按照数据传输的费率来收。

## <a id="section6"></a>问题六：Virtual Network 如何计费？

Virtual Network 本身不计费，只有当您在 Virtual Network 中搭建 VPN 网关时才会产生网关和数据传输的费用。账单中 Virtual Network 的费用其实是网关的费用。
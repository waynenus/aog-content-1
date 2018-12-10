---
title: "如何实现基于现有应用程序网关针对个别客户端 IP 或端口的请求做阻断"
description: "如何实现基于现有应用程序网关针对个别客户端 IP 或端口的请求做阻断"
author: Bingyu83
resourceTags: 'Application Gateway, NSG'
ms.service: application-gateway
wacn.topic: aog
ms.topic: article
ms.author: biyu
ms.date: 12/5/2018
wacn.date: 12/5/2018
---

# 如何实现基于现有应用程序网关针对个别客户端 IP 或端口的请求做阻断

## 背景介绍

无论是基于 Azure PaaS Web 应用还是 Azure IaaS 虚拟机的客户自行搭建的 Web Service，对外（internet）提供服务时，都会遇到网络攻击行为，客户一般会在 Web Service 前端部署具备 WAF 功能的 Azure 应用程序网关亦或是 Azure 镜像市场下载部署的第三方的 WAF 产品。

## 应用场景

使用了应用程序网关，通过访问日志或是防火墙日志亦或是 Web Server 的相关日志，检查到有客户端攻击行为或是安全渗透探测行为，如何基于已有的应用程序网关针对这些已知 IP 做阻断。

由于目前 Azure 应用程序网关还未支持基于网络层的安全策略控制，因此不能针对 IP 或 TCP/UDP 端口做安全控制策略，我们现阶段需要通过其他替代手段来实现，本文为您介绍的是 NSG 策略。

众所周知，应用程序网关部署在虚拟网络中的子网中，既然是在虚拟网络子网中，我们可以就通过 NSG 来实现访问控制。有了 NSG 策略，我们便可以实现针对特定流量做基于传统五元组的访问控制。

## 问题详述

> [!IMPORTANT]
> 在限制访问流量的同时，需要注意平台有一些监控检查以及控制层面的数据流也会受影响，故如果需要配置 deny all 策略，建议在这条 deny all 策略之上，允许所有特定平台的流量，具体可以参考：[应用程序网关配置常见问题](https://docs.azure.cn/application-gateway/application-gateway-faq#configuration)。

**问：是否可以将一些源 IP 的应用程序网关访问权限列入允许列表？**

对应用程序网关子网使用 NSG 可以完成此方案。应按列出的优先顺序对子网采取以下限制：

* 允许来自源 IP/IP 范围的传入流量。
* 允许来自所有源的请求传入端口 65503-65534，进行后端运行状况通信。 此端口范围是进行 Azure 基础结构通信所必需的。 它们受 Azure 证书的保护（处于锁定状态）。 如果没有适当的证书，外部实体（包括这些网关的客户）将无法对这些终结点做出任何更改。
* 允许 NSG 上的传入 Azure 负载均衡器探测（AzureLoadBalancer 标记）和入站虚拟网络流量（VirtualNetwork 标记）。
* 使用“全部拒绝”规则阻止其他所有传入流量。
* 允许发往 Internet 的所有目标的出站流量。

## 参考文档

* [是否可以将一些源 IP 的应用程序网关访问权限列入允许列表](https://docs.microsoft.com/azure/application-gateway/application-gateway-faq#can-i-whitelist-application-gateway-access-to-a-few-source-ips)
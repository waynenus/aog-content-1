---
title: "如何使用 Azure 门户配置 HTTP 流量重定向 HTTPS 的多站点应用程序网关"
description: "如何使用 Azure 门户配置 HTTP 流量重定向 HTTPS 的多站点应用程序网关"
author: YaoYuejian
resourceTags: 'Application Gateway , Https , Sites'
ms.service: application-gateway
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/14/2018
wacn.date: 11/14/2018
---
# 如何使用 Azure 门户配置 HTTP 流量重定向 HTTPS 的多站点应用程序网关

如下以 `site1.21vianet.party` 和 `site2.21vianet.party` 两个站点为例进行部署。

介绍如何执行下列操作：

1. 添加后端池
2. 配置自定义探测
3. 配置 HTTP 设置
4. 添加多站点侦听器
5. 配置重定向规则

以下是基于已创建的应用程序网关配置的步骤：

## 添加后端池

![01](media/aog-application-gateway-howto-config-numbers-sites/01.png "01")

## 添加自定义探测：

对于多站点的应用程序网关，需要使用自定义的运行状况探测。

对应在 “**设置**” - “**运行状况探测**” 中添加，主机字段填写各自的域名。

Site1 配置如下：

![02](media/aog-application-gateway-howto-config-numbers-sites/02.png "02")

Site2 配置如下：

![03](media/aog-application-gateway-howto-config-numbers-sites/03.png "03")

## 添加 HTTP 设置

在 “**设置**” - “**HTTP 设置**” 中分别添加 HTTP 设置，注意调用自定义运行状况探测。

Site1 配置如下：

![04](media/aog-application-gateway-howto-config-numbers-sites/04.png "04")

Site2 配置如下：

![05](media/aog-application-gateway-howto-config-numbers-sites/05.png "05")

## 添加前端 IP 地址：

由于测试域名未备案，所以通过内网 IP 进行部署，在 “**设置**” - “**前端 IP 配置**” 中添加专用 IP。

实际环境中可以对应更换为公网 IP 地址。

![06](media/aog-application-gateway-howto-config-numbers-sites/06.png "06")

## 添加多站点侦听器

在 “**设置**” - “**侦听器**” 中添加创建 Site1 的 HTTPs 侦听器，并上传 PFX 证书。

![07](media/aog-application-gateway-howto-config-numbers-sites/07.png "07")

创建 Site2 的 HTTPs 侦听器，并上传 PFX 证书。

![08](media/aog-application-gateway-howto-config-numbers-sites/08.png "08")

## 配置规则（对应 HTTPS 的流量）

在 “**设置**” - “**规则**” 中添加 Site1_https_rule 配置如下：

![09](media/aog-application-gateway-howto-config-numbers-sites/09.png "09")

Site2_https_rule 配置如下：

![10](media/aog-application-gateway-howto-config-numbers-sites/10.png "10")

以上，已经完成多站点的 HTTPS 的应用程序网关，接下来要进行 HTTP 重定向的配置。

## 配置 HTTP 侦听器

直接按照如下配置添加会出现报错：

![11](media/aog-application-gateway-howto-config-numbers-sites/11.png "11")

![12](media/aog-application-gateway-howto-config-numbers-sites/12.png "12")

此时需要删除默认侦听器，在门户上直接删除即可。

## 再次添加 HTTP 侦听器

Site1 HTTP 侦听器：

![13](media/aog-application-gateway-howto-config-numbers-sites/13.png "13")

Site2 HTTP 侦听器：

![14](media/aog-application-gateway-howto-config-numbers-sites/14.png "14")

## 配置重定向规则

在 “**设置**” - “**规则**” 中，添加重定向配置。

Site1 重定向规则如下：

![15](media/aog-application-gateway-howto-config-numbers-sites/15.png "15")

Site2 重定向规则如下：

![16](media/aog-application-gateway-howto-config-numbers-sites/16.png "16")

## 测试结果

![17](media/aog-application-gateway-howto-config-numbers-sites/17.png "17")

![18](media/aog-application-gateway-howto-config-numbers-sites/18.png "18")

![19](media/aog-application-gateway-howto-config-numbers-sites/19.png "19")

![20](media/aog-application-gateway-howto-config-numbers-sites/20.png "20")
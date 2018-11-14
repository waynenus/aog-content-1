---
title: "如何使 Azure Web 应用只允许从指定的网站进行访问"
description: "如何使 Azure Web 应用只允许从指定的网站进行访问"
author: Chris-ywt
resourceTags: 'App Service Web, Acess Premission'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/8/2018
wacn.date: 11/8/2018
---

# 如何使 Azure Web 应用只允许从指定的网站进行访问

## 问题描述

当 Azure Web 应用中有两个网站 A 和 B，现在要令 B 网站只允许 A 网站进行访问，不容许其他的网站访问，改如何进行配置？

## 问题分析

对于所有用户只要网站部署在同一个 FTPS 区域，那么它们的 Web 应用入口 ip（访问 Azure Web 应用的 ip）和 出口 ip（Azure Web 应用访问其他服务的 ip）都是一样的。由于 Azure Web 应用服务这样共享架构的设计，无法通过限制 ip 来阻止访问。而且入口 ip 和出口 ip 也是不同的，出口 ip 有 4 个是随机分配的。由于出口 ip 是随机的，也是无法通过限制域名来阻止访问。因此目前可以在应用层方面配置 rewrite 规则，只有符合某种条件才可以访问网站 B。

## 解决方法

用户可以通过站点的 **web.config** 文件配置 rewrite 规则，并将该配置上传到 **site/wwwroot/** 目录下。下面是网站 Web 应用 B 只允许 url 包含 *password=123456* 才能访问配置。在 rewrite 下添加如下的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="role1" patternSyntax="ECMAScript" stopProcessing="true">
                    <match url=".*" negate="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="TypeInReasonHere" statusDescription="TypeInDescriptionHere" />
                    <conditions>
                        <add input="{QUERY_STRING}" pattern="^.*password=123456.*$" negate="true" />
                    </conditions>
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

只有通过 *password=123456* 才可以被访问，其他 url 都无法访问。不同的 url 访问结果如下图所示：

![url-123456](media/aog-app-service-web-qa-config-permission-only-allow-specified-site-to-access/url-123456.jpg "url-123456")

![no-url](media/aog-app-service-web-qa-config-permission-only-allow-specified-site-to-access/no-url.jpg "no-url")

![url-123](media/aog-app-service-web-qa-config-permission-only-allow-specified-site-to-access/url-123.jpg "url-123")

关于 rewrite  相关配置可以参考链接：[URL Rewrite Module 参考配置](https://docs.microsoft.com/zh-cn/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)
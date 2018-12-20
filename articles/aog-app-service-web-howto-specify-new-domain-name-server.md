---
title: "如何为 Web 应用程序指定新的域名解析服务器"
description: "如何为 Web 应用程序指定新的域名解析服务器"
author: maysmiling
resourceTags: 'App Service Web, Domain Name Server'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-amazha
ms.date: 12/20/2018
wacn.date: 12/20/2018
---

# 如何为 Web 应用程序指定新的域名解析服务器

在 **【应用程序设置】** 菜单里，可以添加键值对：

|应用设置名称 |值|
|:---:|:---:|
|WEBSITE_DNS_SERVER|8.8.8.8（公网的域名解析服务器 IP 地址）|

另外，我们还可以增加备用的域名解析服务器以备不时之需，如下：

|应用设置名称 |值|
|:---:|:---:|
|WEBSITE_ALT_DNS_SERVER|168.63.129.16 （Azure 域名解析服务器 IP 地址）|

![01](media/aog-app-service-web-howto-specify-new-domain-name-server/01.png "01")
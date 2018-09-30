---
title: 'Node JS 搭建的 Azure Web 应用如何修改响应 Header 的 Cache-Control 字段'
description: 'Node JS 搭建的 Azure Web 应用如何修改响应 Header 的 Cache-Control 字段'
author: hylinux
resourceTags: 'App Service Web, Node JS, Cache-Control'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: hongwei.guo
ms.date: 08/31/2018
wacn.date: 09/03/2018
---

# Node JS 搭建的 Azure Web 应用如何修改响应 Header 的 Cache-Control 字段

请按照以下步骤进行配置：

1. 在 web.config 中配置 `Cache-Control` 属性：

    ```xml
    <configuration>
        <sytem.webServer>
            <httpProtocol>
                <customHeaders>
                    <remove name="Cache-Control" />
                    <add name="Cache-Control" value="public, max-age=99" />
                </customHeaders>
            </httpProtocol>
        </system.webServer>
    </configuration>
    ```

2. 将静态内容和动态内容分开处理：

    ```xml
    <!-- First we consider whether the incoming URL matches a physical file in the /public folder -->
    <rule name="StaticContent">
        <action type="Rewrite" url="public{PATH_INFO}"/>
    </rule>
    <!-- All other URLs are mapped to the node.js site entry point -->
    <rule name="DynamicContent">
        <conditions>
        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="True"/>
        </conditions>
        <action type="Rewrite" url="server.js"/>
    </rule>
    </rules>
    ```

## 参考文档

- [Client Cache <clientCache>](https://docs.microsoft.com/zh-cn/iis/configuration/system.webServer/staticContent/clientCache)
- [管理 Web 内容的到期时间](https://docs.microsoft.com/zh-cn/azure/cdn/cdn-manage-expiration-of-cloud-service-content)
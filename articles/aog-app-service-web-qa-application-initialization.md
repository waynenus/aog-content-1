---
title: 如何预热 Azure Web 应用
description: 如何预热 Azure Web 应用
service: ''
resource: App Service Web
author: Chris-ywt
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'App Service Web, Application Initialization'
cloudEnvironments: MoonCake

ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 09/22/2017
wacn.date: 09/22/2017
---
# 如何预热 Azure Web 应用

## 问题描述

Azure Web 应用在自动缩放时会初始化新的实例，这时前端负载均衡器会将新的用户请求转发给新的实例，如果新的实例初始化需要较长时间，此用户请求有可能会返回 `502.3` 错误。

## 解决方案

通过在 web.config 中配置 `ApplicationInitialization` 属性来解决该问题。

Application Initialization 模块能够预热 Web 应用程序，使得在应用程序初始化完成之前，客户端的请求将会继续发送到原来的实例上，直到应用程序完全加载完成，才会将请求发送到新增加的实例。

以 ASP.NET 应用为例，具体的 web.config 配置如下

```XML
<?xml version="1.0"?>
<configuration>
  <system.web>
      <compilation debug="true" targetFramework="4.5.2" />
      <httpRuntime targetFramework="4.5.2" />
    </system.web>
  <system.webServer>
  <applicationInitialization  remapManagedRequestsTo="loading.htm">
    <add initializationPage="/default.aspx" />
  </applicationInitialization>
</system.webServer>
</configuration>
```

用 `remapManagedRequestsTo` 属性指定在网站初始化期间，将用户请求导向所指定的提示页面。

用 `initializationPage` 属性指定一个页面，例如此页面可以运行用户代码进行初始化工作或者检测初始化进度，Azure 平台会在启动后访问此页面，当此页面返回 `200` 时，Azure 平台即认为初始化完成。

##参考链接

[Application Initialization <applicationInitialization>](https://docs.microsoft.com/zh-cn/iis/configuration/system.webServer/applicationInitialization/)
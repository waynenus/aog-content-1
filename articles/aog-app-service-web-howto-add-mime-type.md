---
title: "Web 应用如何添加新的 MIME 类型"
description: "Web 应用如何添加新的 MIME 类型"
author: maysmiling
resourceTags: 'App Service Web, MIME'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-amazha
ms.date: 10/31/2017
wacn.date: 10/31/2017
---

# Web 应用如何添加新的 MIME 类型

用户可以通过 Web.config 对站点添加 MIME 类型。以 .apk 文件为例，配置内容如下：

```XML
<configuration>
   <system.webServer>
        <staticContent>
            <mimeMap fileExtension=”.apk” mimeType=”application/vnd.android” />
        </staticContent>
   </system.webServer>
</configuration>
```

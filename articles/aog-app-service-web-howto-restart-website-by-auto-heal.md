---
title: "通过 AutoHeal 方式重启网站"
description: "通过 AutoHeal 方式重启网站"
author: Chris-ywt
resourceTags: 'App Service Web, AutoHeal'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-ciwu
ms.date: 12/20/2018
wacn.date: 12/20/2018
---

# 通过 AutoHeal 方式重启网站

Azure Web 应用服务的框架中，文件存储服务器和运行工作服务器是分离的。即用户的代码的存储位置是在一台专门的 FTP 服务器上，通过类似于共享文件夹的功能一样链接到所运行的服务器上。当后台正在发生 storage failed 平台维护的时候，会导致 Azure Web 应用程序文件服务器路径发生变化，造成网站的重启，有时候您网站进程可能会不能适应，持续发生 502 错误。用户可以通过 AutoHeal 方式自动重启网站。

AutoHeal 会根据所选设置（例如配置更改、请求、基于内存的限制或执行请求所需的时间）回收应用的工作进程。在大多数情况下，回收进程是在出现问题后进行恢复的最快方式。尽管始终可以从 Azure 门户直接重新启动 Web 应用，但 AutoHeal 可以自动执行此操作。只需在 Web 应用的根 web.autoheal.config 中添加 monitoring 一些触发器即可，并将该文件上传到 wwwroot 目录下。请注意，即使应用程序并非 .Net 应用程序，这些设置的工作方式也仍然相同。

具体所选配置如下：

1. 通过请求数进行回收
2. 基于内存限制的自定义操作（或回收/日志记录）
3. 基于 HTTP 状态代码记录事件（或回收）
4. 通过请求时间长回收网站

```xml
<system.webServer>
    <monitoring>
        <triggers>
            <request count="5" timeInterval="00:00:30">// 配置在 30s 访问 5 次
            <memory privateBytesInKB="1000000"/>// 进程内存大于 1GB
            <statusCode>
                <add statusCode="500" subStatusCode="100" win32StatusCode="0" count="10" timeInterval="00:00:30">// 10 个请求导致 http 状态代码为 500, 最后30秒的子状态代码为 100
            </statusCode>
            <slowRequests timeTaken="00:00:05" count="1" timeInterval="00:00:30">// 在 30s 内消耗超过 5s 的请求数为 1
        </triggers>
        <actions value="Recycle"/>
    </monitoring>
</system.webServer>
```
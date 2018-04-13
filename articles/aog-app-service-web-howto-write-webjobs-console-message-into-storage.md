---
title: "如何将 Web 作业中控制台信息写入到存储账号中"
description: "如何将 Web 作业中控制台信息写入到存储账号中"
author: hahaxj
resourceTags: 'App Service Web, WebJobs, Console, Storage'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 3/31/2018
wacn.date: 3/31/2018
---

# 如何将 Web 作业中控制台信息写入到存储账号中

## 问题描述

在 Web 作业中通过 `Console.WriteLine` 打印日志，如何将打印的信息写入到存储账号中。

## 解决办法

1. 创建 .net core 控制台程序，添加 Storage package:

    ![01](media/aog-app-service-web-howto-write-webjobs-console-message-into-storage/01.png)

2. 关联存储账号如下，Publish 发布到本地：

    ![02](media/aog-app-service-web-howto-write-webjobs-console-message-into-storage/02.png)

3. 创建 cmd 文件，内容是 `dotnet.exe xxxxx.dll` 并压缩上传到 Web 作业中。

4. 对应的存储账号就会保存 `Console.WriteLine` 的输出内容，而不会保存到网站 FTP 中。

    ![03](media/aog-app-service-web-howto-write-webjobs-console-message-into-storage/03.png)
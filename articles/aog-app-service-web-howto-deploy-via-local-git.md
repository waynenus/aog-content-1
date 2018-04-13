---
title: "如何使用 Git 部署 Java Web 应用"
description: "如何使用 Git 部署 Java Web 应用"
author: zhangyannan-yuki
resourceTags: 'App Service Web, Java, Git'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 3/31/2018
wacn.date: 3/31/2018
---

# 如何使用 Git 部署 Java Web 应用

由于 Azure Web 应用默认的根目录为 `site\wwwroot`，通过 Git 进行部署的路径也是 `wwwroot`。<br>
对于 Java 环境而言，网站的根目录为 `site\wwwroot\webapps\ROOT`，这时我们可以使用 `DEPLOYMENT_TARGET` 环境变量来指定 Java 的部署路径。

在网站的应用程序设置页面，App Settings 应用设置位置进行设置，如下所示：

![01](./media/aog-app-service-web-howto-deploy-via-local-git/01.png)
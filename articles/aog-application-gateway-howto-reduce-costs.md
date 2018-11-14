---
title: "如何在不删除应用程序网关的情况下减少其产生的费用"
description: "如何在不删除应用程序网关的情况下减少其产生的费用"
author: Bingyu83
resourceTags: 'Application Gateway, Billing'
ms.service: application-gateway
wacn.topic: aog
ms.topic: article
ms.author: biyu
ms.date: 11/9/2018
wacn.date: 11/9/2018
---

# 如何在不删除应用程序网关的情况下减少其产生的费用

在部署并应用了应用程序网关后，平台会针对其进行收费，有关收费标准，参考[应用程序网关收费标准](https://www.azure.cn/zh-cn/pricing/details/application-gateway/)。

出于各种需求，我们是否可以在不删除应用程序网关及其上所有配置的同时，但短期内又不使用应用程序网关的情况下，减少费用。

可以通过如下 PowerShell 命令停止应用程序网关：

登录 Azure 账号，选择指定订阅；

```powershell
$AppGw = Get-AzureRmApplicationGateway -Name APPGWname -ResourceGroupName bingresourcegroup
Stop-AzureRmApplicationGateway -ApplicationGateway $AppGw
```

此命令允许成功后，可以通过如下日志再次确认是否已经停止:

![log](media/aog-application-gateway-howto-reduce-costs/log.png "log")

之后，如果需要恢复使用此应用程序网关，可以通过如下命令开启此应用程序网关：

```powershell
$AppGw = Get-AzureRmApplicationGateway -Name APPGWname -ResourceGroupName bingresourcegroup
Start-AzureRmApplicationGateway -ApplicationGateway $AppGw
```
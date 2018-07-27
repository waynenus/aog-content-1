---
title: "存放在 Azure File 中的.Net 网站代码运行时出现权限问题"
description: "存放在 Azure File 中的.Net 网站代码运行时出现权限问题"
author: lickkylee
resourceTags: 'Storage, File, ASP.NET'
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: lqi
ms.date: 07/13/2018
wacn.date: 07/13/2018
---

# 存放在 Azure File 中的 .Net 网站代码运行时出现权限问题

## 问题描述

Azure Windows 中的 IIS 站点的 .Net 代码放在 Azure File Share 中，通过网络浏览器打开站点时会报错，错误代码 `0x80131417`（权限设置问题）。

## 问题分析

ASP .Net 的应用程序代码，使用的是 .Net Framework 2 版本框架，默认该版本的 .Net Framework 不信任存放在网络共享中的代码。

## 解决办法

要让 Azure File 中的应用程序被 .Net Framework 信任，需要手动执行命令。

用管理员身份打开 CDM，切换到出问题版本的 .Net Framework 目录下：`> C:\Windows\Microsoft.NET\Framework64\<version>\`，执行命令：

`> caspol -m -ag 1 -url file://\\<yourstorage>.file.core.windows.net\<yourshare>\*  FullTrust`

> [!WARNING]
> 该命令会更改对应目录中 CONFIG 文件夹下的 **security.config** 和 **security.config.cch** 文件，建议提前进行备份。

若使用了不同版本的.Net Framework，需要切换到不同目录分别执行命令。但在 .Net Framework 3.5 SP1 之后的版本，默认就已经允许应用程序运行在网络文件夹中，无需再设置。

## 更多参考

- [IIS 使用 Azure File 的基本配置步骤及问题排查](https://blogs.iis.net/davidso/azurefile)
- [Caspol 设置网络共享信任](https://blogs.msdn.microsoft.com/shawnfa/2004/12/30/using-caspol-to-fully-trust-a-share/)
- [caspol 命令的更多介绍](https://docs.microsoft.com/en-us/dotnet/framework/tools/caspol-exe-code-access-security-policy-tool)
- [.Net Framework 3.5 SP1 关于网络共享信任的更新](https://blogs.msdn.microsoft.com/vancem/2008/08/13/net-framework-3-5-sp1-allows-managed-code-to-be-launched-from-a-network-share/)
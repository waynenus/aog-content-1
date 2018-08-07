---
title: "如何解决 Azure Functions unable to load one or more of the requested types 错误"
description: "如何解决 Azure Functions 无法加载一个或多个请求类型的错误"
author: Dillion132
resourceTags: 'Azure Functions'
ms.service: azure-functions
wacn.topic: aog
ms.topic: article
ms.author: v-zhilv
ms.date: 6/27/2018
wacn.date: 6/27/2018
---

# 如何解决 Azure Functions unable to load one or more of the requested types 错误

## 问题描述

用户在 Azure Functions V2 (.net core) 应用中添加 Azure Active Directtory 引用并获取 Azure 密钥保管库中的机密，在本地 Visual Studio 2017 中运行应用时遇到 “**Unable to load one or more of the requested types**” 错误，详细信息如下：

```
The following 1 functions are in error:
Function1: Unable to load one or more of the requested types. Retrieve the LoaderExceptions property for more information.

```

![error.PNG](./media/aog-azure-functions-unable-to-load-one-or-more-of-the-requested-types/error.PNG)

本地测试环境信息如下： Visual Studio 2017 15.6.4 版本， Azure Function v2 版本， Azure Function SDK 版本 1.0.6，Microsoft.IdentityModel.Clients.ActiveDirectory 3.19.8 版本，Microsoft.Azure.KeyVault 3.0.0 版本，Azure Functions and Web Jobs Tools 15.0.40215.0 版本。

## 问题分析

该错误是由于 Microsoft.IdentityModel.Clients.ActiveDirectory 与 Azure Functions Runtime 的本地版本不兼容造成的。在上述测试环境下运行函数应用时，发现 Azure Function Runtime 版本为 2.0.11353 版本 (即 Azure Functions core tools 2.0.1-beta 版本)。

![runtime-version.PNG](./media/aog-azure-functions-unable-to-load-one-or-more-of-the-requested-types/runtime-version.PNG)

而该错误已经在 [Azure Functions Runtime 2.0.11857 版本](https://github.com/Azure/azure-functions-host/releases/tag/v2.0.11857-alpha) (即 Azure Functions core tools 2.0.1-beta.29 版本) 中被修复了，因此用户可以尝试使用以下方式解决该问题。

- [更新本地 Azure Function core tools 版本](#update-tool-version)。
- [更新 Microsoft.IdentityModel.Clients.ActiveDirectory 包版本](#update-ad-version)。
- [将 Microsoft.IdentityModel.Clients.ActiveDirectory 包替换为 Microsoft.Azure.Services.AppAuthentication 包](#replace)。

## 解决方法

### <a id="update-tool-version"> </a>更新本地 Azure Function core tools 版本

由于 **Azure Function core tools 2.0.1-beta.29 版本目前尚未提供给 Visual Studio 用户**, 用户可以手动下载该版本，并在应用程序中修改配置，然后使用该工具运行函数应用。

1. 下载用于 Windows 的 [Azure Functions core tools 2.0.1-beta.29 版本](https://github.com/Azure/azure-functions-core-tools/releases)。

2. 在 Visual Studio 中设置函数应用调试配置，使用指定工具运行函数应用。在应用程序属性面板中，打开调试选项卡，修改如下配置：

    * 启动：“可执行文件“
    * 可执行文件：< dotnet.exe 文件路径 >
    * 应用程序参数：< Azure Functions core tools 2.0.1-beta.29 版本工具解压包路径>\Azure.Functions.Cli.win-x64\func.dll start
    * 工作目录：$(TargetDir)

    ![setting.PNG](./media/aog-azure-functions-unable-to-load-one-or-more-of-the-requested-types/setting.PNG)

3. 保存设置后，重新编译 Azure Function 应用并运行，可以看到该问题已经解决。

    ![success.PNG](./media/aog-azure-functions-unable-to-load-one-or-more-of-the-requested-types/success.PNG)

### <a id="update-ad-version"></a>更新 Microsoft.IdentityModel.Clients.ActiveDirectory 包版本

右键单击应用项目，通过 NuGet 包管理器将 Microsoft.IdentityModel.Clients.ActiveDirectory 包更新为 3.14.X 版本，并将 Microsoft.Azure.KeyVault 包更新为 2.3.2 版本。

> [!Note] 
> 
> 使用该方式可能会遇到 Microsoft.IdentityModel.Clients.ActiveDirectory 包与 Microsoft.Azure.KeyVault 3.0.0 版本不兼容，无法加载 Microsoft.Azure.KeyVault 包引用的问题，可以通过 NuGet 包管理器将 Microsoft.Azure.KeyVault 更新为 2.3.2 版本。

### <a id="replace"></a>将 Microsoft.IdentityModel.Clients.ActiveDirectory 包替换为 Microsoft.Azure.Services.AppAuthentication 包

右键单击应用项目，通过 NuGet 包管理器卸载 Microsoft.IdentityModel.Clients.ActiveDirectory 包，然后安装Microsoft.Azure.Services.AppAuthentication 包 v1.0.3 版本。

> [!Note] 
> 
> 使用该方式可能会遇到 Microsoft.Azure.Services.AppAuthentication 包与 Microsoft.Azure.KeyVault 3.0.0 版本不兼容，无法加载 Microsoft.Azure.KeyVault 包引用的问题，可以通过 NuGet 包管理器将 Microsoft.Azure.KeyVault 更新为 2.3.2 版本。
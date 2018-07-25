---
title: "如何解决 Azure Functions ScriptHost initialization failed 错误"
description: "如何解决 Azure Functions ScriptHost initialization failed 错误"
author: Dillion132
resourceTags: 'Azure Functions'
ms.service: azure-functions
wacn.topic: aog
ms.topic: article
ms.author: v-zhilv
ms.date: 7/2/2018
wacn.date: 7/2/2018
---

# 如何解决 Azure Functions ScriptHost initialization failed 错误

## 问题描述

用户在 Visual Studio 2017 中创建了一个 Azure Functions V2 (.net core) 应用，该应用可以在本地运行调试，但是在应用依赖项中发现以下警告：

![warning.PNG](./media/aog-azure-functions-scripthost-initialization-failed-error/warning.PNG)

尝试将 Azure Function SDK 版本从 1.0.6 版本更新到最新版 1.0.14 版本后，警告消失。

![check-sdk.PNG](./media/aog-azure-functions-scripthost-initialization-failed-error/check-sdk.PNG)

但是函数应用却无法运行，并报 “**ScriptHost initialization failed**” 错误，详细错误信息如下：

```
[7/2/2018 5:30:26 AM] ScriptHost initialization failed
[7/2/2018 5:30:26 AM] System.Private.CoreLib: Could not load file or assembly 'Microsoft.AspNetCore.Mvc.Abstractions, Version=2.0.2.0, Culture=neutral, PublicKeyToken=adb9793829ddae60'. Could not find or load a specific file. (Exception from HRESULT: 0x80131621). System.Private.CoreLib: Could not load file or assembly 'Microsoft.AspNetCore.Mvc.Abstractions, Version=2.0.2.0, Culture=neutral, PublicKeyToken=adb9793829ddae60'.
Listening on http://localhost:7071/
Hit CTRL-C to exit...
Object reference not set to an instance of an object.
```

![error.PNG](./media/aog-azure-functions-scripthost-initialization-failed-error/error.PNG)

本地测试环境信息如下： Visual Studio 2017 15.6.4 版本， Azure Function v2 版本， Azure Function SDK 版本 1.0.6，Azure Functions and Web Jobs Tools 15.0.40215.0 版本。

## 问题分析

该错误主要与 Azure Functions Runtime 的本地版本有关。在上述本地测试环境下运行函数应用时，发现 Azure Function Runtime 版本为 2.0.11353 版本 (即： Azure Functions core tools 2.0.1-beta 版本)。

![runtime-version.PNG](./media/aog-azure-functions-scripthost-initialization-failed-error/runtime-version.PNG)

而该错误已经在 [Azure Functions Runtime 2.0.11857 版本](https://github.com/Azure/azure-functions-host/releases/tag/v2.0.11857-alpha) (即： Azure Functions core tools 2.0.1-beta.29 版本) 中被修复了，因此用户可以在本地更新 Azure Function core tools 版本，使用 2.0.1-beta.29 版本运行函数应用。

由于 **Azure Function core tools 2.0.1-beta.29 版本目前尚未提供给 Visual Studio 用户** , 用户可以手动下载该版本，并在应用程序中修改配置，然后使用该工具运行函数应用。

## 解决方法

1. 下载用于 Windows 的 [Azure Functions core tools 2.0.1-beta.29 版本](https://github.com/Azure/azure-functions-core-tools/releases)。

2. 在 Visual Studio 中设置函数应用调试配置，使用指定工具运行函数应用。在应用程序属性面板中，打开调试选项卡，修改如下配置：

    * 启动：“可执行文件“
    * 可执行文件：< dotnet.exe 文件路径 >
    * 应用程序参数：< Azure Functions core tools 2.0.1-beta.29 版本工具解压包路径>\Azure.Functions.Cli.win-x64\func.dll start
    * 工作目录：$(TargetDir)

    ![debug-config.PNG](./media/aog-azure-functions-scripthost-initialization-failed-error/debug-config.PNG)

3. 保存设置后，重新编译 Azure Function 应用并运行，可以看到该问题已经解决。

    ![success.PNG](./media/aog-azure-functions-scripthost-initialization-failed-error/success.PNG)
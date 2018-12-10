---
title: "如何配置 Azure Active Directory 账户密码永不过期（AAD PowerShell 2.0 版本）"
description: "如何配置 Azure Active Directory 账户密码永不过期（AAD PowerShell 2.0 版本）"
author: jessie-pang
resourceTags: 'Azure Active Directory, AAD PowerShell 2.0'
ms.service: active-directory
wacn.topic: aog
ms.topic: article
ms.author: v-ciwu
ms.date: 12/5/2018
wacn.date: 12/5/2018
---

# 如何配置 Azure Active Directory 账户密码永不过期（AAD PowerShell 2.0 版本）

之前已有文章讲述了如何使用 MSOnline PowerShell 1.0 的命令集配置 Azure Active Directory 账户密码永不过期：[如何配置 Azure Active Directory 账户密码永不过期](https://docs.azure.cn/articles/azure-operations-guide/active-directory/aog-active-directory-account-never-expire)。

本文将描述如何使用 Azure Active Directory PowerShell 2.0 的命令集完成这一配置。请注意，通常不建议将用户帐户的密码设置为永不过期，但可以将服务帐户（如用于 Active Directory 同步的帐户）的密码设置为永不过期。本文仅适用于云用户帐户，对于将本地活动目录与 Azure AD 集成的账号，请在本地活动目录内对账号进行配置，并使用同步工具将配置同步至 Azure AD。

1. 安装 [AAD PowerShell 模块](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0)

    ```powershell
    Install-Module AzureAD
    ```

2. 连接至 Azure AD

    ```powershell
    Connect-AzureAD -AzureEnvironmentName AzureChinaCloud
    ```

    在弹出的窗口内输入登录的用户名和密码。

3. 查看用户当前的密码策略

    ```powershell
    (Get-AzureADUser -SearchString "Test").PasswordPolicies
    ```

    以上命令搜索用户名中包含关键字 “Test” 的用户，并显示其密码策略。如果输出为空，说明该用户未设置过密码策略，默认密码过期期限为 90 天。

    可以使用以下命令查看该用户的全部属性信息：

    ```powershell
    Get-AzureADUser -SearchString "Test" | Format-List
    ```

4. 设置密码策略为永不过期

    可以根据关键词搜索相关用户并设置密码策略：

    ```powershell
    Get-AzureADUser -SearchString "Test" | Set-AzureADUser -PasswordPolicies DisablePasswordExpiration
    ```

    或者使用用户的 ObjectId 精确定位到用户：

    ```powershell
    Set-AzureADUser -ObjectId 7f9e31dc-83db-428e-855f-42700b55a367 -PasswordPolicies DisablePasswordExpiration
    ```

    若要将所有用户设置为密码永不过期，请运行以下命令：

    ```powershell
    Get-AzureADUser -All $true | Set-AzureADUser -PasswordPolicies DisablePasswordExpiration
    ```

    以下命令可以查看已经设置过密码永不过期的用户列表：

    ```powershell
    Get-AzureADUser | ? {$_.PasswordPolicies -match "DisablePasswordExpiration"}
    ```
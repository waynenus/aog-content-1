---
title: "通过 Azure Active Directory 认证 Azure 自动化服务"
description: "如何在 Azure 自动化服务中使用 Azure AD 认证"
author: JamborYao
resourceTags: ''
ms.service: automation
wacn.topic: aog
ms.topic: article
ms.author: Jianbo.Yao
ms.date: 01/19/2018
wacn.date: 02/01/2016
---

# 通过 Azure Active Directory 认证 Azure 自动化服务

关于 Azure 自动化的详细概念请阅读[这篇文章](https://www.azure.cn/home/features/automation/)。

Azure 自动化是通过 Windows PowserShell 工作流（也称为 Runbook）来处理 Azure 的资源和第三方应用的创建、部署、监视和维护工作的。在执行 Runbook 的时候自然需要认证是否拥有合理的身份来执行操作。

本文介绍如何通过 Azure Active Directory 来授权。

首先去确认 Azure 订阅的目录名字（本文使用的订阅账户是" 1RMB Trial Offer "），可以在 [Azure 门户](https://portal.azure.cn/) **订阅** 服务里找到，如下图所示：

![](./media/aog-automation-connect-mooncake/check_subscription.PNG)

之后去 Azure Active Directory 里找到对应的目录，或者点击切换目录切换到指定目录，如下图所示：

![](./media/aog-automation-connect-mooncake/change-directory.PNG)

依次点击 “**用户和组**” -> “**所有用户**” -> “**新建用户**”：

![](./media/aog-automation-connect-mooncake/add-new-user.png)

输入要创建的用户名，在 “**配置文件**” 中输入名、姓等内容：

![](./media/aog-automation-connect-mooncake/adduser.PNG)

勾选 “**显示密码**”，并记录下初始密码，如果创建时未勾选，则只能通过重置密码的方式得到登录密码。

然后进入 “**订阅**”，在 “**访问控制(标识和访问管理)**” 中为新添加的用户分配权限：

![](./media/aog-automation-connect-mooncake/addpermission.PNG)

点击添加，设置用户角色为订阅所有者，选中之前添加的新用户：

![](./media/aog-automation-connect-mooncake/addpermission1.PNG)

将新用户设置为协同管理员：

![](./media/aog-automation-connect-mooncake/addpermission2.PNG)

使用新创建的账户在 [Azure 门户](https://portal.azure.cn)中登录，第一次登录时需要修改密码。

> [!NOTE]
> 如果使用初始密码作为 Runbook 的凭证可能会导致操作失败。

然后进入自动化账户，在共享的资源区域中点击 “**凭据**”，然后点击 “**添加凭据**”：

![](./media/aog-automation-connect-mooncake/addcredential.PNG)

输入凭据名称以及刚才创建的用户名和密码：

![](./media/aog-automation-connect-mooncake/newcredential.PNG)

至此 PowerShell 的凭据已经创建完成，下面讨论如何来使用（本文主要是使用自动化服务实现开关虚拟机）。

首先我们进入 **Runbook**，点击 “**添加 Runbook**”：

![](./media/aog-automation-connect-mooncake/entry-runbook.PNG)

选择 “**创建一个新的 Runbook**”:

![](./media/aog-automation-connect-mooncake/addrunbook.PNG)

默认情况下，创建完 Runbook 之后会直接进入编辑草稿界面，如果未进入可以选中 Runbook，然后点击 “**编辑**” 按钮，进入编辑界面：

![](./media/aog-automation-connect-mooncake/edit-runbook-1.png)

在编辑的过程中使用下面的指令来实现授权，`Powercredential` 为在定义凭据时使用的名称。

```PowerShell
$Cred = Get-AutomationPSCredential -Name "Powercredential"; 
Add-AzureRmAccount -Credential $Cred -EnvironmentName AzureChinaCloud;
Select-AzureRmSubscription -SubscriptionName "<订阅名称>";
Start-AzureRmVM -ResourceGroupName "dillion_rg" -Name "managedvm"
```

![](./media/aog-automation-connect-mooncake/edit-runbook.PNG)
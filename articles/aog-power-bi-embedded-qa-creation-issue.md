---
title: "如何解决无法创建 Power BI Embedded 的问题"
description: "如何解决无法创建 Power BI Embedded 的问题"
author: v-DavidTang
resourceTags: 'Power BI Pro'
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 07/19/2018
wacn.date: 07/19/2018
---

# 如何解决无法创建 Power BI Embedded 的问题

## 问题描述

用户在 [Azure 门户](https://portal.azure.cn)上创建 Power BI Embedded 的时候出现 “遇到身份验证问题”，同时当前页面一直处于加载状态下，无法进行下一步操作。

![add-01](media/aog-power-bi-embedded-qa-creation-issue/add-01.png)

## 问题分析

在使用 Azure Power BI Embedded 之前，您首先需要申请 Power BI Pro 的许可证，然后才可以在 [Azure 门户](https://portal.azure.cn)中使用 Power BI Embedded 服务。

## 解决方法

### 前提条件

1. 如果.您已经有了 Power BI 的许可证，可以直接登录 [Power BI 的管理界面](https://app.powerbi.cn) 使用 Power BI，可以跳过以下步骤直接参见[步骤 8](#step8)。
2. 如果.您暂时还没有 Power BI 的许可证，请确保.您拥有 Office 365 全局管理员或账单管理员的权限。具体操作步骤参见下文。

### 操作步骤

1. 使用您的账号登录 [Office 365 Admin center](https://portal.partner.microsoftonline.cn/AdminPortal/Home#/homepage)。

2. 在左侧 **账单** 栏下找到 **购买服务** 选项。在右侧的服务中找到 **Power BI Pro**。点击 **开始免费试用**。

    ![01](media/aog-power-bi-embedded-qa-creation-issue/01.png)

3. 购买成功后，**账单** 栏下找到 **许可证**，可以在右边看见购买的 Power BI Pro 的许可证，表示此时已经成功申请了试用。

    ![02](media/aog-power-bi-embedded-qa-creation-issue/02.png)

4. 回到左侧 **账单** 栏，点击 **订阅**。可以看到全局管理员自动获得了一个许可证。剩余可用 24 个。

    ![03](media/aog-power-bi-embedded-qa-creation-issue/03.png)

5. 点击许可证下方的 **分配给用户**，进入活动用户页面，单击需要授权的用户。

    ![04](media/aog-power-bi-embedded-qa-creation-issue/04.png)

6. 在产品许可证栏点击 **编辑**。

    ![05](media/aog-power-bi-embedded-qa-creation-issue/05.png)

7. 将 Power BI Pro 的开关打开，并点击 **保存**。

    ![06](media/aog-power-bi-embedded-qa-creation-issue/06.png)

    保存完毕后可以看到新用户已经授权了 Power BI Pro 许可证。

    ![07](media/aog-power-bi-embedded-qa-creation-issue/07.png)

8. <a id="step8"></a>此时登录 [Power BI 的管理界面](https://app.powerbi.cn)，就可以使用了 Power BI 服务了。

    ![08](media/aog-power-bi-embedded-qa-creation-issue/08.png)

9. 接下来登录 [Azure 门户](https://portal.azure.cn)，此时您可以顺利的创建 Power BI Embedded。

    ![add-02](media/aog-power-bi-embedded-qa-creation-issue/add-02.png)

10. 创建成功后尽情使用 Power BI Embedded。

    ![add-03](media/aog-power-bi-embedded-qa-creation-issue/add-03.png)
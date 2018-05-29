---
title: "Azure 门户或 Azure 帐户中心“找不到任何订阅”提示"
description: "针对登录 Azure 门户或账户中心时看到“看来您还未创建任何订阅”信息的解决方案。"
author: V-duji
resourceTags: '订阅'
ms.service: billing
wacn.topic: aog
ms.topic: troubleshooting
ms.author: v-duji
ms.date: 01/11/2018
wacn.date: 12/27/2017
---

# Azure 门户或 Azure 帐户中心“找不到任何订阅”提示

当您尝试登录 [Azure 门户](https://portal.azure.cn/) 或 [Azure 账户中心](https://account.windowsazure.cn/)时，可能会出现 “**找不到任何订阅**” 错误消息。<br>
本文提供针对此问题的解决方案。

## **症状一**

### **现象描述**

当您登录 [Azure 门户](https://portal.azure.cn/) 或 [Azure 账户中心](https://account.windowsazure.cn/) 看到 “**未找到任何订阅**” 或 “**看来您还未创建任何订阅**” 的提示信息。

<img src="media/aog-billing-no-subscription-found/classic.png" alt="classic" height="255" width="400" style="display: inline-block">&nbsp;&nbsp;<img src="media/aog-billing-no-subscription-found/portal.png" alt="portal" height="255" width="400" style="display: inline-block">


### **原因分析**

如果您的 Azure 账户尚未成功创建订阅，则会出现此问题。

### **解决方法**

- 线上购买用户

    1. 保持您登录的状态，在同一个页面，返回 [Azure 中国官网](http://www.azure.cn/)。
    2. 根据您需要的订阅，在页面右上角点击相对应的注册页面。1 元试用订阅请点击 “**申请试用**”，标准预付费订阅请点击 “**我要购买**”。请留意每位新用户仅能申请一次 1 元试用订阅，更多细节规定请参考 [1 元试用订阅详情](https://www.azure.cn/offers/ms-mc-azr-44p/)。
    3. 进行订阅注册操作流程。
    4. 在您完成费用支付后，订阅即创建成功，您可开始进行 Azure 服务。

- 企业协议用户

    1. 请用您合同中的企业账户登录[企业门户网站](http://www.ea.azure.cn/)。
    2. 根据您的需要在企业门户网站添加账户管理员，在账户管理员成功添加后，选择添加订阅。您将被指引到我们的 [Azure 账户中心](https://account.windowsazure.cn/)完成第一个订阅的添加流程。

## **症状二**

### **现象描述**

当您登录 [Azure 门户](https://portal.azure.cn/) 或 [Azure 账户中心](https://account.windowsazure.cn/)，看到 “**切换目录**” 的提示信息。

![01](media\aog-billing-no-subscription-found\01.png)

### **原因分析**

如果选择了错误的目录，或者帐户没有足够的权限，则会出现此问题。 

### **解决方法**

- **[Azure 门户](https://portal.azure.cn/)选择错误目录**

    1. 通过单击右上角的帐户确保已选择正确的 Azure 目录。

        ![02](media\aog-billing-no-subscription-found\02.png)

    2. 如果已选择正确的 Azure 目录，但仍收到错误消息，请确认[将帐户添加为对应订阅的所有者](https://docs.azure.cn/zh-cn/billing/billing-add-change-azure-subscription-administrator)。

- **[Azure 门户](https://portal.azure.cn/)账户没有足够权限**

    请检查使用的帐户是否是帐户管理员。<br>
    要验证谁是帐户管理员，请执行下列步骤：

    1.	登录到 [Azure 门户中的订阅视图](https://portal.azure.cn/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)。

        ![03](media\aog-billing-no-subscription-found\03.png)

    2.	选择要检查的订阅，并关注 “**设置**” 下的信息。

    3.	选择 “**属性**”。 订阅的帐户管理员会显示在 “**帐户管理员**” 框中。

        ![04](media\aog-billing-no-subscription-found\04.png)

## **了解更多**

[Azure 门户使用概览](https://school.azure.cn/courses/48)

[注册标准预付费订阅](https://www.azure.cn/pricing/billing/azure-pia-application-and-signup/)

## **问题未解决？联系支持人员**

如果仍需帮助，请[联系支持人员](https://www.azure.cn/support/contact/)以快速解决问题。
---
title: "Azure 线上注册的常见问题"
description: "针对注册 Azure 的过程中常遇到的问题提供了解决方案。"
author: V-duji
resourceTags: '注册，没有订阅'
ms.service: billing
wacn.topic: aog
ms.topic: article
ms.author: v-duji
ms.date: 01/11/2018
wacn.date: 12/27/2017
---

# Azure 线上注册的常见问题

本文列出了用户在线上注册 Azure 服务时常见的操作问题及其解决方案。如果您所遇到的问题仍无法被解决，请[联系支持人员](https://www.azure.cn/support/contact/)获取协助。

1. **回填短信代码后无法点击 “继续”，如下图：**

    ![01](media/aog-billing-sign-in-issues-faq/01.png)

    **解决方案：** 在您填入验证码之后，您需要先点击 “**验证代码**” 进行验证。验证成功后，才能点击 “**继续**”。

2. **注册过程中，进度条挂起，长时间持续，如下图：**

    ![02](media/aog-billing-sign-in-issues-faq/02.png)

    **解决方案：** 您的浏览器必须允许第三方 cookie。您可参考以下方法修改您的浏览器 cookie 设置。不同版本的浏览器，设置的方法可能不同，请参考浏览器的帮助内容。

    - Chrome：设置 > 显示高级设置 > 隐私 > 内容设置 > 清除 “禁止第三方 cookie 和站点数据”。
    - Edge：设置 > 查看高级设置 > Cookie > 不要阻止 cookie。
    - Safari：设置 > Safari > 阻止 Cookie > 总是允许。

    修改完成后，请刷新当前的 Azure 注册页，检查问题是否已解决。如果刷新没有解决此问题，请退出并重新启动您的浏览器，然后再试一次。

3. **注册页面出现 “抱歉！1 元人民币的试用订阅/标准预付费服务不可用。您没有资格获得此特定 Promo 产品/服务” 的提示信息。**

    **解决方案：** 若您之前申请过 Azure 试用，但没有完成注册流程。这个提示信息表示您未能在有效期内使用注册激活码，您可点击以下链接重新申请。

    [Azure 试用申请表](https://www.azure.cn/pricing/1rmb-trial-full/?form-type=identityauth)

    [Azure 预付费产品购买申请表](https://www.azure.cn/pricing/pia-waiting-list/?form-type=identityauth)

4. **当您登录 Azure 门户或 Azure 账户中心看到 “看来您还未创建任何订阅”的提示信息。**

    **解决方案：** 这表示您的 Azure 账户尚未创建订阅成功，请参考[登录后未创建任何订阅](aog-billing-no-subscription-found.md)完成 Azure 账户创建。

## **了解更多**

[注册 1 元试用订阅](https://www.azure.cn/pricing/billing/azure-1rmb-trial-application-and-signup/)

[注册标准预付费订阅](https://www.azure.cn/pricing/billing/azure-pia-application-and-signup/)

## **问题未解决？联系支持人员**

如果仍需帮助，请[联系支持人员](https://www.azure.cn/support/contact/)以快速解决问题。
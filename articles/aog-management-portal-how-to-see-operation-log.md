---
title: "如何查看云服务执行的相关操作记录"
description: "本文介绍如何查看云服务执行的相关操作记录"
author: hello-azure
resourceTags: ''
ms.service: multiple
wacn.topic: aog
ms.topic: article
ms.author: Jianbo.Yao
ms.date: 01/19/2018
wacn.date: 12/18/2015
---

# 如何查看云服务执行的相关操作记录

**本文包含以下内容**
- [操作日志的作用](#function)
- [如何访问日志](#access)
- [详细步骤](#detail)

## <a id="function"></a>操作日志的作用

1. 可以通过操作日志的详细情况知道我们的操作失败的具体原因。
2. 通过操作日志可以在失败的详情中找出操作 ID，在联系技术支持的过程中可以对应到具体的操作。

## <a id="access"></a>如何访问日志

我们可以通过云服务左侧菜单 “**操作日志（经典）**” 查看日志信息。

## <a id="detail"></a>详细步骤

云服务提供了重启日志以及特定经典资源的操作的日志，请通过以下步骤查看：

1. 登录 [Azure 门户](https://portal.azure.cn/)网站，选择 “**云服务**”：

    ![](./media/aog-management-portal-how-to-see-operation-log/choose_cloudservice.PNG)

2. 选择要查看的 **云服务** -> 点击 “**操作日志（经典）**”，可以选择时间跨度，查看相关的操作日志和启动日志：

    ![](./media/aog-management-portal-how-to-see-operation-log/operationlog.PNG)

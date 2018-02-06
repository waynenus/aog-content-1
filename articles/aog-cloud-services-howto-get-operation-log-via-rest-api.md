---
title: "如何使用 REST API 获取云服务操作日志"
description: "如何使用 REST API 获取云服务操作日志"
author: hello-azure
resourceTags: 'Cloud Services, REST API'
ms.service: cloud-services
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 02/05/2018
wacn.date: 01/31/2018
---

# 如何使用 REST API 获取云服务操作日志

## 问题描述

在某些情况下，我们需要监视服务的操作状态，比如云服务（PaaS）停止、重启等行为，目前服务 SDK 并没有提供此类的 API，那么是否可以达成这个需求呢？

## 问题分析

Azure 门户提供了一个操作日志（Operation Log）的功能，可以用于查询云服务的操作日志，详情可以参考[如何查看云服务执行的相关操作记录](/articles/azure-operations-guide/others/aog-management-portal-how-to-see-operation-log)。<br>
同时，Azure 也提供了相应的 REST API，请参考：[List Subscription Operations](https://msdn.microsoft.com/en-us/library/azure/gg715318.aspx)。

## 解决方法

1. 构建请求

    - 请求 URI：

        > [!NOTE]
        > 文档中给出的是国际版 Azure 的终结点地址，使用中国区 Azure 需要将 `management.windows.net` 修改为 `management.core.chinacloudapi.cn`。
        
        ![01](media/aog-cloud-services-howto-get-operation-log-via-rest-api/01.png)

    - URI 参数：

        上述请求并没有做限制（Filter），是获取订阅下所有的操作日志，为了达成需求，需要使用以下 3 个参数来做限定：

        - `StartTime=<start-of-timeframe>`
        - `EndTime=<end-of-timeframe>`
        - `ObjectIdFilter=<object-url>`

	参数详解请参考[文档说明](https://msdn.microsoft.com/en-us/library/azure/gg715318.aspx)，此外您还需要指定`api-version`，构建的请求格式为:

    `https://management.core.chinacloudapi.cn/<subscription-id>/operations?ObjectIdFilter<object-url>&StartTime=<start-of-timeframe>&EndTime=<end-of-timeframe>`

      具体参数字段需要替换为您实际项目中的值，针对cloud Service(Paas)对应的ObjectIdFilter的格式为：
      ObjectIdFilter=/<subscription-id>/services/hostedservices/<cloud-service-name>.
      因此获取某个特定的cloud Service(Paas)的示例请求如下：

    `https://management.core.chinacloudapi.cn/5bbf0cbb-647d-****-****-26629f109bd7/operations?ObjectIdFilter=/5bbf0cbb-647d-****-****-26629f109bd7/services/hostedservices/kevin1&StartTime=2018-01-01&EndTime=2018-01-31&api-version=2014-01`

2. 请求参数

    - `x-ms-version`：2012-03-01 （或更高版本）
    - `Authorization`：调用以下 REST API 来获取
        - `RESTAPI: https://login.chinacloudapi.cn/common/oauth2/token?api-version=1.0` 
        - `Method: POST`
        - `HEADER: Content-Type: application/x-www-form-urlencoded`
        - `POST DATA`: 
            - `grant_type: password  # 固定值`
            - `resource: https://management.core.chinacloudapi.cn/  # 固定值`
            - `username: 订阅登录账户`
            - `password: 订阅登录密码`
            - `client_id:  1950a258-227b-4e31-a9cf-717495945fc2  # 固定值`

    如下示例：`access_token` 的值即 `Authorization` 值：
    
    ![02](media/aog-cloud-services-howto-get-operation-log-via-rest-api/02.png)

3. 调用请求

    ![03](media/aog-cloud-services-howto-get-operation-log-via-rest-api/03.png)

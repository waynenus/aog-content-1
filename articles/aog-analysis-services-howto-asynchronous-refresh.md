---
title: '如何对中国区 Azure 分析服务模型进行异步刷新'
description: '如何对中国区 Azure 分析服务模型进行异步刷新'
author: kustbilla
resourceTags: 'Analysis Services, REST API'
ms.service: analysis-services
wacn.topic: aog
ms.topic: article
ms.author: weihuan
ms.date: 08/30/2018
wacn.date: 08/30/2018
---

# 如何对中国区 Azure 分析服务模型进行异步刷新

## 问题描述

中国区 Azure 的官方文档中并没有如何对 Azure 分析服务(以下简称 AAS)模型进行刷新的介绍：

![01](media/aog-analysis-services-howto-asynchronous-refresh/01.png)

客户如若希望对 AAS 模型执行异步刷新，可参考以下文档并对相应的 URI 做一些修改：

- [使用 REST API 进行刷新](https://docs.microsoft.com/zh-cn/azure/analysis-services/analysis-services-async-refresh)

由于上述文档是针对国际版 Azure，若要适用于中国区 Azure 需参考以下文档修改 URI：

- [中国区 Azure 应用程序开发说明](https://docs.azure.cn/zh-cn/articles/guidance/developerdifferences)

## 操作步骤

客户可以参考以下示例执行模型的异步刷新：

1. 参照文档：[Azure Analysis Services - Adventure Works 教程](https://docs.azure.cn/zh-cn/analysis-services/tutorials/aas-adventure-works-tutorial) 内容，部署名为 “stanleysqldb1400”，数据源为 Azure SQL 数据库的 AAS 模型。Management Studio 上查得表 dbo.Products 的记录如下：

    ![02](media/aog-analysis-services-howto-asynchronous-refresh/02.png)

    在 Power BI 桌面版上用 connect live 方式连接 AAS 数据库后，查到的此表结果如下：

    ![03](media/aog-analysis-services-howto-asynchronous-refresh/03.png)

    ![04](media/aog-analysis-services-howto-asynchronous-refresh/04.png)

2. 参照文档：[使用 REST API 执行异步刷新](https://docs.microsoft.com/zh-cn/azure/analysis-services/analysis-services-async-refresh) 执行以下操作：

    1. 在 Azure 门户上注册 AAD 本机应用，重定向 URI 填入 `urn:ietf:wg:oauth:2.0:oob`。

        ![05](media/aog-analysis-services-howto-asynchronous-refresh/05.png)

    2. 记录应用程序 ID：

        ![06](media/aog-analysis-services-howto-asynchronous-refresh/06.png)

    3. 设置所有者和添加 Azure Analysis Services 权限：

        ![07](media/aog-analysis-services-howto-asynchronous-refresh/07.png)

        ![08](media/aog-analysis-services-howto-asynchronous-refresh/08.png)

3. 从 [Analysis Services](https://github.com/Microsoft/Analysis-Services) 处下载示例代码，打开 RestApiSample 并做以下更改：

    1. 将代码中所有的 `windows.net` 均改为 `chinacloudapi.cn`；

        ![09](media/aog-analysis-services-howto-asynchronous-refresh/09.png)

    2. 填入 base url，格式如下所示：

        ```
        https://chinanorth.asazure.chinacloudapi.cn/servers/<server_name>/models/s<model_name>/
        ```

        ![10](media/aog-analysis-services-howto-asynchronous-refresh/10.png)

    3. 更新之前记录的应用 ID：

        ![11](media/aog-analysis-services-howto-asynchronous-refresh/11.png)

    4. 点击 Tools > NuGet Package Manager > Package Manager Console 执行以下命令，然后再 build:

        ![12](media/aog-analysis-services-howto-asynchronous-refresh/12.png)

        然后对 dbo.Products 表删除记录，运行 RestApiSample 等待刷新成功，便可在 Power BI Desktop 上看到刷新后的结果了：

        ![13](media/aog-analysis-services-howto-asynchronous-refresh/13.png)

        ![14](media/aog-analysis-services-howto-asynchronous-refresh/14.png)

        ![15](media/aog-analysis-services-howto-asynchronous-refresh/15.png)

        ![16](media/aog-analysis-services-howto-asynchronous-refresh/16.png)

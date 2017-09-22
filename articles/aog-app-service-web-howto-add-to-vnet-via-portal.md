---
title: 如何通过 Azure 门户将 Web 应用添加到 Vnet 中
description: 如何通过 Azure 门户将 Web 应用添加到 Vnet 中
service: ''
resource: App Service Web
author: maysmiling
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'App Service Web, Vnet, Azure Portal'
cloudEnvironments: MoonCake

ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-amazha
ms.date: 09/22/2017
wacn.date: 09/22/2017
---
# 如何通过 Azure 门户将 Web 应用添加到 Vnet 中

## 前提/要求：

1. 网站必须为标准或者高级模式。
2. 添加虚拟网络目前只支持点但站点（point to site）模式。
3. 目标网站及目标虚拟网络在同一个订阅下。
4. Vnet 必须连接虚拟网络网关。

    ![01](media/aog-app-service-web-howto-add-to-vnet-via-portal/01.png)

## 操作步骤

1. 在网站的【**网络**】配置页配置即可，如果是非标准模式，请按照提示进行升级网站。

    ![02](media/aog-app-service-web-howto-add-to-vnet-via-portal/02.png)

2. 升级完成以后，点击【**安装**】。

    ![03](media/aog-app-service-web-howto-add-to-vnet-via-portal/03.png)

3. 在导向的虚拟网络配置页面选择要连接的虚拟网络或者新建虚拟网络。

    ![04](media/aog-app-service-web-howto-add-to-vnet-via-portal/04.png)

4. 如果网络信息显灰色，可以点击小符号进行查看无法选择的原因。

    ![05](media/aog-app-service-web-howto-add-to-vnet-via-portal/05.png)

5. 连接成功以后，会在【**网络**】配置模块看到连接的信息。

    ![06](media/aog-app-service-web-howto-add-to-vnet-via-portal/06.png)

6. 另外，也可以进入相应的 Vnet 中查看到相关连接信息。

    ![07](media/aog-app-service-web-howto-add-to-vnet-via-portal/07.png)
    ![08](media/aog-app-service-web-howto-add-to-vnet-via-portal/08.png)
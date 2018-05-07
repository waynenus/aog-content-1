---
title: "如何解决无法创建 IoT Hub 的问题"
description: "如何解决无法创建 IoT Hub 的问题"
author: Dillion132
resourceTags: 'IoT Hub'
ms.service: iot-hub
wacn.topic: aog
ms.topic: article
ms.author: v-zhilv
ms.date: 04/27/2018
wacn.date: 04/27/2018
---

# 如何解决无法创建 IoT Hub 的问题

## 问题描述

用户在通过 [Azure 门户](https://portal.azure.cn) 创建 IoT Hub 时遇到以下问题，导致无法创建 IoT Hub。

![errormessage.PNG](./media/aog-iot-hub-can-not-create-iot-hub/errormessage.PNG)

## 问题分析

在 Azure 中，每个资源提供程序都会提供一组用于 Azure 服务的资源和操作。例如 “Microsoft.Compute”（提供虚拟机资源）、“Microsoft.Storage”（提供存储帐户资源）和 “Microsoft.Web”（提供与 Web 应用相关的资源）。而使用 Azure IoT 中心，则需要 “Microsoft.Devices” 提供所需的资源，因此在创建使用 Azure IoT Hub 之前，必须要注册 “Microsoft.Devices”。默认情况下，订阅中的 “Microsoft.Devices” 处于未注册状态，需要先注册，然后才能创建 Azure IoT Hub。

## 解决方法

登录 [Azure 门户](https://portal.azure.cn)，在 “所有服务” 中点击 “订阅” ，选中当前订阅，点击 “资源提供程序”，注册 “Microsoft.Devices”。

![getmanageddisks.PNG](./media/aog-iot-hub-can-not-create-iot-hub/subscription.PNG)

![getmanageddisks.PNG](./media/aog-iot-hub-can-not-create-iot-hub/registerprovider.PNG)

重新进入 IoT Hub 服务，已经可以创建 IoT Hub。

![getmanageddisks.PNG](./media/aog-iot-hub-can-not-create-iot-hub/createiothub.PNG)
---
title: "如何通过代码查询 IoT Hub 的作业信息"
description: "如何通过代码查询 IoT Hub 的作业信息"
author: v-DavidTang
resourceTags: 'IoT Hub, Job'
ms.service: iot-hub
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 5/31/2018
wacn.date: 5/31/2018
---

# 如何通过代码查询 IoT Hub 的作业信息

目前 IotHub 查询作业有两个 API，一个是含有 V2 的 api，另一个则没有 v2 的 api，如果 Java（C#） 通过 JobClient 创建的作业，则只能通过 Java（C#）的 JobClient 才能查询到信息，如果 Java（C#）通过 RegistryManager 创建的作业，则只能通过 Java（C#）的 RegistryManager 才能查询到信息。

如下表所示：

**Create Job :**

|   | RegistryManager | JobClient |
| - | --------------- | --------- |
| C# | ImportDevicesAsync 创建的作业使用的是没有 v2 的 Api | ScheduleTwinUpdateAsync 创建作业使用的是有 v2 的 Api |
| Java | importDevices 方法创建的作业使用的是没有 v2 的 Api | scheduleUpdateTwin 创建作业使用的是有 v2 的 Api |

**Query Job :**

|   | RegistryManager | JobClient |
| - | --------------- | --------- |
| C# | GetJobAsync 使用的没有 V2 的 Api | GetJobAsync 使用的是有 V2 的 Api |
| Java | getJob 使用的没有 V2 的 Api | queryDeviceJob 使用的是有 V2 的 Api |

## 示例代码

- [JobClient 的 Github 示例](https://github.com/Azure/azure-iot-sdk-java/blob/master/service/iot-service-samples/job-client-sample/src/main/java/samples/com/microsoft/azure/sdk/iot/JobClientSample.java)
- [RegistryManager 的 Github 示例](https://github.com/Azure/azure-iot-sdk-java/blob/master/service/iot-service-samples/device-manager-sample/src/main/java/samples/com/microsoft/azure/sdk/iot/DeviceManagerImportSample.java)

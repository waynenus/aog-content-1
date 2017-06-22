---
title: 迁移 Web 应用到新的应用服务计划的相关限制和说明
description: 迁移 Web 应用到新的应用服务计划的相关限制和说明
services: Web Apps
documentationCenter: ''
author: hahaxj
manager: ''
editor: ''

ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-johch
ms.date: 06/21/2017
wacn.date: 06/21/2017
---

# 迁移 Web 应用到新的应用服务计划的相关限制和说明

## 现象描述

当前 Web 应用所在的应用服务计划和目标应用服务计划属于同一个资源组，但是通过 Portal 点击 “更改应用服务计划”，依旧看不到目标应用服务计划。

## 问题分析

导致上述问题的原因是，用户之前手动更改过原应用服务计划的资源组信息。举例如下：

创建时 ：

- Web a 属于应用服务计划 Plan a, Plan a 属于资源组 Resource a;

- Web b 属于应用服务计划 Plan b, Plan b 属于资源组 Resource b;

而后手动将 Web b 和 Plan b 都迁移到了资源组 Recourse a 中，但 Web a 依旧无法从 Plan a 迁移到 Plan b 中，因为 Plan b 的原始资源组是 Resource b; Web a 只能迁移到原始资源组为 Recourse a 的应用服务计划。
---
title: "如何解决 Cosmos DB 使用 Mongo API 时遇到的连接错误"
description: "如何解决 Cosmos DB 使用 Mongo API 时遇到的连接错误"
author: longfeiwei
resourceTags: 'Cosmos DB, Mongo API'
ms.service: cosmos-db
wacn.topic: aog
ms.topic: article
ms.author: longfei.wei
ms.date: 4/30/2018
wacn.date: 4/30/2018
---

# 如何解决 Cosmos DB 使用 Mongo API 时遇到的连接错误

## 问题描述

客户在使用 Cosmos DB Mongo API 时，如果 session 长时间没有活动会偶尔发现报错。
在日志中会出现如下错误： “**prematurely reached end of stream**”。

```
by: com.mongodb.MongoSocketReadException: Prematurely reached end of stream
```

## 问题分析

该类型的问题是由于在连接 Cosmos DB 时会连接 Azure Gateway。 Gateway 会在 session 超过 4 分钟没有任何操作时终结该连接。

## 解决方法

这种类型的问题可以通过以下两种方式解决：

1. 在创建 Mongo Client 的时候使用以下参数方式创建，该操作可以启用 **Socket Keep Alive** 功能，使不活动的连接依然发送心跳包给 Gateway。

```csharp
MongoClientOptions.Builder b = new MongoClientOptions.Builder().socketKeepAlive(true).heartbeatFrequency(1000).maxConnectionIdleTime(18000);
MongoClientURI uri = new MongoClientURI("mongodb://xxxxx:FPe52YbXxqmL09FDrEkPXK3nUDQkZ8ukx5znnwQ5F4nHLhuasuPPYxSQcvhLdmVn5uJdvuG4abvDCnLMjRQ2Lg==@xxxxx.documents.azure.cn:10255/?ssl=true&replicaSet=globaldb",b);
```

2. 在创建 Mongo Client 时，URI 添加 `maxConnectionIdleTime`，在 idle 超过 4 分钟前提前关闭 connection。当重新活动时，创建新的 Client。 这种方式可以用于不方面进行代码变更的项目。

```
mongodb://xxxxx:FPe52YbXxqmL09FDrEkPXK3nUDQkZ8ukx5znnwQ5F4nHLhuasuPPYxSQcvhLdmVn5uJdvuG4abvDCnLMjRQ2Lg==@xxxxx.documents.azure.cn:10255/?ssl=true& replicaSet=globaldb& maxConnectionIdleTime=18000
```
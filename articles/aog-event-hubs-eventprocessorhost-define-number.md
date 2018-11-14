---
title: "EventProcessorHost 如何定义数量"
description: "EventProcessorHost 如何定义数量"
author: Chen Zheng
resourceTags: 'event hubs, EventProcessorHost'
ms.service: event-hubs
wacn.topic: aog
ms.topic: article
ms.author: Chen Zheng
ms.date: 11/7/2018
wacn.date: 11/7/2018
---

# EventProcessorHost 如何定义数量

Azure 事件中心是一个大数据流式处理平台和事件引入服务，每秒能够接收和处理数百万个事件，事件中心针对百万级别的数据提供低延迟、可无缝集成的分布式流处理平台，并在 Azure 的内部和外部提供数据和分析服务用于构建完整的大数据管道。

EventProcessorHost 是一个智能使用者代理，可以简化检查点、租用和并行事件读取器的管理。

1. 如何定义 EventProcessorHost 的 Instance 数量：

    需要基于分区数量，消息数量、机器处理的能力等自行定义 EPH 数量，运行的 EventProcessorHost Instance 会自动分配去取分区的数据实现自动负载。建议把不同的 EventProcessorHost Instance 分配在不同的机器上，EventProcessorHost Instance 尽量 <= 分区数。

2. 接收端假如出现丢数据：

    大量数据接收时假如报 exception，eventhub 会跳过异常，继续接收下一条数据。如果对于数据要求敏感必须全收到，建议使用 service bus 产品，因为 service bus 的 Peeklock 模式可以实现精确接收。如果您想使用 eventhub 精确地接收，通过 github 的 lib 在接收数据时 retry。接收数据实现 retry library 地址：[App-vNext/Polly](https://github.com/App-vNext/Polly)。

    ```csharp
    public async Task ProcessEventsAsync(PartitionContext context, IEnumerable messages)
    {
    // Filter out all the events that must be forwarded to a service bus topic.
    var interactiveEvents = messages.Where(msg => msg.Properties["IsInteractive"] == true).ToList();

    // Publish the filtered events to the topic, wrapped in a Polly retry policy.
    Policy
        .Handle()
        .WaitAndRetryForeverAsync(_ => TimeSpan.FromSeconds(1))
        .ExecuteAsync(() => PublishToServiceBus(interactiveEvents));
    // Set a checkpoint to mark our progress.
    await context.CheckpointAsync();
    }
    ```

## 其他资源

- [EventProcessorHost 介绍](https://docs.azure.cn/zh-cn/event-hubs/event-hubs-event-processor-host)
- [ServiceBus 接收模式](https://docs.azure.cn/zh-cn/service-bus-messaging/service-bus-queues-topics-subscriptions#receive-modes)
---
title: "EventProcessorHost 类使用注意事项一二"
description: "EventProcessorHost 类使用注意事项一二"
author: chenzheng1988
resourceTags: 'event hubs, EventProcessorHost'
ms.service: event-hubs
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/7/2018
wacn.date: 11/7/2018
---

# EventProcessorHost 类使用注意事项一二

Azure 事件中心是一个大数据流式处理平台和事件引入服务，每秒能够接收和处理数百万个事件，事件中心针对百万级别的数据提供低延迟、可无缝集成的分布式流处理平台，并在 Azure 的内部和外部提供数据和分析服务用于构建完整的大数据管道。

EventProcessorHost 是微软开发用于接收 Event Hub 数据的重要技术，当用户使用 .net 技术做 Event Hub 的开发工作，我们强烈推荐用户使用 EventProcessorHost 类处理来自事件中心的数据。EventProcessorHost 为 Event Hub 事件处理实现提供了安全的线程，多进程，安全的运行时环境，还提供了检查点（checkpointing）和分（partition lease）管理。我们就用户关于 EventProcessorHost 类经常咨询的相关问题做以下解释：

1. 如何定义 EventProcessorHost 的 Instance 数量：

    用户需要基于分区数量，消息数量、机器处理的能力等自行评估定义 EPH 数量，运行的 EventProcessorHost Instance 会自动分配去取分区的数据实现自动负载。建议把不同的 EventProcessorHost Instance 分配在不同的机器上，这样可以保证当某台机器性能出现异常时，对应的分区数据可以被其他机器中的EventProcessorHost Instance 获取，从而保证应用正常运行，EventProcessorHost Instance 尽量 <= 分区数这样可以减少不必要的资源浪费。

2. 接收端出现丢数据现象：

    用户有时候在大量数据接收时可能会遇到某些 exception（例如某些数据格式异常，无法被event hub正常处理），eventhub 会跳过异常，继续接收下一条数据，从而保证接收数据不间断。如果对于数据要求敏感必须全收到，建议使用 service bus 产品，因为 service bus 的 Peeklock 模式可以实现精确接收。如果您想使用 eventhub 精确地接收，可以通过在接收数据时加入自定义retry机制。接收数据实现 retry 参考地址：[App-vNext/Polly](https://github.com/App-vNext/Polly)。

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

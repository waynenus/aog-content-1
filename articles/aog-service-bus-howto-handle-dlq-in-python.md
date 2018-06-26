---
title: "Python 处理 Service bus 中的死信队列"
description: "Python 处理 Service bus 中的死信队列"
author: taroyutao
resourceTags: 'Service Bus, Python, DLQ'
ms.service: service-bus
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 06/07/2018
wacn.date: 06/07/2018
---

# Python 处理 Service bus 中的死信队列

## 简介

Azure 服务总线队列和主题订阅提供一个名为 “死信队列 (DLQ)” 的辅助子队列。死信队列不需要显式创建，并且不能删除或以其他方式独立于主实体进行管理。<br>
死信队列的用途是存放无法传递给任何接收方的消息或存放无法处理的消息。 然后，可从 DLQ 中删除和检查这些消息。应用程序可能会在操作员的帮助下，更正问题并重新提交消息，记录出错的实际情况和执行更正操作。<br>
超过 `MaxDeliveryCount` 和超过 `TimeToLive` 是造成消息进入死信队列的主要原因，更多的原因可参考[服务总线死信队列概述](https://docs.azure.cn/zh-cn/service-bus-messaging/service-bus-dead-letter-queues)。

本文主要介绍如何使用 Python SDK 处理进入死信队列中的消息。

## 示例代码

1. 创建 Queue 及发送消息到 Queue：

    ```python
    from azure.servicebus import ServiceBusService, Message

    bus_service = ServiceBusService(
        service_namespace='servicebus',
        shared_access_key_name='RootManageSharedAccessKey',
        shared_access_key_value='PPLtepzy8qg9mfRRqajdjhYPmItwgM8zjyLHFE11rnw=',
        host_base=".servicebus.chinacloudapi.cn")

    # create queue
    queue_options = Queue()
    queue_options.max_size_in_megabytes = '5120'
    queue_options.default_message_time_to_live = 'PT1M'
    bus_service.create_queue('taskqueue', queue_options)
    print("Create queue success")

    # send message to queue
    for i in range(100):
        msg = Message(b'Test Message' + b'i')
        bus_service.send_queue_message('test', msg)
        print(i)

    print("Send end success!")
    ```

2. 使消息进入死信队列:

    ```python
    # 通过使用超过最大传递次数的方式使消息进入死信队列
    while True:
        msg = bus_service.receive_queue_message('test', peek_lock=True)
        time.sleep(1)
        print(msg.body)
    ```

3. 处理进入死信队列的消息：

    ```python
    msg = bus_service.receive_queue_message(ServiceBusService.format_dead_letter_queue_name('test'), peek_lock=True)
    print(msg.body)
    msg.delete()
    ```

> [!NOTE]
> 死信队列实际也是一个队列，路径与常规队列比只是多了 `/$DeadLetterQueue`，所以使用常规队列的消息接收方式，将路径修改为 `<queue>/$DeadLetterQueue` 即可接收对应死信队列中的消息。

## 参考链接

[如何通过 Python 使用服务总线队列](https://docs.microsoft.com/zh-cn/azure/service-bus-messaging/service-bus-python-how-to-use-queues)
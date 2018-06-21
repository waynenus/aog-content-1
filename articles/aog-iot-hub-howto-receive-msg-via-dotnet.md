---
title: ".Net 接收 IotHub 消息的两种方式"
description: "本文讲述.Net 接收 IotHub 消息的两种方式"
author: chenzheng1988
resourceTags: 'IoT Hub, .NET'
ms.service: iot-hub
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 5/31/2018
wacn.date: 5/31/2018
---

# .Net 接收 IotHub 消息的两种方式

## 直接接收

需要指定分区执行循环接收。`ReceiveMessagesFromDeviceAsync` 方法中的 `eventHubClient.GetDefaultConsumerGroup().CreateReceiver()` 第一个参数为分区,第二个参数为 `Offset`，此处为当前日期，只能接收当前启动以后的消息，也可以设置为 `-1`，接收全部的消息。

GitHub 示例地址：[azure-iot-samples-csharp](https://github.com/Azure-Samples/azure-iot-samples-csharp/blob/master/iot-hub/Quickstarts/read-d2c-messages/ReadDeviceToCloudMessages.cs)

```csharp
        static string connectionString = "IotHub 的共享访问策略中的 iothubowner 的连接字符串";
        static string iotHubD2cEndpoint = "messages/events";
        static EventHubClient eventHubClient;
        static void Main(string[] args)
        {
            Console.WriteLine("Receive message. Ctrl-C to exit.\n");
            eventHubClient = EventHubClient.CreateFromConnectionString(connectionString,iotHubD2cEndpoint);
            var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;
            CancellationTokenSource cts = new CancellationTokenSource();

            System.Console.CancelKeyPress += (s, e) =>
            {
                e.Cancel = true;
                cts.Cancel();
                Console.WriteLine("Exiting...");
            };
            var tasks = new List<Task>();
            foreach (string partition in d2cPartitions)
            {
                tasks.Add(ReceiveMessagesFromDeviceAsync(partition,cts.Token));
            }
            Task.WaitAll(tasks.ToArray());
        }
        /// <summary>
        /// 接收设备到 iothub 的消息（仅接收当前时间启动后发送的消息）
        /// </summary>
        /// <param name="partition">分区</param>
        /// <param name="ct"></param>
        /// <returns></returns>
        private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
        {
            var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition,DateTime.UtcNow);
            //var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition,-1");
            while (true)
            {
                if (ct.IsCancellationRequested) break;
                EventData eventData = await eventHubReceiver.ReceiveAsync();
                if (eventData == null) continue;
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition,data);
            }
        }
```

## 通过 EPH 方式接收消息

客户端需要从存储中的 container 读取 offset 位置信息（`-1` 全部读取），采用租约的方式控制并发，防止其他的客户端进入到这个 container 中读取信息。

Github 示例地址：[azure-event-hubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver)

```csharp
private const string EventHubConnectionString = "Event Hubs connection string";
        private const string EventHubName = "event hub name";
        private const string StorageContainerName = "Storage account container name";
        private const string StorageAccountName = "Storage account name";
        private const string StorageAccountKey = "Storage account key";

        private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);

        public static void Main(string[] args)
        {
            MainAsync(args).GetAwaiter().GetResult();
        }

        private static async Task MainAsync(string[] args)
        {
            Console.WriteLine("Registering EventProcessor...");

            var eventProcessorHost = new EventProcessorHost(
                EventHubName,
                PartitionReceiver.DefaultConsumerGroupName,
                EventHubConnectionString,
                StorageConnectionString,
                StorageContainerName);

            // Registers the Event Processor Host and starts receiving messages
            await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

            Console.WriteLine("Receiving. Press enter key to stop worker.");
            Console.ReadLine();

            // Disposes of the Event Processor Host
            await eventProcessorHost.UnregisterEventProcessorAsync();
        }
```
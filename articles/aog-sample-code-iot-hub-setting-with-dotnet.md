---
title: IoT 中心管理 .NET 示例代码
description: IoT 中心管理 .NET 示例代码
services: IoT Hub
documentationCenter: ''
author: allenhula
manager: ''
editor: ''
tags: ''

ms.service: iot-hub
wacn.topic: aog
ms.topic: article
ms.author: v-johch
ms.date: 06/21/2017
wacn.date: 06/21/2017
---

# IoT 中心管理 .NET 示例代码

## 注册本机客户端应用程序到 Azure Acitive Directory 中

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)
2. 在左侧的导航栏中单击 “**Active Directory**” 选择要在其中注册应用程序的租户。
3. 单击 “**应用程序**” 选项卡，然后在底部抽屉中单击 “**添加**”。
4. 根据提示创建一个新的本机客户端应用程序。

    - 应用程序的 “**名称**” 向最终用户描述你的应用程序
    - “**重定向 URI**” 是 Azure AD 要用来返回令牌响应的方案与字符串组合。输入特定于应用程序的值，例如 http://IoTHubManagementDemo。

5. 完成注册后，AAD 将为应用分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“**配置**”选项卡复制此值。
6. 请在“**配置**”选项卡中，找到“**针对其他应用程序的权限**”部分。添加“**Windows Azure Service Management API**”应用程序，在“**委派的权限**”下添加“**Access Azure Service Management**”权限。

## 添加依赖

- [Microsoft.Azure.Management.IotHub](https://www.nuget.org/packages/Microsoft.Azure.Management.IotHub/) （所有IoT相关的管理操作）
- [Microsoft.Azure.Management.ResourceManager.Fluent](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager.Fluent/) （用于构建认证信息）

## 获取认证信息

对于本机客户端应用程序，AAD 不会为之生成客户端密钥，所以要通过用户登陆来获取认证信息。

    var adServiceSettings = new ActiveDirectoryServiceSettings
    {
        AuthenticationEndpoint = new Uri(AzureEnvironment.AzureChinaCloud.AuthenticationEndpoint),
        TokenAudience = new Uri(AzureEnvironment.AzureChinaCloud.ResourceManagerEndpoint),
        ValidateAuthority = true
    };
    var adClientSettings = new ActiveDirectoryClientSettings()
    {
        ClientId = nativeClientId,
        ClientRedirectUri = new Uri(redirectUri)
    };
    SynchronizationContext.SetSynchronizationContext(new SynchronizationContext());
    var azureCredential = UserTokenProvider.LoginWithPromptAsync(tenantId, adClientSettings, adServiceSettings).Result;

## 获取 IoT Hub 配置并更新

    var iothubClient = new IotHubClient(
    new Uri("https://management.chinacloudapi.cn/"), 
    azureCredential, 
    new RetryDelegatingHandler());
    iothubClient.SubscriptionId = subscriptionId;

    var iothubResource = iothubClient.IotHubResource;

    // get iothub description
    var resourceDescription = iothubResource.Get(rgName, resourceName);
    Console.WriteLine(resourceDescription.Name);

    // set C2D message default ttl to 2 hours
    resourceDescription.Properties.CloudToDevice.DefaultTtlAsIso8601 = TimeSpan.FromHours(2);

    try
    { 
        // commit the change                 
    iothubResource.CreateOrUpdate(rgName, resourceName, resourceDescription);
        Console.WriteLine("Update successfully!");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
    Console.WriteLine("Press ENTER to exit!");
    Console.ReadLine();            

## 源代码链接

[IoTHubManagementDemo](https://github.com/wacn/AOG-CodeSample/tree/master/IotHub/CSharp/IoTHubManagementDemo)
---
title: "如何在 Service Fabric 配置 Docker Daemon 启动参数"
description: "如何在 Service Fabric 配置 Docker Daemon 启动参数"
author: chenzheng1988
resourceTags: 'Service Fabric, Docker Daemon'
ms.service: service-fabric
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/8/2018
wacn.date: 11/8/2018
---

# 如何在 Service Fabric 配置 Docker Daemon 启动参数

通过模板部署 Service Fabric ，在模板中搜索 **fabric Settings**，在 **fabric Settings** 里添加如下代码（示例是与不安全的注册表通信）：

```json
,{
    "name":"Hosting",
    "parameters": [
        {
            "name": "ContainerServiceArguments",
            "value": "—insecure-registry 127.0.0.1:5000"
        }
    ]
}
```

添加代码处如图所示：

![deamon-parameters](media/aog-service-fabric-howto-config-daemon-start-parameters/deamon-parameters.png "deamon-parameters")

然后保存，重新部署。

> [!NOTE]
> 使用 Service Fabric 运行时的 6.2 版本及更高版本，您可以使用自定义参数启动 Docker Daemon。

## 其他资源

- [Service Fabric 启动 Daemon 程序](https://docs.azure.cn/zh-cn/service-fabric/service-fabric-get-started-containers-linux#start-the-docker-daemon-with-custom-arguments)
- [Docker 参数](https://docs.docker.com/engine/reference/commandline/dockerd/)
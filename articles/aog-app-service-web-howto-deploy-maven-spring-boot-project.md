---
title: 如何在 Web 应用上部署 Maven Spring Boot 项目
description: 如何在 Web 应用上部署 Maven Spring Boot 项目
service: ''
resource: App Service Web
author: maysmiling
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'App Service Web, Maven Spring Boot'
cloudEnvironments: MoonCake

ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-amazha
ms.date: 09/22/2017
wacn.date: 09/22/2017
---
# 如何在 Web 应用上部署 Maven Spring Boot 项目

可以通过以下步骤在 Web 应用上完成 Maven Spring Boot 项目的部署：

1. 构建 Maven 的 SpringBoot 项目，您可以在此下载[示例代码](https://github.com/wacn/AOG-CodeSample/tree/master/AppServiceWeb/Java/SpringBootMavenSampleForWebapp-1.0.0)。

2. 在 Azure 中创建一个 Web 应用。

    ![01](media/aog-app-service-web-howto-deploy-maven-spring-boot-project/01.png)

3. 配置 Java 运行环境。

    ![02](media/aog-app-service-web-howto-deploy-maven-spring-boot-project/02.png)

4. 上载 SpringBoot 的可执行 jar 文件到 wwwroot 下。

    ![03](media/aog-app-service-web-howto-deploy-maven-spring-boot-project/03.png)

5. 配置 web.config 文件，内容如下, 相关参数请按照实际情况进行编写：

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
    <system.webServer>
        <handlers>
        <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" />
        </handlers>
        <httpPlatform processPath="%JAVA_HOME%\bin\java.exe" stdoutLogEnabled ="true"
            arguments="-Djava.net.preferIPv4Stack=true -Dserver.port=%HTTP_PLATFORM_PORT% -jar &quot;%HOME%\site\wwwroot\springbootsample-1.0.0.jar&quot;">
        </httpPlatform>
    </system.webServer>
    </configuration>
    ```
6. 启动应用执行。

    ![04](media/aog-app-service-web-howto-deploy-maven-spring-boot-project/04.png)

## 链接资源

[在 App Service 中配置 Web 应用](https://docs.microsoft.com/zh-cn/azure/app-service-web/web-sites-java-custom-upload)
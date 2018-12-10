---
title: "Web 应用如何自定义错误页面"
description: "Web 应用如何自定义错误页面"
author: zhangyannan-yuki
resourceTags: 'App Service Web, Error Page'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-ciwu
ms.date: 12/5/2018
wacn.date: 12/5/2018
---

# Web 应用如何自定义错误页面

## 解决方案

### .NET 网站

需要在 web.config 中进行配置，若没有这个配置文件，需要上传一个 web.config 文件到 `site/wwwroot` 目录下，具体配置如下：

```xml
?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <system.webServer>
    <httpErrors errorMode="DetailedLocalOnly" defaultResponseMode="File" >
        <remove statusCode="404" subStatusCode="-1" />
        <error statusCode="404" path="/error.html" responseMode="ExecuteURL"/>
        <remove statusCode="403" subStatusCode="-1" />
        <error statusCode="403" path="/error.html" responseMode="ExecuteURL"/>
    </httpErrors>
    </system.webServer>
</configuration>
```

配置好之后最好可以在门户重启一下网站。

> [!NOTE]
> 可以在 `statusCode` 指定状态码 ，`path` 为指定的文件（可以指定同一个文件也可以指定不同文件内容）。

关于参数介绍可以参考 [HTTP Errors](https://docs.microsoft.com/iis/configuration/system.webServer/httpErrors/)。

### Java 网站

需要在 Java 的 web.xml 进行配置 error-page，具体内容如下：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<web-app
    xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" version="3.1" metadata-complete="true">
<display-name>Welcome to Tomcat</display-name>
<welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
<description>Welcome to Tomcat</description>
<error-page>
    <error-code>404</error-code>
    <location>/404.html</location>
</error-page>
<error-page>
    <error-code>403</error-code>
    <location>/403.html</location>
</error-page>
<error-page>
    <error-code>500</error-code>
    <location>/500.html</location>
</error-page>
</web-app>
```

配置好之后再重新发布网站。
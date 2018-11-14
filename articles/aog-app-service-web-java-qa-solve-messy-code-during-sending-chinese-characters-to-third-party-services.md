---
title: "如何解决 Java Web 应用在向第三方服务传送中文时出现乱码的问题"
description: "如何解决 Java Web 应用在向第三方服务传送中文时出现乱码的问题"
author: 123Jun321
resourceTags: 'App Service Web, Java , Garbled'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/9/2018
wacn.date: 11/9/2018
---

# 如何解决 Java Web 应用在向第三方服务传送中文时出现乱码的问题

## 问题描述

部署在 Azure 上的 Web APP 应用在向第三方传送中文字符串时，第三方服务接受到的是类似于 `？？` 之类的乱码，而本地运行发送是正常的。

## 问题分析

这是因为 APP Service 环境的默认编码为 GBK ，在向第三方发送数据时要做默认的转码工作，即执行 `new String (“您要传送的字符串”.getBytes(),”UTF-8”)` ，而 getBytes 方法如果没有参数的话会使用系统默认的编码方式编码， gbk 编码和 utf-8 编码方式不同，所以在转换之后会出现乱码的现象。

## 解决方法

若要解决这个问题，在 **site/wwwroot** 文件夹下创建 **web.config** 文件（如果已经有请直接修改），添加以下内容，主要目的时修改 jvm 的默认编码方式，修改成 utf-8 保证传输中编码不会出错：

```xml
<configuration>
  <system.webServer>
    <handlers>
      <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" />
    </handlers>
    <httpPlatform processPath="%AZURE_TOMCAT85_HOME%\bin\startup.bat" arguments="">
      <environmentVariables>
        <environmentVariable name="JAVA_OPTS" value="-Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8" />
      </environmentVariables>
    </httpPlatform>
  </system.webServer>
</configuration>
```
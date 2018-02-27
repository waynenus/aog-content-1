---
title: 'Web 作业无法直接运行 jar 文件'
description: 'Web 作业无法直接运行 jar 文件'
author: maysmiling
resourceTags: 'Web Apps, Web Jobs, jar, bat'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 02/27/2018
wacn.date: 01/20/2017
---

# Web 作业无法直接运行 jar 文件

## **问题描述**

在 [Azure 门户](https://portal.azure.cn)中将直接压缩的 jar 文件打包为 zip 包，上传到 web 作业时报错。 jar 文件打包为 zip 包，上传到 web 作业时报错。

## **解决方法**

jar 文件的运行需要依托于 java 进程，所以在运行 jar 文件时，我们都会以格式 `“java –jar xxx.jar”` 的方式运行。由于 web 作业后台默认直接运行上传的文件，所以需要我们在 jar 文件外层再包含一层来直接调用 jar 文件，通常会以 bat 格式的文件来进行调用。格式如下：

```
set JAVA_HOME=D:\Program Files (x86)\Java\jdk1.8.0_73
set PATH=%JAVA_HOME%\bin;%PATH%
java -Djava.net.preferIPv4Stack=true -jar <xxx>.jar
```

## 链接资源：

[Executing Java Web Jobs on Azure](https://blogs.msdn.microsoft.com/azureossds/2015/04/28/executing-java-web-jobs-on-azure/)
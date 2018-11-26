---
title: "使用 az 命令通过模板创建 MySQL PaaS"
description: "使用 az 命令通过模板创建 MySQL PaaS"
author: xingbing0909
resourceTags: 'Mysql, az, template'
ms.service: Mysql
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/20/2018
wacn.date: 11/20/2018
---

# 使用 az 命令通过模板创建 MySQL PaaS

选择新建 MySQL PaaS，填写参数之后，不选择创建，选择自动化模板，然后点下载，将 template.json 和 parameters.json 文件下载。

![01](media/aog-mysql-howto-create-mysql-by-template-on-az/01.png "01")

![02](media/aog-mysql-howto-create-mysql-by-template-on-az/02.png "02")

首先使用 `az` 命令登陆 Azure 门户账号：

![03](media/aog-mysql-howto-create-mysql-by-template-on-az/03.jpg "03")

接下来使用命令 `az group deployment` 来部署：

![04](media/aog-mysql-howto-create-mysql-by-template-on-az/04.jpg "04")

此处 `Servername` 以 `zhtest` 为例，其中 parameters.json 文件里，添加密码参数：

![05](media/aog-mysql-howto-create-mysql-by-template-on-az/05.jpg "05")

最后部署成功：

![06](media/aog-mysql-howto-create-mysql-by-template-on-az/06.jpg "06")
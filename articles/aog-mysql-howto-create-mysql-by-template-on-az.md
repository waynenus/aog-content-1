---
title: "使用 az 命令通过模板在Linux上创建 MySQL Database on Azure"
description: "使用 az 命令通过模板在Linux上创建 MySQL Database on Azure"
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

在Ibiza门户上选择新建 MySQL Database on Azure，填写参数之后，不选择创建，选择自动化模板，然后点下载，将 template.json 和 parameters.json 文件下载。

![01](media/aog-mysql-howto-create-mysql-by-template-on-az/01.png "01")

![02](media/aog-mysql-howto-create-mysql-by-template-on-az/02.png "02")

首先使用 `az login` 命令登陆 Azure 门户账号，可以先执行 `az cloud set --name AzureChinaCloud`设置登陆环境为中国区Azure

![03](media/aog-mysql-howto-create-mysql-by-template-on-az/03.jpg "03")

接下来在parameters.json中添加密码参数，然后使用命令 `az group deployment` 来部署：
az group deployment create --subscription "<subscription_name>" --name "<server_name>" --resource-group "<resourcegroup_name>" --template-file "<template.json_path>" --parameters "<parameters.json_path>"

![04](media/aog-mysql-howto-create-mysql-by-template-on-az/04.jpg "04")

此处 <server_name> 以 `zhtest` 为例

![05](media/aog-mysql-howto-create-mysql-by-template-on-az/05.jpg "05")

最后部署成功：

![06](media/aog-mysql-howto-create-mysql-by-template-on-az/06.jpg "06")

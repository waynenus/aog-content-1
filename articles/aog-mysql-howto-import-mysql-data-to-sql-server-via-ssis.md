---
title: "通过 SSIS 导入 MySQL 数据到 SQL Server"
description: "通过 SSIS 导入 MySQL 数据到 SQL Server"
author: xingbing0909
resourceTags: 'Mysql, SQL Server, SSIS'
ms.service: Mysql
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/20/2018
wacn.date: 11/20/2018
---

# 通过 SSIS 导入 MySQL 数据到 SQL Server

1. 安装并配置 ODBC 驱动：

    ![01](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/01.jpg "01")

2. 在 Visual Studio 中创建 SSIS package, 配置源组件和目标组件：

    ![02](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/02.jpg "02")

    ![03](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/03.jpg "03")

    ![04](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/04.jpg "04")

3. 点击 **start** 开始运行，运行成功后保存并发布包。

    ![05](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/05.jpg "05")

4. 在 SSMS 里面创建 job, 新增 step 配置如下：

    ![06](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/06.jpg "06")

5. 勾选 32 位运行环境：

    ![07](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/07.jpg "07")

6. 点击确定，运行 job, 即可成功运行：

    ![08](media/aog-mysql-howto-import-mysql-data-to-sql-server-via-ssis/08.jpg "08")
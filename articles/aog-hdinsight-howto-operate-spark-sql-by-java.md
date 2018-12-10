---
title: "如何用 Java 操作 Spark SQL"
description: "如何用 Java 操作 Spark SQL"
author: Li Bao
resourceTags: 'Hdinsight, Spark SQL'
ms.service: hdinsight
wacn.topic: aog
ms.topic: article
ms.author: Li Bao
ms.date: 12/6/2018
wacn.date: 12/6/2018
---

# 如何用 Java 操作 Spark SQL

对于如何用 Java 操作 Spark SQL 的 demo 您可以参考该[示例代码](D:\GitHub\AOG-CodeSample\HDInsight\aog-hdinsight-howto-operate-spark-sql-by-java\JavaSparkSQLDemo-master)：

1. 将示例代码下载到本地后，用 maven 打包成 sql-1.0.jar。
2. 用 WinSCP 工具将打包成的 sql-1.0.jar 上传到您的 Spark 集群的 `/home/sshuser/` 目录下。
3. 在头结点上直接运行如下语句即可运行该示例代码：

    ```shell
    spark-submit \
    --class com.libao.spark.App \
    --name App \
    --master yarn \
    --executor-memory 1G \
    --num-executors 2 \
    /home/sshuser/sql-1.0.jar
    ```

通过如上步骤就可以将一个 Java 写的 Spark SQL 程序运行在您的 Spark 集群上了。
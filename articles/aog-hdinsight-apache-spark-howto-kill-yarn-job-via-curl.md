---
title: 如何使用 Curl 停止 Spark 中某个正在运行的 Yarn 作业
description: 在 Spark 群集中，如果想停止某个正在运行的 Yarn 作业, 不仅可以使用 SSH 登录后来停止，也可以通过 Curl 来提交停止的请求。
service: ''
resource: HDInsight
author: jiangziang
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'HDInsight, Curl, Spark Cluster, Yarn Job'
cloudEnvironments: MoonCake

ms.service: hdinsight
wacn.topic: aog
ms.topic: article
ms.author: ziangj
ms.date: 10/25/2017
wacn.date: 10/25/2017
---

# 如何使用 Curl 停止 Spark 中某个正在运行的 Yarn 作业

在当前的 Spark 群集中，如果我们想停止某个正在运行的 Yarn 作业，我们可以使用 SSH 登录到已有的 Spark 节点，使用 `yarn application` 来实现（如下图）。

![01](media/aog-hdinsight-apache-spark-howto-kill-yarn-job-via-curl/01.png)

除此之外，我们也可以通过 Curl 来提交停止的请求，下面是具体的步骤：

1.	使用以下命令找到所有正在运行的作业：

    `curl -u <ambari user name>:<ambari password> -H "Content-Type: application/json" -X GET https://<cluster name>.azurehdinsight.cn/ws/v1/cluster/apps/<Application ID>`

    ![02](media/aog-hdinsight-apache-spark-howto-kill-yarn-job-via-curl/02.png)

2.	确认待停止的作业后，使用以下命令终止正在运行的作业：

    `curl -u <ambari user name>:<ambari password> -v -X PUT -H "Content-Type: application/json" -d '{"state": "KILLED"}' "https://<cluster name>.azurehdinsight.cn/ws/v1/cluster/apps/<application ID>/state"`

    ![03](media/aog-hdinsight-apache-spark-howto-kill-yarn-job-via-curl/03.png)

3.	运行成功后，运行语句确认作业状态。

    ![04](media/aog-hdinsight-apache-spark-howto-kill-yarn-job-via-curl/04.png)
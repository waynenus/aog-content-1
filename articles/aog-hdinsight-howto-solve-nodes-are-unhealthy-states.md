---
title: "如何解决集群的 Node 节点都处在 Unhealthy 状态"
description: "如何解决集群的 Node 节点都处在 Unhealthy 状态"
author: Li Bao
resourceTags: 'Hdinsight, Node'
ms.service: hdinsight
wacn.topic: aog
ms.topic: article
ms.author: Li Bao
ms.date: 12/6/2018
wacn.date: 12/6/2018
---

# 如何解决集群的 Node 节点都处在 Unhealthy 状态

## 问题描述

在客户的 Yarn UI 可以看到有 `local-dirs are bad`，`log-dirs are bad` 报错，node 节点都是 unhealthy 状态，如图：

![01](media/aog-hdinsight-howto-solve-nodes-are-unhealthy-states/01.png "01")

## 原因分析

之所以会有 `local-dirs are bad`，`log-dirs are bad` 报错，是因为在我们集群的 yarn-site.xml 里参数 `yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage` 的默认值是 90%，到达到这个值会触发上述报错。之所以会触发 `yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage` 的默认值是因为：

1. 客户的数据是存储在 Storage 里的，在进行计算之前需要将数据先读取到各个节点，客户在读取数据生成 RDD 之后对 RDD 的缓存采用的是 `MEMORY_AND_DISK_2`，也没有采用序列化的方式，也就是说如果内存存不下会存到本地磁盘，而且会存在不同的节点上存 2 份。
2. 在计算的过程中会产生大量的中间数据结果，如果内存中存不下也会将结果写到磁盘上。
3. 一个 batch 的数据还没处理完，下一个 batch 就来了，上一个 batch 占用的内存还没释放（batch 任务还没执行完），最后都会堆积到磁盘上。

## 解决方法

1. 首先要先 kill 掉所有的任务，因为当前客户的所有 node 节点已经是 unhealthy 状态了，可以采用如下办法 kill 所有的任务：

    ```shell
    for app in `yarn application -list | awk '$6 == "ACCEPTED" { print $1 }'`; do yarn application -kill "$app";  done
    ```

2. 重启所有节点。
3. 重新提交任务。

## 如何避免

1. 将 `yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage` 的值改大，这样治标不治本。
2. 将 RDD 的存储级别改成 `MEMORY_ONLY_SER`, 这样会节省内存空间，内存存不下的数据会重新计算。
3. 优化代码逻辑，读尽量少的数据。
4. 放缓批处理的速度（客户采用的是 crontab 定时任务）。
5. 如果业务数据量确实大，任务多，可以增加计算节点。
6. 多观察 Ambari 的监控数据，做到提早预防。
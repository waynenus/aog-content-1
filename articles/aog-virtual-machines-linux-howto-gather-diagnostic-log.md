---
title: "如何收集 Linux 虚拟机的诊断日志"
description: "如何收集 Linux 虚拟机的诊断日志"
author: chpaadmin
resourceTags: 'Virtual Machines, Linux, Diagnostic Log'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: chpa
ms.date: 06/30/2018
wacn.date: 06/30/2018
---

# 如何收集 Linux 虚拟机的诊断日志

当 Linux 虚拟机遇到异常时，技术支持工程师经常会通过 Linux 系统的系统日志来对异常问题进行分析和排查，进行修复。以下针对 CentOS/Redhat, Ubuntu, Suse 这三个应用比较广泛的操作系统进行收集系统日志的步骤说明。

## CentOS/Redhat

如何收集 CentOS/Redhat 的系统日志：

1. 以管理员身份登陆虚拟机。

2. 安装 sos 软件包，命令如下：

    ```shell
    # yum install sos
    ```

3. 安装完毕后，执行 `sosreport` 命令，并按照向导收集系统日志：

    ```shell
    # sosreport -e azure
    ```

4. 收集完毕后，将生成的系统日志压缩文件提交给技术支持工程师进行分析排查。


## Ubuntu

如何收集 Ubuntu 的系统日志：

1. 以管理员身份登陆虚拟机。

2. 安装 sos 软件包，命令如下：

    ```shell
    # apt-get install sosreport
    ```

3. 安装完毕后， 执行 `sosreport` 命令，并按照向导收集系统日志：

    ```shell
    # sosreport -e azure
    ```

4. 收集完毕后，将生成的系统日志压缩文件提交给技术支持工程师进行分析排查。

## SUSE

如何收集 SUSE 的系统日志：

1. 以管理员身份登陆虚拟机。

2. 安装 supportconfig 软件包，命令如下：

    ```shell
    # zypper install supportutils
    ```

3. 安装完毕后， 执行 `supportconfig` 命令，并按照向导收集系统日志：

    ```shell
    # supportconfig -Al
    ```

4. 收集完毕后，将生成的系统日志压缩文件提交给技术支持工程师进行分析排查。
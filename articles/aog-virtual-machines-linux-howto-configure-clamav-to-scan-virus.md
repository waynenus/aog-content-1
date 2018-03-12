---
title: "如何在 Linux 环境下安装配置 ClamAV 进行病毒扫描"
description: "如何在 Linux 环境下安装配置 ClamAV 进行病毒扫描"
author: chpaadmin
resourceTags: 'Virutal Machines, Linux, ClamAV, Scan Virus'
ms.service: virtual-machines-linux
wacn.topic: aog
ms.topic: article
ms.author: chpa
ms.date: 02/28/2018
wacn.date: 02/28/2018
---

# 如何在 Linux 环境下安装配置 ClamAV 进行病毒扫描

## 问题描述

在当下, Linux 服务器的安全问题越来越频发,服务器被恶意入侵,业务数据被恶意删除和加密以及服务器被劫持作为 DDos 肉鸡等.为了增强 Linux 服务器的安全性,给大家推荐一款开源的防病毒软件 ClamAV,并附上相关安装配置步骤供参考.

关于 ClamAV 的更多详情, 参考 [ClamAV 官网](https://www.clamav.net/)。

> [!IMPORTANT]
> 1. 由于 ClamAV 是开源软件,在使用过程中,遇到任何问题,无法获取官方技术支持。
> 2. 文档测试环境为 CentOS 7.4,如果您选用的是其他版本的 Linux 系统环境, 请注意其差别性。

## 操作步骤

### 安装 ClamAV

1. 执行如下命令, 下载 ClamAV 的 RPM 软件包.如果链接失效, 请访问其他 RPM 网站进行下载.

    ```
    # wget --no-check-certificate http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/clamd-0.99.3-3470.el7.art.x86_64.rpm
    # wget --no-check-certificate http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/clamav-db-0.99.3-3470.el7.art.x86_64.rpm
    # wget --no-check-certificate http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/clamav-0.99.3-3470.el7.art.x86_64.rpm
    ```

2. 执行如下命令进行安装:

    ```
    # rpm -ivh clamav-db-0.99.3-3470.el7.art.x86_64.rpm
    # rpm -ivh clamav-0.99.3-3470.el7.art.x86_64.rpm
    # rpm -ivh clamd-0.99.3-3470.el7.art.x86_64.rpm
    ```

### 配置 ClamAV

1. 修改/etc/freshclam.conf 配置文件:

    - 找到包含 `Example` 的行, 添加 `#` 号, 进行注释。
    - 找到包含 `#DatabaseOwner clamav` 的行,改为 `DatabaseOwner root`。
    - 找到包含 `#UpdateLogFile /var/log/freshclam.log` 的行,将 `#` 号去除。
    - 找到包含 `#LogFileMaxSize 2M` 的行,将 `#` 号去除, 并设置相应的大小,比如 20M。
    - 找到包含 `#LogRotate yes` 的行，将 `#` 号去除。
    - 找到包含 `#LogTime yes` 的行,将 `#` 号去除。
    - 找到包含 `#DatabaseDirectory /var/lib/clamav` 的行,将 `#` 号去除。

2. 保存并退出.

### 更新病毒库

1. 执行如下命令创建病毒库目录:

    ```
    # mkdir /var/lib/clamav
    # chmod 755 /var/lib/clamav/
    ```

2. 执行如下命令,更新病毒库(第一次更新需要半个小时以上, 请耐心等待):

    ```
    # freshclam --datadir=/var/lib/clamav/
    ```

### 进行病毒扫描

1. 执行如下命令进行病毒扫描: 

    ```
    # clamscan -ri </path1/to/scan> </path2/to/scan>
    ```

2. `clamscan` 常用选项说明:

    ```
    --recursive[=yes/no(*)]     -r      递归查找
    --infected                  -I      只打印受影响的文件信息
    --remove[=yes/no(*)]                删除受影响的文件.(不建议采用,根据扫描结果进行手动删除,避免误删)
    ```

3. 更多参数说明, 请参考: `#clamscan --help`。

### 定期扫描

1. 为了不影响 Linux 服务器的正常使用,建议将病毒库升级操作和病毒扫描操作放在非业务高峰期间运行。
2. 以下是定时任务的范例:

    - `30 2 * * * /bin/freshclam --datadir=/var/lib/clamav/`
    - `30 3 * * * /bin/clamscan -ri <path/to/scan> | mail -s "clamscan daily report" 'youremailaddress'`

上述范例会在每天凌晨 2 点半进行病毒库更新, 在凌晨 3 点半进行病毒扫描, 并通过邮件方式把扫描结果发送到邮箱。

---
title: "如何手动设置发送心跳信号防止数据库连接过期"
description: "如何手动设置发送心跳信号防止数据库连接过期"
author: kustbilla
resourceTags: 'SQL Database, KeepAlive'
ms.service: sql-database
wacn.topic: aog
ms.topic: article
ms.author: weihuan
ms.date: 3/31/2018
wacn.date: 3/31/2018
---

# 如何手动设置发送心跳信号防止数据库连接过期

## 概述

应用客户端需要先通过 Azure 网关( Gateway )才能连接到 Azure 上的 PaaS 数据库(例如 Azure SQL Database，MySQL Database on Azure 等)，网关连接的默认设置是四分钟，若四分钟内无数据包发送，该连接会过期。

为避免连接过期出现报错，客户可以选择采用连接池的方式，通过为每个给定的连接配置保留一组活动连接来更好地管理连接。此外客户也可以选择设置发送心跳信号的方式确保所用连接一直处于活动状态，以避免被网关认定为过期。

对于 SQL Server Management Studio 和 MySQL Workbench 等工具而言，其中已经内置了心跳信号设置功能；而如果用的是客户自行开发的应用，就需要手动设置心跳信号，具体方法可参考以下内容。

## 设置心跳信号

以下分别为在 Windows 系统和 Linux 系统中设置心跳信号的方法，参数修改值可酌情自选。

### Windows 系统

可以打开 "运行", 输入 `regedit`, 修改(如果没有则添加)注册表：<br>
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveTime`<br>
打开 Parameters 后,在右侧界面空白处鼠标单击右键创建 `DWORD(32 位)`值。<br>
把名字修改为 `KeepAliveTime`, 值选十进制，输入 `60000`。<br>
之后重启服务器以使注册表更改生效。

### Linux 系统

对于 Linux 客户端而言，需要修改以下四个 `keepalive` 参数：<br>

```
tcp_keepalive_probes - the number of probes that are sent and unacknowledged before the client considers the connection broken and notifies the application layer
tcp_keepalive_time - the interval between the last data packet sent and the first keepalive probe
tcp_keepalive_intvl - the interval between subsequent keepalive probes
tcp_retries2 - the maximum number of times a packet is retransmitted before giving up
```

修改的方法是在 Linux 上运行以下四个 echo 命令：

```
echo "6" > /proc/sys/net/ipv4/tcp_keepalive_time
echo "1" > /proc/sys/net/ipv4/tcp_keepalive_intvl
echo "10" > /proc/sys/net/ipv4/tcp_keepalive_probes
echo "3" > /proc/sys/net/ipv4/tcp_retries2
```

`tcp_keepalive_time` 和 `tcp_keepalive_intvl` 值的单位是秒。若要使得修改的值在系统重启后仍然生效，需要将这两个参数添加到 `/etc/sysctl.conf` 中。
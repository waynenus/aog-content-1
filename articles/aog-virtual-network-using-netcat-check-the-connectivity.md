---
title: "使用 Netcat 测试虚拟机 TCP/UDP 端口连通性"
description: "使用 Netcat 测试虚拟机 Tcp/UDP 端口连通性"
author: Dillion132
resourceTags: 'virtual network'
ms.service: Virtual Network
wacn.topic: aog
ms.topic: article
ms.author: v-zhilv
ms.date: 05/23/2018
wacn.date: 05/23/2018
---

# 使用 Netcat 测试虚拟机 TCP/UDP 端口连通性

Netcat 是一个用于 TCP/UDP 连接和监听的工具，主要用于网络传输和调试。本文主要介绍如何使用 Netcat 测试 Windows/Linux 虚拟机 TCP/UDP 端口的连通性。主要包含以下内容：

- [Linux OS 环境下，使用 Netcat 命令分别测试 TCP/UDP 端口连通性](#linuxos)

- [Windows OS 环境下，使用 Netcat 命令分别测试 TCP/UDP 端口连通性](#windowsos)

## 前提条件

1. 使用 Azure 门户创建 [Linux 虚拟机](https://docs.azure.cn/virtual-machines/linux/quick-create-portal) 和 [Windows 虚拟机](https://docs.azure.cn/virtual-machines/windows/quick-create-portal)。本文 Linux 虚拟机使用的是 CentOS 镜像，Windows 虚拟机使用的是 Windows server 2012 DataCenter 镜像。

2. 在客户端和服务器端虚拟机上分别安装 Netcat 工具。

    - Linux 虚拟机 ( Centos 版本) 安装命令如下：

        ```bash
        sudo yum install nc
        ```

    - Windows 虚拟机可以从 Internet 下载 Netcat 工具包。

## <a id="linuxos"></a>Linux OS 环境下，使用 Netcat 命令分别测试 TCP/UDP 端口连通性

### 测试 TCP 端口连通性

本示例创建一台 Azure Linux 虚拟机做为服务器，通过 5000 端口测试 TCP 端口连通性，具体步骤如下：

1. 通过 Azure 门户选中创建的 Linux 虚拟机，在边栏选项卡中选中网络，在网络安全组中添加 TCP 5000 入站端口规则。

    ![linux-nsg-tcp.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/linux-nsg-tcp.PNG)

    > [!NOTE]
    > 注意网络安全组规则的优先级，确保入站规则有效，并处于 “**允许**” 状态。
    > 默认情况下 Linux 虚拟机中未启用 “iptables” 服务，如果启用 “iptables” 服务，则需要在 “iptables” 中打开相应的端口。在本示例中 Linux 虚拟机未启用 iptables 服务。

2. 远程连接到 Linux 虚拟机，安装 Netcat 工具之后，执行以下命令，实现 TCP 方式监听服务器 5000 端口

    ```bash
    nc -l <端口号> -v
    ```

    > [!NOTE]
    > -l：指明 Netcat 处于 TCP 监听模式；-v：用于显示详细信息。
    > 如果想要退出监听模式，可以使用 Ctrl + C 。
    > **必须要先在服务器端监听端口，然后在客户端访问该端口，才能建立连接。**

    在安装有 Netcat 的客户端机器上，执行以下命令：

    ```bash
    nc <服务器端 IP 地址> <端口号>
    ```

    在客户端输入如上命令后，可以在客户端输入任意字符，我们可以看到，客户端输入的字符均会在服务器端打印出来。测试结果如下：

    客户端(本示例客户端使用的是 Linux 虚拟机，如果客户端机器是 windows 系统，使用 cmd 命令提示符进入到 nc.exe 所在目录，然后执行如上命令)：

    ![linux-tcp-client.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/linux-tcp-client.PNG)

    服务器端：

    ![linux-tcp-server.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/linux-tcp-server.PNG)

### 测试 UDP 端口连通性

1. 通过 Azure 门户选中创建的 Linux 虚拟机 ，在边栏选项卡中选中网络，在网络安全组中添加 UDP 5001 入站端口规则。

    ![linux-nsg-udp.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/linux-nsg-udp.PNG)

    > [!NOTE]
    > 注意网络安全组规则的优先级，确保入站规则有效，并处于 “**允许**” 状态。
    > 默认情况下 Linux 虚拟机中未启用 “iptables” 服务，如果启用 “iptables” 服务，则需要在 “iptables” 中打开相应的端口。在本示例中 Linux 虚拟机未启用 iptables 服务。

2. 远程连接到 Linux 虚拟机，安装 Netcat 工具之后，执行以下命令，实现 UDP 方式监听 5001 端口

    ```bash
    nc -lu <端口号> -v
    ```

    > [!NOTE]
    > -lu：指明 Netcat 处于 UDP 监听模式；-v：用于显示详细信息。
    > 如果想要退出监听模式，可以使用 Ctrl + C 。
    > **必须要先在服务器端监听端口，然后在客户端访问该端口，才能建立连接。**

    在安装有 Netcat 的客户端机器上，执行以下命令：

    ```bash
    nc -u <服务器端 IP 地址> <端口号>
    ```

    在客户端输入如上命令后，可以在客户端输入任意字符，我们可以看到，客户端输入的字符均会在服务器端打印出来。测试结果如下：

    客户端(本示例客户端使用的是 Linux 虚拟机，如果客户端机器是 windows 系统，使用 cmd 命令提示符进入到 nc.exe 所在目录，然后执行如上命令)：

    ![linux-udp-client.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/linux-udp-client.PNG)

    服务器端：

    ![linux-udp-server.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/linux-udp-server.PNG)

## <a id="windowsos"></a>Windows OS 环境下，使用 Netcat 命令分别测试 TCP/UDP 端口连通性

### 测试 TCP 端口连通性

本示例创建一台 Azure Windows 虚拟机做为服务器，本地 Windows 计算机做为客户端，客户端和服务器通过 5000 端口相连，具体步骤如下：

1. 通过 Azure 门户选中 Windows 虚拟机 ( dillionvm )，在边栏选项卡中选中网络，在网络安全组中添加 TCP 5000 入站端口规则。

    ![windows-nsg-tcp.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/windows-nsg-tcp.PNG)

    > [!IMPORTANT]
    > 注意网络安全组规则的优先级，确保入站规则有效，并处于 “**允许**” 状态。
    > 对于 Windows 虚拟机，从 Azure 门户添加完入站端口规则之后，需要远程连接到虚拟机，在防火墙中添加相对应的 TCP 入站规则。

2. 远程连接到 Azure Windows 虚拟机，使用 cmd 命令导航至 nc.exe 所在目录，执行以下命令，实现 TCP 方式监听服务器端 5000 端口。

    ```
    nc -l -p <端口号> -v
    ```

    > [!NOTE]
    > -l：指明 Netcat 处于 TCP 监听模式；-p : 指定端口号；-v：用于显示详细信息。
    > 如果想要退出监听模式，可以使用 Ctrl + C 。
    > **必须要先在服务器端监听端口，然后在客户端访问该端口，才能建立连接。**

    在安装有 Netcat 的客户端机器上，执行以下命令：

    ```bash
    nc <服务器端 IP 地址> <端口号>
    ```

    在客户端输入如上命令后，可以在客户端输入任意字符，然后按回车键，我们可以看到，客户端输入的字符均会在服务器端打印出来。测试结果如下：

    客户端：

    ![windows-tcp-client.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/windows-tcp-client.PNG)

    服务器端：

    ![windows-tcp-server.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/windows-tcp-server.PNG)

### 测试 UDP 端口连通性

1. 通过 Azure 门户选中 Windows 虚拟机 ( dillionvm ) ，在边栏选项卡中选中网络，在网络安全组中添加 UDP 5001 入站端口规则。

    ![windows-nsg-udp.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/windows-nsg-udp.PNG)

    > [!IMPORTANT]
    > 注意网络安全组规则的优先级，确保入站规则有效，并处于 “**允许**” 状态。
    > 对于 Windows 虚拟机，从 Azure 门户添加完入站端口规则之后，需要远程连接到虚拟机，在防火墙中添加相对应的 UDP 入站规则。

2. 远程连接到 Azure Windows 虚拟机，使用 cmd 命令导航至 nc.exe 所在目录，执行以下命令，实现 UDP 方式监听服务器 5001 端口。

    ```bash
    nc -lu -p <端口号> -v
    ```

    > [!NOTE]
    > -lu：指明 Netcat 处于 UDP 监听模式；-p：指定端口号；-v：用于显示详细信息。
    > 如果想要退出监听模式，可以使用 Ctrl + C 。
    > **必须要先在服务器端监听端口，然后在客户端访问该端口，才能建立连接。**

    在安装有 Netcat 的客户端机器上，执行以下命令：

    ```bash
    nc -u <服务器端 IP 地址> <端口号>
    ```

    在客户端输入如上命令后，可以在客户端输入任意字符，然后按回车键，我们可以看到，客户端输入的字符均会在服务器端打印出来。测试结果如下：

    客户端：

    ![windows-udp-client.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/windows-udp-client.PNG)

    服务器端：

    ![windows-udp-server.PNG](./media/aog-virtual-network-using-netcat-check-the-connectivity/windows-udp-server.PNG)
---
title: "如何查询虚拟机流量异常及相关安全加固建议"
description: "如何查询虚拟机流量异常及相关安全加固建议"
author: JoiGE
resourceTags: 'Virtual Machines, Linux, Windows, Traffic Exception, Security Reinforce'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: yunjia.ge
ms.date: 3/26/2018
wacn.date: 5/4/2018
---

# 如何查询虚拟机流量异常及相关安全加固建议

## 应用场景

当从 Azure 门户的 Dashboard 上发现虚拟机流量异常高，或收到了监控警报时，我们一般通过流量分析可以确定这些流量属于正常流量还是异常攻击。

那么，怎么样提前加固虚拟机的安全，对虚拟机的安全问题做防范呢？

以下提供了相关的安全加固建议。

## 如何查询虚拟机流量、快速排查是否遭受攻击

您可以直接在 [Azure 门户](https://portal.azure.cn) – “**虚拟机**” - “**Overview**” 界面看到 **Dashboard**：

![01](media/aog-virtual-machines-howto-query-abnormal-traffic-and-security-reinforce-recommendations/01.png)

在虚拟机 **Alert rules** 中，您也可以针对 Network 等对虚拟机设置报警监控。

如果您发现虚拟机的流量可能存在异常，可以通过以下几个步骤快速排查虚拟机是否遭到攻击（以下以 Linux 为例，Windows 也有对应的文件及命令）：

1. /var/log 目录下 wtmp 和 secure 日志文件可看到问题时段是否有异常 IP 登录、异常操作（如 reboot 等）。
2. 如果可以登录虚拟机（RDP 或 Console），使用 `ps -ef` 命令查看是否有异常进程，比如长期占用大量 CPU 或内存（`top` 命令可查看），或非用户所属进程。
3. 在虚拟机中查看关键文件是否有被篡改或删除的情况。
4. 使用 `history` 命令查看历史操作记录或 crontab 计划列表中是否有异常。
5. 必要的情况下，建议您删除虚拟机，将原虚拟机磁盘挂载到新部署的服务器上取出业务数据，重新部署业务。

## 平台层面安全加固建议

在 Azure Platform 层面增加安全防护：

1. 所有虚拟机的业务端口, 按照实际需求, 有针对性的开放, 如数据库端口只开放给应用虚拟机。

    由于 Linux 虚拟机是直接暴露在公网的，请尽量减少开放的端口，如果您的服务是有限范围内访问的，可以针对 Endpoint 设置 ACL，或对虚拟机或虚拟子网设置 NSG。建议将敏感的 RDP 登录设置 ACL 白名单，即只允许指定 IP 进行 RDP/SSH。

2. 添加白名单

    为虚拟机的关键端口配置 ACL，指定受信任的 IP 进行连接该服务器。<br>
    参考官方文档：[经典虚拟机添加相应端口并配置 ACL](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/virtual-network/aog-virtual-network-add-endpoint-and-acl)。

    为虚拟机所在虚拟网络添加 NSG，指定受信任的 IP 进行连接该服务器。<br>
    统一用途的虚拟机放置到同一子网, 集中化配置网络安全组；有特殊需求的虚拟机，对网络接口单独设置网络安全组。<br>
    参考官方文档：[使用门户管理 NSG](https://docs.azure.cn/zh-cn/virtual-network/virtual-network-manage-nsg-arm-portal)。

3. 如果环境允许，可以在 Azure 中配置一台堡垒机作为跳板，堡垒机通过内网 IP 到连接到关键业务的生产 VM（生产 VM 不需要开放 ssh 公共端口，Internet 无法连接），杜绝外部 SSH/RDP 端口被扫描。

4. 如果环境允许，禁止用户通过密码登录，改用 ssh key 方式来登录访问, 防止密码泄露或者被包里破解；修改所有常用用户的密码为复杂密码，且用户密码要定期更新。

5. 修改常用的 Endpoint 外部端点为非默认端点，如修改 RDP/SSH 的 3389/22 端口为高位随机端口等。

6. 定期备份虚拟机数据，防止数据一旦损坏或丢失，造成数据不可用。

## VM 层面安全加固建议 - Linux 的虚拟机

1. 安装 VM 注意事项

    安装时，请注意修改默认用户名（azureuser），并选择使用 SSH public key 方式认证。<br>
    密钥创建的详细方法可以参考官方文档：[如何在 Azure 上的 Windows 中使用 SSH 密钥](https://docs.azure.cn/zh-cn/virtual-machines/linux/ssh-from-windows)。<br>
    建议禁止 root 账号登录虚拟机，并增加密码的复杂度（大小写字母，数字，特殊字符的组合）。

2. 加固 SSH

    1. 修改 SSH 端口为其他端口（比如 51124），建议禁止 root 登录，禁止空密码登录，禁止密码登录（原始文件会被备份为/etc/ssh/sshd_config.bak）

        ```shell
        sed -i.bak -e 's/^#Port 22/Port 51124/' -e 's/^#PermitRootLogin yes/PermitRootLogin no/' -e 's/PermitRootLogin yes/PermitRootLogin no/' -e 's/^#PermitEmptyPasswords yes/PermitEmptyPasswords no/' -e 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
        ```

    2. 验证修改

        ```shell
        egrep "Port|PermitRootLogin|PermitEmptyPasswords|PasswordAuthentication" /etc/ssh/sshd_config
        ```

    3. 重启 sshd 服务使修改生效

        ```shell
        systemctl restart sshd
        ```

    4. 通过 [Azure 门户](https://portal.azure.cn)修改 NSG 或者 Endpoint 中 SSH 对应的端口为对应端口（此处为 51124），然后在 SSH 客户端中修改连接端口（51124）并重新连接。

3. 在虚拟机中安装常用杀毒软件

    ClamAV 是 Linux 平台免费开源额主流杀毒软件之一，这里简单介绍如何快速使用这个基于病毒扫描的命令行工具，排查 Linux VM 是否被攻击。

    以下示例环境为 CentOS 7，使用 yum 安装。

    1. 安装 EPEL repository，从中安装 ClamAV

        ```shell
        yum -y update
        yum -y install epel-release
        yum -y update
        yum clean all
        yum -y install clamav-server clamav-data clamav-update clamav-filesystem clamav clamav-scanner-systemd clamav-devel clamav-lib clamav-server-systemd
        ```

    2. 删除 ClamAV 的 Scanner Configuration 中 Example

        配置文件默认路径是 “/etc/clamd.d/scan.conf”，在使用这个配置文件之前，我们需要手动删除或注释配置中的 Example 内容。<br>
        删除的命令参考如下：

        ```shell
        cp /etc/clamd.d/scan.conf /etc/clamd.d/scan.conf.backup
        sed -i -e "s/^Example/#Example/" /etc/clamd.d/scan.conf
        ```

        如果该命令执行失败，请通过文本编辑的方式删除 Example 段。

    3. 配置用户

        在 “/etc/clamd.d/scan.conf” 文件中，在 User 栏中配置用户：

        ```shell
        # Run as another user 
        # Default: don't drop privileges
        User clamscan
        ```

    4. 配置 server type

        在同一个文件下拉几行，可以看到以下部分：

        ```shell
        # The daemon can work in local mode, network mode or both.
        # Due to security reasons we recommend the local mode.
        # Path to a local socket file the daemon will listen on.
        # Default: disabled (must be specified by a user)
        # LocalSocket /var/run/clamd.scan/clamd.sock
        ```
        一般我们选择使用 Local mode，将 `LocalSocket` 行的 “# ” 去掉即可。

    5. 配置 FreshClam 以便更新病毒库

        FreshClam 位于 “/etc/freshclam.conf”，用于将病毒库更新到 server 上。与 ClamAV 默认配置类似，我们需要手动删除或注释配置中的 Example 内容：

        ```shell
        cp /etc/freshclam.conf /etc/freshclam.conf.bakup
        sed -i -e "s/^Example/#Example/" /etc/freshclam.conf
        ```

        修改完毕后即可通过 `freshclam` 这条命令直接启动 Freshclam。<br>
        执行后在命令行中也可以看到目前 server 使用的病毒数据库是否已为最新版，如果不是最新版，此时也会更新。<br>
        我们建议定期执行 Freshclam 以保证所使用的病毒库保持更新，这可以通过 crontab 或者 systemd 服务来实现，crontab 示例比如：

        ```shell
        00 01,13 * * *  /usr/bin/freshclam --quiet
        ```

        上述命令会让 freshclam 每两天的早上 1 点和下午 1 点执行，更多自定义的 crontab 或 systemd 服务还请客户根据实际情况设置，这里就不多做赘述了。

    6. 启动 Scanner Service：

        执行命令 `systemctl start clamd@scan` 即可启动。<br>
        如果希望开机启动可设置为 `systemctl enableclamd@scan`。

## VM 层面安全加固建议 - Windows 的虚拟机

1. 安装 VM 及配置用户时，建议设常用用户的密码为复杂密码，且用户密码要定期更新。
2. 修改常用的 Endpoint 外部端点为非默认端点，比如：修改 RDP 的 3389 端口为高位随机端口等。
3. 建议安装常用杀毒软件及拓展。

Azure 官方文档中提供了对虚拟机批量安装并配置 Microsoft Anti-Malware 扩展的 PowerShell 脚本，该脚本适用于经典部署虚拟机和资源管理部署虚拟机。

详细配置及脚本请参考[为订阅内虚拟机批量安装并配置 Microsoft Anti-Malware 扩展](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/virtual-machines/windows/aog-virtual-machines-howto-batch-config-anti-malware )一文。
---
title: Linux 虚拟机的安全审计
description: Linux 虚拟机的安全审计
service: ''
resource: Virtual Machines Linux
author: chpaadmin
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'Virtual Machines Linux, Security, Audit'
cloudEnvironments: MoonCake

ms.service: virtual-machines-linux
wacn.topic: aog
ms.topic: article
ms.author: chpa
ms.date: 09/20/2017
wacn.date: 09/20/2017
---
# Linux 虚拟机的安全审计

## 背景介绍

对于系统管理员来说，系统出现问题是常事，出现问题，解决问题，不出现问题也要制造问题，然后解决问题。最为难也最费力的是领导的一句话：为什么会发生这个问题，以后怎么避免？如果是显性的问题，比如配置错误，性能问题，都可以很清晰的回答。但是如果是文件丢失了，权限被篡改了等问题就很不好回答，难点在于，没有问题发生的具体时间点以及现场环境。

那么，如果我们对系统的重要数据有一套安全审计机制，那么这个秋后算账就会变得有理有据了。以下，通过 Linux 的 auditd 服务来配置文件和目录的安全审计。

> [!IMPORTANT]
> 以下范例基于 Azure 平台部署的 CentOS7.3，如果您使用的是其他版本的 Linux 操作系统，请注意其中的差异性。

## 方案介绍

配置文件说明：

- `/etc/audit/auditd.conf`

    Audit 服务配置文件，可以配置日志存放路径，大小等。

- `/etc/audit/rules.d/audit.rules`

    Audit 审计规则，配置要审计的内容和规则。

- `/var/log/audit/audit.log`

    Audit 审计日志（默认），存放满足审计规则的活动记录。

文件和目录审计配置：

1. 执行命令安装 Audit 软件包：

    `# yum install audit`

2. 启动 auditd 服务：

    `# service auditd start`

3. 添加 auditd 为自启动服务：

    `# systemctl enable auditd`

4. 配置文件和目录审计规则：

    1. 编辑文件 `/etc/audit/rules.d/audit.rules`

    2. 添加如下内容：

        `-w /etc/audit/ -p wa -k test_audit_rule`

    3. 保存并退出。

        > [!NOTE]
        > `-w`: 监视
        > `-p`: 权限（`rwax`| 读·写·修改属性·执行）
        > `-k`: 关键字 （用来作为索引）

5. 重启 auditd 服务使规则生效：

    ```
    # service auditd stop
    # service auditd start
    ```

6. 查看生效的审计规则：

    `# auditctl -l`

7. 查看审计报表：

    `# aureport -k`

8. 查看审计日志：

    `# cat /var/log/audit/audit.log`

## 相关建议

1. 如果审计的内容比较多，那么日志生成会比较大，建议通过修改 `/etc/audit/auditd.conf` 中的 `log_file` 行，指定 log 保存目录，避免根目录磁盘空间问题导致系统故障。

2. 对于系统安全角度，建议针对系统命令 `/sbin:/bin:/usr/sbin:/usr/bin` 进行目录级别的写和修改权限审计，避免恶意病毒和木马对系统命令进行篡改和伪装。对系统配置目录 `/etc/` 进行目录级别的写和修改权限审计，避免配置文件的误删除或者文件内容的恶意删除修改。

## 相关文档

- [Linux 虚拟机的安全加固](aog-virtual-machines-linux-security-reinforce.md)+
- [Linux 虚拟机的 SSH 双重登录认证](aog-virtual-machines-linux-ssh-two-factor-authentication.md)
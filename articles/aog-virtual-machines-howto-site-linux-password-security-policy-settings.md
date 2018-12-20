---
title: "Linux 密码安全策略设置"
description: "Linux 密码安全策略设置"
author: Bob Sun
resourceTags: 'Virtual Machines, Linux, Password Security'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: Bob Sun
ms.date: 12/17/2018
wacn.date: 12/17/2018
---

# Linux 密码安全策略设置

## 问题描述

在 Linux 系统下如何配置以下密码策略来实现安全加固：

* 密码不能够以明文方式记录在系统中
* 管理员为用户设置初始密码，用户第一次登录必须修改初始密码
* 密码长度至少包含 8 个字符
* 密码至少由 3 个类别（数字，小写字母，大写字母，其他）的字符组成
* 过去 10 个密码不可以重复使用
* 每 90 天必须更换密码
* 5 次登录失败后账户被锁定至少 30 分钟，或者由管理员解除锁定

## 解决方法

以 Centos 7.x 为例根据以上要求对密码策略进行设置:

1. 使用 `useradd` 命令创建 user，然后使用 `passwd` 为 user 设置密码，`/etc/shadow` 中记录的 user password 则是经过加密的。

    ```bash
    [root@localhost ~]# useradd jay
    [root@localhost ~]# passwd jay
    [root@localhost ~]# getent shadow
    ```

    输出如下，其中 `$6$CnI.JR/9$vYTKkBreSvrZJUZgamae54brCMGdn.dCz8shweV5frmQqtvzqglgqzSTYknKu5CWqcbuSe/5e3vSMCGlDU2VQ.` 为经过加密的密码。

    ```bash
    jay:$6$CnI.JR/9$vYTKkBreSvrZJUZgamae54brCMGdn.dCz8shweV5frmQqtvzqglgqzSTYknKu5CWqcbuSe/5e3vSMCGlDU2VQ.:17873:0:99999:7:::
    ```

2. 使用 `passwd -e` 命令强制用户在下次尝试登录的时候修改密码。

    ```bash
    [root@localhost ~]# passwd -e jay
    ```

    输出如下：

    ```bash
    Expiring password for user jay.
    passwd: Success
    ```

    用户 jay 使用初始密码尝试登录后，被强制要求修改密码，提示如下：

    ```bash
    You are required to change your password immediately (root enforced)
    WARNING: Your password has expired.
    You must change your password now and login again!
    Changing password for user jay.
    Changing password for jay.
    (current) UNIX password:
    ```

3. 密码至少包含 8 个字符。

    在 `/etc/security/pwquality.conf` 中添加以下:

    ```bash
    minlen = 8
    dcredit = 0
    ucredit = 0
    lcredit = 0
    ocredit = 0
    ```

    用户 jay 使用 `passwd` 设置新密码，当密码长度低于 8 时，提示如下:

    ```bash
    [jay@localhost ~]$ passwd
    Changing password for user jay.
    Changing password for jay.
    (current) UNIX password:
    New password:
    BAD PASSWORD: The password is shorter than 8 characters
    New password:
    ```

4. 密码需要由 3 个类别（数字，小写字母，大写字母，其他）的字符组成。

    在 `/etc/security/pwquality.conf` 中添加以下:

    ```bash
    minclass  = 3
    ```

    用户 jay 使用 `passwd` 设置新密码，当密码字符类别小于 3 时，提示如下：

    ```bash
    [jay@localhost ~]$ passwd
    Changing password for user jay.
    Changing password for jay.
    (current) UNIX password:
    New password:
    BAD PASSWORD: The password contains less than 3 character classes
    New password:
    ```

5. 记录过去使用过的 10 个密码（不可重复使用）。

    在 `/etc/pam.d/system-auth` 和 `/etc/pam.d/password-auth` 中添加以下（在 `pam_pwquality.so` 这一行之后）：

    ```bash
    password    requisite     pam_pwhistory.so remember=10 use_authtok
    ```

6. 密码需要每 90 天更新一次。

    使用 `chage-M` 命令设置：

    ```bash
    [root@localhost ~]# chage -M 90 jay
    ```

    使用 `change -l` 命令查看：

    ```bash
    [root@localhost ~]# chage -l jay
    Last password change                                    : password must be changed
    Password expires                                        : password must be changed
    Password inactive                                       : password must be changed
    Account expires                                         : never
    Minimum number of days between password change          : 0
    Maximum number of days between password change          : 90
    Number of days of warning before password expires       : 7
    ```

7. 5 次登录尝试失败后账户将被锁定至少 30 分钟，或者由管理员将其解锁。

    在 `/etc/pam.d/system-auth` 和 `/etc/pam.d/password-auth` 中分别添加如下所示第 2 行和第 4 行：

    ```bash
    1 auth        required      pam_env.so
    2 auth        required      pam_faillock.so preauth silent audit deny=5 unlock_time=1800
    3 auth        sufficient    pam_unix.so nullok try_first_pass
    4 auth        [default=die] pam_faillock.so authfail audit deny=5 unlock_time=1800
    5 auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
    6 auth        required      pam_deny.so
    ```

    在以上 2 个文件的 account 部分，添加如下内容：

    ```bash
    account     required      pam_faillock.so
    ```

## 参考文档

* [Chapter 4. Hardening Your System with Tools and Services](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/chap-hardening_your_system_with_tools_and_services)

* [Set a password policy in Red Hat Enterprise Linux 7](https://access.redhat.com/solutions/2808101)
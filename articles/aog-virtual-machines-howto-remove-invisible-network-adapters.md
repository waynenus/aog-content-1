---
title: "如何移除隐藏网络适配器"
description: "如何移除隐藏网络适配器"
author: garfield0
resourceTags: 'Virtual Machines, Network Adapters'
ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: tonglei.wang
ms.date: 06/30/2018
wacn.date: 06/30/2018
---

# 如何移除隐藏网络适配器

在特定场景下我们发现 Windows Server 虚拟机中会存在多个隐藏网络适配器，这通常是由对虚拟网卡的相关操作触发的。例如，已删除的网卡仍以隐藏设备的形式出现在设备管理器中等。若虚拟机中存在大量隐藏网络适配器，这可能会影响虚拟机的网络连通性，从而导致挂载 Azure File Share 失败等问题。在这种情况下，客户可以通过如下操作手动删除多余的隐藏网络适配器。

## 存在少量隐藏网络适配器

可以根据以下方法进行删除:

1. 在 CMD 窗口中输入以下命令：

    ```bash
    set devmgr_show_nonpresent_devices=1
    start devmgmt.msc
    ```

2. 在设备管理器中勾选显示隐藏设备，删除隐藏的网络适配器：

    ![01](media/aog-virtual-machines-howto-remove-invisible-network-adapters/01.png)

## 存在大量隐藏网络适配器

可以使用如下方法进行批量删除：

1. 在虚拟机中安装 [Windows Driever Kit(WDK)](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/download-the-wdk#download-iconimagesdownload-installpng-step-2-install-wdk-for-windows-10-version-1803) 来获取 devcon.exe。

    ![02](media/aog-virtual-machines-howto-remove-invisible-network-adapters/02.png)

2. 下载、解压并导入 [Device Management Powershell Cmdlets](https://blogs.technet.microsoft.com/wincat/2012/09/06/device-management-powershell-cmdlets-sample-an-introduction/)。

    ![03](media/aog-virtual-machines-howto-remove-invisible-network-adapters/03.png)

    在下载的压缩文件属性中点击 **Unblock** 并 **应用**，然后解压该文件。

    ![01-added](media/aog-virtual-machines-howto-remove-invisible-network-adapters/01-added.png)

    然后在 PowerShell 中以管理员身份运行下列命令：

    ```PowerShell
    Set-Location C:\DeviceManagement\Release\
    Import-Module .\DeviceManagement.psd1 -verbose
    ```

    结果如下：

    ![04](media/aog-virtual-machines-howto-remove-invisible-network-adapters/04.png)

3. 使用 `Get-Device` 命令列出隐藏的网络适配器。

    ```PowerShell
    Get-Device -ControlOptions DIGCF_ALLCLASSES | Sort-Object -Property Name | Where-Object {($_.IsPresent -eq $false) -and ($_.Name -like “Microsoft Hyper-V Network Adapter*”) } | ft Name, DriverVersion, DriverProvider, IsPresent, HasProblem, InstanceId -AutoSize
    ```

4. 获取隐藏的网络适配器。

    ```PowerShell
    $hiddenHypVNics = Get-Device -ControlOptions DIGCF_ALLCLASSES | Sort-Object -Property Name | Where-Object {($_.IsPresent -eq $false) -and ($_.Name -like “Microsoft Hyper-V Network Adapter*”) }
    ```

5. 在 Powershell 中设定路径到 devcon.exe 下。

    ![05](media/aog-virtual-machines-howto-remove-invisible-network-adapters/05.png)

6. 使用以下循环语句来删除隐藏网络设备：

    ```bash
    Set-Location "C:\Program Files (x86)\Windows Kits\10\Tools\x64"
    ForEach ($hiddenNic In $hiddenHypVNics) { Update the $deviceid = “@" + $hiddenNic.InstanceId; .\devcon.exe /r remove $deviceid }
    ```

7. 最终隐藏网络设备删除完毕。

    ![06](media/aog-virtual-machines-howto-remove-invisible-network-adapters/06.png)

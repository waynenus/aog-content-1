---
title: 通过 Powershell 来调整 ARM 模式下虚拟机的尺寸
description: 通过 Powershell 来调整 ARM 模式下虚拟机的尺寸
services: Virtual Machines
documentationCenter: ''
author: forester123
manager: ''
editor: ''
tags: ''

ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: v-johch
ms.date: 06/23/2017
wacn.date: 06/23/2017
---

# 通过 Powershell 来调整 ARM 模式下虚拟机的尺寸

## 需求描述

在部署完 ARM 模式的虚拟机以后，可以通过 PowerShell 命令来调整虚拟机的尺寸，以下是通过 PowerShell 命令来调整 ARM 模式的虚拟机尺寸。

> [!NOTE]
> 本文只限于 ARM 模式下的虚拟机，经典模式的虚拟机不适用。

## 操作步骤

1. 首先，我们需要使用下面的命令定义需要调整尺寸的虚拟机的基本信息：

        #定义资源组名称
        $ResourceGroupName = <资源组名称>
        #定义需要调整尺寸的虚拟机名称
        $VMName = <虚拟机名称>
        #定义虚拟机的新尺寸，这里以Standard_A1为例，详细尺寸列表见附录部分
        $NewVMSize = "Standard_A1"

2. 定义完成后，使用下面的命令将虚拟机调整到新的尺寸：

        #获取虚拟机对象
        $vm = Get-AzureRmVM -ResourceGroupName <资源组名称> -Name <虚拟机名称>
        #设定新尺寸
        $vm.HardwareProfile.vmSize = $NewVMSize
        #将虚拟机调整为新尺寸
        Update-AzureRmVM -ResourceGroupName $ResourceGroupName -VM $vm

3. 待命令执行完成后，可以看到虚拟机的尺寸已调整完毕：

    ![](media/aog-virtual-machines-arm-scale-size-with-powershell/portal.png)

## 附录

| 尺寸名称    | 尺寸变量名             |
|------------ | --------------------- |
| A0基本      | Basic_A0              |
| A1基本      | Basic_A1              |
| A2基本      | Basic_A2              |
| A3基本      | Basic_A3              |
| A4基本      | Basic_A4              |
| A0标准      | Standard_A0           |
| A1标准      | Standard_A1           |
| A1_v2标准   | Standard_A1_v2        |
| A2标准      | Standard_A2           |
| A2_v2标准   | Standard_A2_v2        |
| A2m_v2标准  | Standard_A2m_v2       |
| A3标准      | Standard_A3           |
| A4标准      | Standard_A4           |
| A4_v2标准   | Standard_A4_v2        |
| A4m_v2标准  | Standard_A4m_v2       |
| A5标准      | Standard_A5           |
| A6标准      | Standard_A6           |
| A7标准      | Standard_A7           |
| A8_v2标准   | Standard_A8_v2        |
| A8m_v2标准  | Standard_A8m_v2       |
| D1标准      | Standard_D1           |
| D1_v2标准   | Standard_D1_v2        |
| D11标准     | Standard_D11          |
| D11_v2标准  | Standard_D11_v2       |
| D11_v2促销  | Standard_D11_v2_Promo |
| D12标准     | Standard_D12          |
| D12_v2标准  | Standard_D12_v2       |
| D12_v2促销  | Standard_D12_v2_Promo |
| D13标准     | Standard_D13          |
| D13_v2标准  | Standard_D13_v2       |
| D13_v2_促销 | Standard_D13_v2_Promo |
| D14标准     | Standard_D14          |
| D14_v2标准  | Standard_D14_v2       |
| D14_v2_促销 | Standard_D14_v2_Promo |
| D15_v2标准  | Standard_D15_v2       |
| D2标准      | Standard_D2           |
| D2_v2标准   | Standard_D2_v2        |
| D2_v2_促销  | Standard_D2_v2_Promo  |
| D3标准      | Standard_D3           |
| D3_v2标准   | Standard_D3_v2        |
| D3_v2_促销  | Standard_D3_v2_Promo  |
| D4标准      | Standard_D4           |
| D4_v2标准   | Standard_D4_v2        |
| D4_v2_促销  | Standard_D4_v2_Promo  |
| D5_v2标准   | Standard_D5_v2        |
| D5_v2_促销  | Standard_D5_v2_Promo  |
| DS1标准     | Standard_DS1          |
| DS1_v2标准  | Standard_DS1_v2       |
| DS11标准    | Standard_DS11         |
| DS11_v2标准 | Standard_DS11_v2      |
| DS11_v2_促销| Standard_DS11_v2_Promo|
| DS12标准    | Standard_DS12         |
| DS12_v2标准 | Standard_DS12_v2      |
| DS12_v2_促销| Standard_DS12_v2_Promo|
| DS13标准    | Standard_DS13         |
| DS13_v2标准 | Standard_DS13_v2      |
| DS13_v2_促销| Standard_DS13_v2_Promo|
| DS14标准    | Standard_DS14         |
| DS14_v2标准 | Standard_DS14_v2      |
| DS14_v2_促销| Standard_DS14_v2_Promo|
| DS15_v2标准 | Standard_DS15_v2      |
| DS2标准     | Standard_DS2          |
| DS2_v2标准  | Standard_DS2_v2       |
| DS2_v2_促销 | Standard_DS2_v2_Promo |
| DS3标准     | Standard_DS3          |
| DS3_v2标准  | Standard_DS3_v2       |
| DS3_v2_促销 | Standard_DS3_v2_Promo |
| DS4标准     | Standard_DS4          |
| DS4_v2标准  | Standard_DS4_v2       |
| DS4_v2_促销 | Standard_DS4_v2_Promo |
| DS5_v2标准  | Standard_DS5_v2       |
| DS5_v2_促销 | Standard_DS5_v2_Promo |
| F1标准      | Standard_F1           |
| F16标准     | Standard_F16          |
| F16s标准    | Standard_F16s         |
| F1s标准     | Standard_F1s          |
| F2标准      | Standard_F2           |
| F2s标准     | Standard_F2s          |
| F4标准      | Standard_F4           |
| F4s标准     | Standard_F4s          |
| F8标准      | Standard_F8           |
| F8s标准     | Standard_F8s          |

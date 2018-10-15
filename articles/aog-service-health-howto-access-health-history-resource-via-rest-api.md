---
title: '如何通过 Azure 服务运行状况的 Rest API 获取历史资源运行状况'
description: '如何通过 Azure 服务运行状况的 Rest API 获取历史资源运行状况'
author: EmiliaHuang
resourceTags: 'Service Health, Rest API'
ms.service: service-health
wacn.topic: aog
ms.topic: article
ms.author: Xuzhou.Huang
ms.date: 10/15/2018
wacn.date: 10/15/2018
---

# 如何通过 Azure 服务运行状况的 Rest API 获取历史资源运行状况

## 问题描述

当需要获取较多资源的历史运行状况时，频繁地通过网页点击获取信息，可能会引发过多的访问请求，超过 [Azure 订阅服务限制](https://docs.azure.cn/zh-cn/azure-resource-manager/resource-manager-request-limits)。

## 解决方法

建议通过调用 Azure 服务运行状况提供的 Rest API 获取所需的资源运行状况。

通过 ARMClient 调用 Azure API 的主要操作流程为：

1. [安装 Chocolatey](https://chocolatey.org/install#installing-chocolatey)
2. [安装 ARMClient](https://github.com/projectkudu/ARMClient)
3. [调用 Azure 服务运行状况的 Rest API](https://blogs.msdn.microsoft.com/premier_developer/2017/04/06/how-to-use-azure-resource-health-api-to-gain-visibility-into-the-health-of-a-vm-web-app-or-sql-database/)

以获取订阅下所有虚拟机的历史资源运行状况为例，在完成上述步骤 1 和 2 之后，在 PowerShell 运行以下脚本以获取目标运行状况结果，输出为 .csv 文件：

```powershell
#custom object
function RHstatus()
{
    param ($resourcegroup,$vmname,$occuredtime,$availState,$summary,$reportedtime,$reasontype,$rootCauseAttributionTime,$resolutionETA)
    $RHstatus = new-object PSObject
    $RHstatus | add-member -type NoteProperty -Name ResourceGroup -Value $resourcegroup
    $RHstatus | add-member -type NoteProperty -Name VMName -Value $vmname
    $RHstatus | add-member -type NoteProperty -Name OccuredTime -Value $occuredtime
    $RHstatus | add-member -type NoteProperty -Name availabilityState -Value $availState
    $RHstatus | add-member -type NoteProperty -Name Summary -Value $summary
    $RHstatus | add-member -type NoteProperty -Name ReportedTime -Value $reportedtime
    $RHstatus | add-member -type NoteProperty -Name reasontype -Value $reasontype
    $RHstatus | add-member -type NoteProperty -Name rootCauseAttributionTime -Value $rootCauseAttributionTime
    $RHstatus | add-member -type NoteProperty -Name resolutionETA -Value $resolutionETA

    return $RHstatus
}

$exportCSV = New-Object -TypeName System.Collections.ArrayList

#login mooncake
ARMClient Login Mooncake
Login-AzureRmAccount -EnvironmentName AzureChinaCloud

#filter all VMs
$VMresources = Get-AzureRmResource | Where-Object { $_.'ResourceType' -eq "Microsoft.Compute/virtualMachines" }

foreach ($resource in $VMresources){

    #get resource health
    $uri = $resource.ResourceId + "/providers/Microsoft.ResourceHealth/availabilityStatuses/?api-version=2015-01-01"
    $result = ARMClient GET $uri

    $resultJSON = $result | ConvertFrom-Json

    foreach ($status in $resultJSON.value){

        $vmInfo = $status.id.Split("/",[StringSplitOptions]::RemoveEmptyEntries)
        $vmResourceHealth = RHstatus -resourcegroup $vmInfo[3] -vmname $vmInfo[7]`
        -occuredtime $status.properties.occuredtime`
        -availState $status.properties.availabilityState -summary $status.properties.summary`
        -reportedtime $status.properties.reportedtime`
        -reasontype $status.properties.reasonType -rootCauseAttributionTime $status.properties.rootCauseAttributionTime`
        -resolutionETA $status.properties.resolutionETA
  
        $exportCSV.Add($vmResourceHealth)
    }
}

$exportCSV | export-csv D:\result.csv
```
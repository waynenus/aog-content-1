---
title: 如何将同一云服务下的虚拟机从经典部署模型迁移到 Azure Resource Manager
description: 如何将同一云服务下的虚拟机从经典部署模型迁移到 Azure Resource Manager
service: ''
resource: Virtual Machines
author: garfield0
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'Virtual Machines, ASM, ARM, Cloud Service, Migration'
cloudEnvironments: MoonCake

ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: tonglei.wang
ms.date: 08/31/2017
wacn.date: 08/31/2017
---
# 如何将同一云服务下的虚拟机从经典部署模型迁移到 Azure Resource Manager

## 适用场景

用户希望将特定云服务下的所有虚拟机从经典部署模型(以下简称：ASM)迁移到 Azure Resource Manager(以下简称：ARM)。

> [!NOTE]
> 如果云服务下使用 VNET 也希望将虚拟机从 ASM 模式迁移到 ARM 模式，您可以参考这篇文章：[如何将同一个 VNET 下的虚拟机从 ASM 迁移到 ARM 上](aog-virtual-machines-howto-migrate-from-asm-to-arm-within-same-vnet)

## 解决方案


1.	首先，我们登陆到需要迁移的虚拟机所在的订阅下，注册迁移服务：

    ```PowerShell
    #登陆到需要迁移的虚拟机所在的订阅下
    PS C:\windows\system32> Login-AzureRmAccount –Environment AzureChinaCloud

    Environment           : AzureChinaCloud
    Account               : XXX@mcpod.partner.onmschina.cn
    TenantId              : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    SubscriptionId        : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    SubscriptionName      : <订阅名称>
    CurrentStorageAccount :

    #如果您需要迁移某个特定订阅下的虚拟机，需要手动进行指定
    PS C:\windows\system32> Select-AzureRmSubscription –SubscriptionName "<订阅名称>"

    #注册迁移服务
    PS C:\windows\system32> Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

    #迁移服务注册一般需要 5 分钟左右，您可以通过下述命令查看完成情况，
    PS C:\windows\system32> Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

    ProviderNamespace : Microsoft.ClassicInfrastructureMigrate
    RegistrationState : Registered
    ResourceTypes     : {classicInfrastructureResources}
    Locations         : {China North, China East}

    #最后登陆到经典模式中需要迁移的虚拟机所在订阅下
    PS C:\windows\system32> Add-AzureAccount -Environment AzureChinaCloud

    Id                                Type Subscriptions                        Tenants
    --                                ---- -------------                        -------
    XXX@mcpod.partner.onmschina.cn User xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx { xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx }

    #如果您需要迁移某个特定订阅下的虚拟机，需要手动进行指定
    Select-AzureSubscription –SubscriptionName "<订阅名称>"
    ```

    > [!NOTE]
    > 注册迁移服务为一次性操作，注册完成后以后迁移时无需再次注册，但是如果您在未注册前尝试迁移，会收到报错说该订阅未注册迁移服务。

2.	检查 ARM 模式下您订阅里的配额，确保需要迁移的虚拟机有足够的配额可以使用：

    ```PowerShell
    #您可以根据虚拟机所在的区域选择 China North 或者 China East
    PS C:\windows\system32> Get-AzureRmVMUsage -Location "China East"

    Name                             Current Value Limit  Unit
    ----                             ------------- -----  ----
    Availability Sets                            2  2000 Count
    Total Regional Cores                        39   100 Count
    Virtual Machines                            17 10000 Count
    Virtual Machine Scale Sets                   1  2000 Count
    Standard Dv2 Family Cores                    4   100 Count
    Standard FS Family Cores                    16   100 Count
    Standard A0-A7 Family Cores                 11   100 Count
    Standard D Family Cores                      1   100 Count
    Standard Av2 Family Cores                    2   100 Count
    Standard DSv2 Family Cores                   4   100 Count
    Standard DS Family Cores                     1   100 Count
    Basic A Family Cores                         0   100 Count
    Standard A8-A11 Family Cores                 0   100 Count
    Standard G Family Cores                      0   100 Count
    Standard GS Family Cores                     0   100 Count
    Standard F Family Cores                      0   100 Count
    Standard NV Family Cores                     0     0 Count
    Standard NC Family Cores                     0     0 Count
    Standard H Family Cores                      0     0 Count
    Standard LS Family Cores                     0   100 Count
    Standard Dv2 Promo Family Cores              0   100 Count
    Standard DSv2 Promo Family Cores             0   100 Count
    Standard MS Family Cores                     0     0 Count
    Standard Dv3 Family Cores                    0   100 Count
    Standard DSv3 Family Cores                   0   100 Count
    Standard Ev3 Family Cores                    0   100 Count
    Standard ESv3 Family Cores                   0   100 Count
    Standard Storage Managed Disks               0 10000 Count
    Premium Storage Managed Disks                0 10000 Count
    ```

3.	将同一个云服务下的虚拟机从 ASM 迁移到 ARM 上:

    ```PowerShell
    #查询云服务并找出您想要迁移的云服务，此处以 1vnet2cs02 为例
    PS C:\windows\system32> Get-AzureService|ft Servicename

    ServiceName
    -----------
    1vent2cs04
    1vnet2cs01
    1vnet2cs02
    1vnet2cs03

    #获取部署名称
    PS C:\windows\system32> $serviceName = "1vnet2cs02"
    PS C:\windows\system32> $deployment = Get-AzureDeployment -ServiceName $serviceName
    PS C:\windows\system32> $deploymentName = $deployment.DeploymentName
    Result             : Validation Passed with warnings. Please see ValidationMessages object for a list of resources
                        that will be migrated and additional detail on the warnings.
    ValidationMessages : {test4as, test01, test01, test01...} 
    ```

    - 选择 1：将 VM 迁移到一个新建的虚拟网络中:

        ```PowerShell
        #验证云服务
        PS C:\windows\system32> Move-AzureService -Validate -ServiceName $serviceName ` -DeploymentName $deploymentName -CreateNewVirtualNetwork

        OperationId        : 119a8709-4160-4122-859d-a1bc2dc8c603
        Result             : Validation Passed. Please see ValidationMessages object for a list of resources that will be migrated.
        ValidationMessages : {cs03, cs03, cs03}

        #上述验证通过后接下来执行迁移准备操作
        PS C:\windows\system32> Move-AzureService -Prepare -ServiceName $serviceName `
        >>     -DeploymentName $deploymentName -CreateNewVirtualNetwork

        OperationDescription OperationId                          OperationStatus
        -------------------- -----------                          ---------------
        Move-AzureService    7a6931ef-6f42-47dc-8d99-0929fbdc5658 Succeeded

        #如果迁移准备步骤操作成功，那么可以执行下述命令来生效迁移操作
        PS C:\windows\system32> Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName

        OperationDescription OperationId                          OperationStatus
        -------------------- -----------                          ---------------
        Move-AzureService    e88c11c0-21d4-46c0-abb0-700c324b21ba Succeeded
        ```

    - 选择 2：将 VM 迁移到一个已有的虚拟网络中:
 
        ```PowerShell
        #准备已有虚拟网络的相关信息
        PS C:\windows\system32> $existingVnetRGName = "<虚拟网络所在的资源组名称>"
        PS C:\windows\system32> $vnetName = "<虚拟网络名称>"
        PS C:\windows\system32> $subnetName = "<子网名称>"

        #验证云服务
        PS C:\windows\system32> Move-AzureService -Validate -ServiceName $serviceName `
        >>     -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName

        #上述验证通过后接下来执行迁移准备操作
        PS C:\windows\system32> Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
        >>     -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
        >>     -VirtualNetworkName $vnetName -SubnetName $subnetName

        OperationDescription OperationId                          OperationStatus
        -------------------- -----------                          ---------------
        Move-AzureService    9c9e01a6-183e-4884-b51d-04f25112934e Succeeded

        #如果迁移准备步骤操作成功，那么可以执行下述命令来生效迁移操作
        PS C:\windows\system32> Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName

        OperationDescription OperationId                          OperationStatus
        -------------------- -----------                          ---------------
        Move-AzureService    c803cc0d-1446-4922-bcb1-af1330828c69 Succeeded
        ```

4.	迁移虚拟机所在的存储账号:

    ```PowerShell
    #检查存储账号下是否有未被迁移的虚拟机的 VHD 存在
    PS C:\windows\system32> $storageAccountName ="<存储账号名称>"
    PS C:\windows\system32> Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Select-Object -ExpandProperty AttachedTo -Property `
    >>   DiskName | Format-List -Property RoleName, DiskName

    #检查存储账号下是否有已分离的虚拟机磁盘，如有需要进行删除，如果您仍然需要这些磁盘可以将其复制到其他存储账号中
    PS C:\windows\system32> Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Where-Object -Property AttachedTo -EQ $null | Format-List
    -Property DiskName

    DiskName : disktest

    #删除磁盘
    PS C:\windows\system32> Remove-AzureDisk -DiskName "disktest"

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Remove-AzureDisk     9661659d-3bcf-4e6e-bdc4-cce7c8d7a1e0 Succeeded

    #删除存储账号中 OS 盘和数据盘中的虚拟机镜像
    PS C:\windows\system32> Get-AzureVmImage | Where-Object { $_.OSDiskConfiguration.MediaLink -ne $null -and $_.OSDiskConfiguration.MediaLink.Host.Contains($storageA
    ccountName)`
    >>                            } | Select-Object -Property ImageName, ImageLabel

    ImageName                   ImageLabel
    ---------                   ----------
    captureTest-20160902-436096

    PS C:\windows\system32> Get-AzureVmImage | Where-Object {$_.DataDiskConfigurations -ne $null `
    >>                                     -and ($_.DataDiskConfigurations | Where-Object {$_.MediaLink -ne $null -and $_.MediaLink.Host.Contains($storageAccountName)
    }).Count -gt 0 `
    >>                                    } | Select-Object -Property ImageName, ImageLabel

    PS C:\windows\system32> Remove-AzureVMImage -ImageName "captureTest-20160902-436096"

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Remove-AzureVMImage  b8f1c164-147a-4d98-9805-432dcd3b7105 Succeeded


    #验证存储账号是否符合迁移条件
    PS C:\windows\system32> Move-AzureStorageAccount -Validate -StorageAccountName $storageAccountName

    OperationId        : 68f3501e-f965-4732-aae5-a1da5b58103b
    Result             : Validation Passed. Please see ValidationMessages object for a list of resources that will be migrated.
    ValidationMessages : {<存储账号名称>}

    PS C:\windows\system32> $val=Move-AzureStorageAccount -Validate -StorageAccountName $storageAccountName
    PS C:\windows\system32> $val.ValidationMessages

    ResourceType       : Storage
    ResourceName       : <存储账号名称>
    Category           : Information
    Message            : Storage Account tcportalvhdsgrnnb3k173zr is eligible for migration.
    VirtualMachineName :

    #上述验证通过后接下来执行迁移准备操作
    PS C:\windows\system32> Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName

    OperationDescription     OperationId                          OperationStatus
    --------------------     -----------                          ---------------
    Move-AzureStorageAccount 4086a517-a14a-4360-97aa-2714543dd345 Succeeded

    #如果在迁移准备操作中出现了报错，或者您想取消本次迁移操作，可以使用下面命令进行取消

    PS C:\windows\system32> Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName

    OperationDescription     OperationId                          OperationStatus
    --------------------     -----------                          ---------------
    Move-AzureStorageAccount 8473i405-b3fg-7942-345h-8236hdy63i2f Succeeded

    #如果迁移准备步骤操作成功，那么可以执行下述命令来生效迁移操作

    PS C:\windows\system32> Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName

    OperationDescription     OperationId                          OperationStatus
    --------------------     -----------                          ---------------
    Move-AzureStorageAccount cf4e80b3-8e26-4d65-96f2-98dda2266d49 Succeeded
    ```
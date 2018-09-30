---
title: '如何使用自定义脚本升级 Azure 自动化服务 PowerShell 模块'
description: '如何使用自定义脚本升级 Azure 自动化服务 PowerShell 模块'
author: sscchh2001
resourceTags: 'Automation, PowerShell, Script'
ms.service: automation
wacn.topic: aog
ms.topic: article
ms.author: chesh
ms.date: 09/30/2018
wacn.date: 09/30/2018
---

# 如何使用自定义脚本升级 Azure 自动化服务 PowerShell 模块

众所周知，Azure 自动化服务支持 PowerShell ，默认安装低版本的 Azure PowerShell 模块，一般需要升级到最新版本使用。

由于新版 Azure PowerShell 模块和命令调用了最新版本的 Azure REST API，而新版 REST API 在某些 Azure 特权云（如 Azure US 政府云）中暂时不受支持，导致用户使用 Azure 自动化服务的升级 Azure PowerShell 模块功能后会遇到自动化脚本报错的情况。Azure 中国云暂时未遇到此问题。

为了避免用户在 Azure 特权云中遇到升级模块导致脚本运行失败的情况，产品组需要临时将升级模块功能关闭，并会在将来进行功能优化后重新开放此功能。在升级 Azure 模块功能关闭期间，如果用户需要批量升级 Azure PowerShell 模块，请根据以下步骤，创建自定义脚本完成升级模块。

> [!NOTE]
> 更新模块可能会造成现有脚本与最新模块不兼容。建议在测试环境中使用最新模块对脚本完成可行性测试后，再对生产环境的模块进行更新。

脚本配置步骤：

1. 在自动化账号中创建 `Runbook Update-ModulesInAutomationToLatestVersion`，类型为 PowerShell。
2. Runbook PowerShell 代码如下：

    ```powershell
    <#PSScriptInfo
    .VERSION 1.02
    .GUID fa658952-8f94-45ac-9c94-f5fe23d0fcf9
    .AUTHOR Automation Team
    .COMPANYNAME Microsoft
    .COPYRIGHT
    .TAGS AzureAutomation OMS Module Utility
    .LICENSEURI
    .PROJECTURI https://github.com/azureautomation/runbooks/blob/master/Utility/Update-ModulesInAutomationToLatestVersion.ps1

    .ICONURI
    .EXTERNALMODULEDEPENDENCIES
    .REQUIREDSCRIPTS
    .EXTERNALSCRIPTDEPENDENCIES
    .RELEASENOTES
    #>

    #Requires -Module AzureRM.Profile
    #Requires -Module AzureRM.Automation

    <#
    .SYNOPSIS 
        This Azure/OMS Automation runbook imports the latest version on PowerShell Gallery of all modules in an 
        Automation account.If a new module to import is specified, it will import that module from the PowerShell Gallery
        after all other modules are updated from the gallery.

    .DESCRIPTION
        This Azure/OMS Automation runbook imports the latest version on PowerShell Gallery of all modules in an 
        Automation account. By connecting the runbook to an Automation schedule, you can ensure all modules in
        your Automation account stay up to date.
        If a new module to import is specified, it will import that module from the PowerShell Gallery
        after all other modules are updated from the gallery.

    .PARAMETER ResourceGroupName
        Optional. The name of the Azure Resource Group containing the Automation account to update all modules for.
        If a resource group is not specified, then it will use the current one for the automation account
        if it is run from the automation service

    .PARAMETER AutomationAccountName
        Optional. The name of the Automation account to update all modules for.
        If an automation account is not specified, then it will use the current one for the automation account
        if it is run from the automation service

    .PARAMETER NewModuleName
        Optional. The name of a module in the PowerShell gallery to import after all existing modules are updated

    .EXAMPLE
        Update-ModulesInAutomationToLatestVersion -ResourceGroupName "MyResourceGroup" -AutomationAccountName "MyAutomationAccount" -NewModuleName "AzureRM.Batch"

    .NOTES
        AUTHOR: Automation Team & CSS
        LASTEDIT: September, 2018  
    #>

    param(
        [Parameter(Mandatory=$false)]
        [String] $ResourceGroupName = 'RGName',

        [Parameter(Mandatory=$false)]
        [String] $AutomationAccountName = 'AccountName',

        [Parameter(Mandatory=$false)]
        [String] $NewModuleName
    )
    $ErrorActionPreference = 'Continue'
    $ModulesImported = @()

    function _doImport {
        param(
            [Parameter(Mandatory=$true)]
            [String] $ResourceGroupName,

            [Parameter(Mandatory=$true)]
            [String] $AutomationAccountName,

            [Parameter(Mandatory=$true)]
            [String] $ModuleName,

            # if not specified latest version will be imported
            [Parameter(Mandatory=$false)]
            [String] $ModuleVersion
        )

        $Url = "https://www.powershellgallery.com/api/v2/Search()?`$filter=IsLatestVersion&searchTerm=%27$ModuleName%27&targetFramework=%27%27&includePrerelease=false&`$skip=0&`$top=40"

        $SearchResult = Invoke-RestMethod -Method Get -Uri $Url -UseBasicParsing

        if($SearchResult.Length -and $SearchResult.Length -gt 1) {
            $SearchResult = $SearchResult | Where-Object -FilterScript {
                return $_.properties.title -eq $ModuleName
            }
        }

        if(!$SearchResult) {
            Write-Warning "Could not find module '$ModuleName' on PowerShell Gallery. This may be a module you imported from a different location"
        }
        else {
            $ModuleName = $SearchResult.properties.title # get correct casing for the module name
            $PackageDetails = Invoke-RestMethod -Method Get -UseBasicParsing -Uri $SearchResult.id 

            if(!$ModuleVersion) {
                # get latest version
                $ModuleVersion = $PackageDetails.entry.properties.version
            }

            $ModuleContentUrl = "https://www.powershellgallery.com/api/v2/package/$ModuleName/$ModuleVersion"

            # Make sure module dependencies are imported
            $Dependencies = $PackageDetails.entry.properties.dependencies

            if($Dependencies -and $Dependencies.Length -gt 0) {
                $Dependencies = $Dependencies.Split("|")

                # parse depencencies, which are in the format: module1name:module1version:|module2name:module2version:
                $Dependencies | ForEach-Object {
                    if($_ -and $_.Length -gt 0) {
                        $Parts = $_.Split(":")
                        $DependencyName = $Parts[0]
                        $DependencyVersion = $Parts[1] -replace '[^0-9.]'

                        # check if we already imported this dependency module during execution of this script
                        if(!$ModulesImported.Contains($DependencyName)) {
                            $AutomationModule = Get-AzureRmAutomationModule `
                                -ResourceGroupName $ResourceGroupName `
                                -AutomationAccountName $AutomationAccountName `
                                -Name $DependencyName `
                                -ErrorAction SilentlyContinue

                            # check if Automation account already contains this dependency module of the right version
                            if((!$AutomationModule) -or [System.version]$AutomationModule.Version -lt [System.version]$DependencyVersion) {
                                Write-Output "Importing dependency module $DependencyName of version $DependencyVersion first."

                                # this dependency module has not been imported, import it first
                                _doImport `
                                    -ResourceGroupName $ResourceGroupName `
                                    -AutomationAccountName $AutomationAccountName `
                                    -ModuleName $DependencyName `
                                    -ModuleVersion $DependencyVersion
                                $ModulesImported += $DependencyName
                            }
                        }
                    }
                }
            }

            # Find the actual blob storage location of the module
            do {
                $ActualUrl = $ModuleContentUrl
                $ModuleContentUrl = (Invoke-WebRequest -Uri $ModuleContentUrl -MaximumRedirection 0 -UseBasicParsing -ErrorAction Ignore).Headers.Location 
            } while(!$ModuleContentUrl.Contains(".nupkg"))

            $ActualUrl = $ModuleContentUrl

            Write-Output "Importing $ModuleName module of version $ModuleVersion to Automation"

            $AutomationModule = New-AzureRmAutomationModule `
                -ResourceGroupName $ResourceGroupName `
                -AutomationAccountName $AutomationAccountName `
                -Name $ModuleName `
                -ContentLink $ActualUrl

            while(
                (!([string]::IsNullOrEmpty($AutomationModule))) -and
                $AutomationModule.ProvisioningState -ne "Created" -and
                $AutomationModule.ProvisioningState -ne "Succeeded" -and
                $AutomationModule.ProvisioningState -ne "Failed"
            )
            {
                Write-Verbose -Message "Polling for module import completion"
                Start-Sleep -Seconds 10
                $AutomationModule = $AutomationModule | Get-AzureRmAutomationModule
            }

            if($AutomationModule.ProvisioningState -eq "Failed") {
                Write-Error "Importing $ModuleName module to Automation failed, retrying."
                _doImport `
                    -ResourceGroupName $ResourceGroupName `
                    -AutomationAccountName $AutomationAccountName `
                    -ModuleName $ModuleName `
                    -ModuleVersion $ModuleVersion
            }
            else {
                Write-Output "Importing $ModuleName module to Automation succeeded."
            }
        }
    }

    try {
        $RunAsConnection = Get-AutomationConnection -Name "AzureRunAsConnection"

        Write-Output ("Logging in to Azure...")

        $env = Get-AzureRmEnvironment -Name 'AzureChinaCloud'
        Add-AzureRmAccount `
            -ServicePrincipal `
            -TenantId $RunAsConnection.TenantId `
            -ApplicationId $RunAsConnection.ApplicationId `
            -CertificateThumbprint $RunAsConnection.CertificateThumbprint `
            -Environment $env

        Select-AzureRmSubscription -SubscriptionId $RunAsConnection.SubscriptionID  | Write-Verbose

        # Find the automation account or resource group is not specified
        if  (([string]::IsNullOrEmpty($ResourceGroupName)) -or ([string]::IsNullOrEmpty($AutomationAccountName)))
        {
        Write-Output ("Finding the ResourceGroup and AutomationAccount that this job is running in ...")
        if ([string]::IsNullOrEmpty($PSPrivateMetadata.JobId.Guid))
        {
                    throw "This is not running from the automation service. Please specify ResourceGroupName and AutomationAccountName as parameters"
        }
        $AutomationResource = Get-AzureRmResource -ResourceType Microsoft.Automation/AutomationAccounts

            foreach ($Automation in $AutomationResource)
            {
                $Job = Get-AzureRmAutomationJob -ResourceGroupName $Automation.ResourceGroupName -AutomationAccountName $Automation.Name -Id $PSPrivateMetadata.JobId.Guid -ErrorAction SilentlyContinue
                if (!([string]::IsNullOrEmpty($Job)))
                {
                        $ResourceGroupName = $Job.ResourceGroupName
                        $AutomationAccountName = $Job.AutomationAccountName
                        break;
                }
            }
        }
    }
    catch {
        if(!$RunAsConnection) {
            throw "Connection AzureRunAsConnection not found. Please create one"
        }
        else {
            throw $_.Exception
        }
    }

    $Modules = Get-AzureRmAutomationModule `
        -ResourceGroupName $ResourceGroupName `
        -AutomationAccountName $AutomationAccountName

    foreach($Module in $Modules) {

        $Module = Get-AzureRmAutomationModule `
            -ResourceGroupName $ResourceGroupName `
            -AutomationAccountName $AutomationAccountName `
            -Name $Module.Name

        $ModuleName = $Module.Name
        $ModuleVersionInAutomation = $Module.Version

        Write-Output "Checking if module '$ModuleName' is up to date in your automation account"

        $Url = "https://www.powershellgallery.com/api/v2/Search()?`$filter=IsLatestVersion&searchTerm=%27$ModuleName%27&targetFramework=%27%27&includePrerelease=false&`$skip=0&`$top=40"

        $SearchResult = Invoke-RestMethod -Method Get -Uri $Url -UseBasicParsing

        if($SearchResult.Length -and $SearchResult.Length -gt 1) {
            $SearchResult = $SearchResult | Where-Object -FilterScript {
                return $_.properties.title -eq $ModuleName
            }
        }

        if(!$SearchResult) {
            Write-Warning "Could not find module '$ModuleName' on PowerShell Gallery. This may be a module you imported from a different location"
        }
        else {
            $PackageDetails = Invoke-RestMethod -Method Get -UseBasicParsing -Uri $SearchResult.id 
            $LatestModuleVersionOnPSGallery = $PackageDetails.entry.properties.version

            if($ModuleVersionInAutomation -ne $LatestModuleVersionOnPSGallery) {
                Write-Output "Module '$ModuleName' is not up to date. Latest version on PS Gallery is '$LatestModuleVersionOnPSGallery' but this automation account has version '$ModuleVersionInAutomation'"
                Write-Output "Importing latest version of '$ModuleName' into your automation account"

                _doImport `
                    -ResourceGroupName $ResourceGroupName `
                    -AutomationAccountName $AutomationAccountName `
                    -ModuleName $ModuleName
            }
            else {
                Write-Output "Module '$ModuleName' is up to date."
            }
    }
    }

    # Import module if specified
    if (!([string]::IsNullOrEmpty($NewModuleName)))
    {
        # Check if module exists in the gallery
        $Url = "https://www.powershellgallery.com/api/v2/Search()?`$filter=IsLatestVersion&searchTerm=%27$NewModuleName%27&targetFramework=%27%27&includePrerelease=false&`$skip=0&`$top=40"

        $SearchResult = Invoke-RestMethod -Method Get -Uri $Url -UseBasicParsing

        if($SearchResult.Length -and $SearchResult.Length -gt 1) {
            $SearchResult = $SearchResult | Where-Object -FilterScript {
                return $_.properties.title -eq $NewModuleName
            }
        }

        if(!$SearchResult) {
            throw "Could not find module '$NewModuleName' on PowerShell Gallery."
        }  

        if ($NewModuleName -notin $Modules.Name)
        {
            Write-Output "Importing latest version of '$NewModuleName' into your automation account"

            _doImport `
                -ResourceGroupName $ResourceGroupName `
                -AutomationAccountName $AutomationAccountName `
                -ModuleName $NewModuleName
        }
        else
        {
            Write-Output ("Module $NewModuleName is already in the automation account")
        }
    }
    ```

3. 可以修改 Parameter 中的输入参数为自动化账号的实际资源组和账号名，否则需要在运行 Runbook 时提供对应参数。如果输入 NewModuleName，在更新完其他模块后，脚本会尝试从 PowerShell Gallery 中查找输入的模块信息并自动下载。
4. 如果在更新过程中遇到报错，请参考文档[如何手动更新 Azure 自动化服务的 PowerShell 模块](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/automation/aog-automation-powershell-module-update)，手动更新 AzureRm.Profile 和 AzureRm.Automation 模块，再重新运行脚本。
5. 如果在更新后遇到模块与脚本的兼容性问题，也请参考步骤 4 中的文档，手动更新或倒回有兼容行问题的 PowerShell 模块。

## 参考文档

- [如何手动更新 Azure 自动化服务的 PowerShell 模块](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/automation/aog-automation-powershell-module-update)

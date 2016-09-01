#自定义脚本使用示例


自定义脚本扩展程序可以从Azure存储自动下载脚本和文件，并在VM启动PowerShell脚本。 本例，我们将演示如何使用自定义脚本来完成Windows组件的安装

## windows组件的安装

1.	创建安装组件的脚本(如update.ps1)：

		Install-WindowsFeature -Name Web-Server, Web-Mgmt-Tools

2.	将脚本上传到blob storage并设置其访问权限为public read access：
 
	![](media/aog-virtual-machine-custom-script/upload-script.png)

	![](media/aog-virtual-machine-custom-script/container-permission.png)
 
3.	创建虚拟机： 

	![](media/aog-virtual-machine-custom-script/create-vm.png)
        
4.	点击”FROM STORAGE“，选择所要使用的的powershell脚本

	![](media/aog-virtual-machine-custom-script/choose-from-blob.png)

5.	虚拟机创建完成后，将会自动安装所需的组件

	![](media/aog-virtual-machine-custom-script/result.png)


经过以上几步，我们可以确保每一台创建的VM都安装IIS组件





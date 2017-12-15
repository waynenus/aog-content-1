---
title: Python 基于固定 IP 来命名 ARM 虚拟机的实现
description: 如何通过 Python 批量创建基于固定 IP 来命名的 ARM 虚拟机
service: ''
resource: virtualmachines
author: hello-azure
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: Virtual Machines, ARM, Python, Static IP
cloudEnvironments: MoonCake

ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 04/18/2017
wacn.date: 04/18/2017
---

# Python 基于固定 IP 来命名 ARM 虚拟机的实现

## **问题描述**

希望通过 Python 批量创建 ARM 虚拟机，并且在虚拟机命名时加入固定 IP 信息，方便管理维护。

![vm](./media/aog-virtual-machines-qa-arm-batch-create-set-static-ip-with-python/vm.png)

## **问题分析**

在创建 ARM 虚拟机之前，先创建固定 IP，然后获取固定 IP 地址，创建虚拟机时通过该 IP 地址格式化虚拟机名称。然后将固定 IP 配置到网络接口，基于该网络接口配置创建 ARM 虚拟机。

## **解决方法**

### **模块安装**

本文在 Windows Python 环境下进行测试，环境及模块依赖如下：

- 官网下载 msi 安装包，管理员命令行执行以下安装脚本

    ```Bash
    msiexec /package python-xxx.msi
    ```

- 使用 PIP 安装 Azure（需要 pip 9+ 支持，Python 2.7 环境已内置 pip 9+ 版本，不需更新）

    ```Bash
    pip install azure
    ```

- 查看已安装的模块

    ```Bash
    pip freeze
    ```

### **代码实现**

```Python
from azure import *
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.network import NetworkManagementClient
from azure.mgmt.resource import ResourceManagementClient
from azure.common.credentials import UserPassCredentials
from azure.mgmt.compute.models import *
from msrest.serialization import *

credentials = UserPassCredentials(
    "订阅账户",
    "账户密码",
    china=True
)

resource_client = ResourceManagementClient(
credentials,
'订阅 ID',
base_url = 'https://management.chinacloudapi.cn'
)

resource_client.providers.register('Microsoft.Compute')
resource_client.providers.register('Microsoft.Network')

compute_client = ComputeManagementClient(
credentials,
'订阅 ID',
base_url = 'https://management.chinacloudapi.cn'
)

network_client = NetworkManagementClient(
credentials,
    '订阅 ID',
base_url = 'https://management.chinacloudapi.cn'
)

# Create Public IP
# result = network_client.public_ip_addresses.create_or_update(
#     'geogroup',     #group_name
#     'geo-ip-01',        #ip_name
#     PublicIPAddress(
#         location='China North',
#         public_ip_allocation_method=IPAllocationMethod.static,
#         idle_timeout_in_minutes=4,
#         ),
#     )
# result.wait()

public_ip_addresses = 'public_ip_name'
group_name = ''public_ip_group'

result = network_client.public_ip_addresses.get(group_name,public_ip_addresses)
print result.__dict__.items()
print result.ip_address
print result.ip_address.replace(".","-")

storageName = "storage account name"
vmName = "geovm-"+result.ip_address.replace(".","-")
print vmName

location = "chinanorth"
print location

os_profile = OSProfile(
    computer_name= vmName,
    admin_username='username',
    admin_password='password,
)
print os_profile

hardware_profile = HardwareProfile(
    vm_size=VirtualMachineSizeTypes.standard_a0
)
print hardware_profile

storage_profile = StorageProfile(
    os_disk=OSDisk(
        caching=CachingTypes.none,
        create_option=DiskCreateOptionTypes.from_image,
        name=vmName,
        vhd=VirtualHardDisk(
uri='https://'+storageName+'.blob.core.chinacloudapi.cn/vhds/'+vmName+'.vhd',
        ),
    ),
)

storage_profile.image_reference = ImageReference(
    publisher='Canonical',
    offer='UbuntuServer',
    sku='16.04.0-LTS',
    version='latest'
)
print storage_profile

network_profile = NetworkProfile(
    network_interfaces=[
        NetworkInterfaceReference(
            id="在新门户，网络接口-属性中获取资源 ID,该网络接口需要配置固定 IP",
        ),
    ],
)
print network_profile

params_create = VirtualMachine(
    location=location,
    os_profile=os_profile,
    hardware_profile=hardware_profile,
    network_profile=network_profile,
    storage_profile=storage_profile,
)
print params_create

result_create = compute_client.virtual_machines.create_or_update(
    group_name,
    vmName,
    params_create
)

result_create.wait()
print 'ok'
```
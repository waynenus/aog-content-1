---
title: Python 获取 Token 进行验证
description: Python 获取 Token 进行验证
service: ''
resource: multiple
author: taroyutao
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'Python, Token, Azure AD'
cloudEnvironments: MoonCake

ms.service: multiple
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 08/31/2017
wacn.date: 08/31/2017
---
# Python 获取 Token 进行验证

在使用 Python SDK 操作 Azure 资源时，首先需要获取认证的 Token，然后才能对资源的做进一步的操作。本文分别介绍了基于 Azure AD 和 Azure 账户名和密码两种获取 Token 的方式，实现 managerClient 对象的创建，进而操作 Azure 资源。

## Azure AD 方式

```
import azure.mgmt.resource
import azure.mgmt.network
from msrestazure.azure_active_directory import AADTokenCredentials, ServicePrincipalCredentials

GROUP_NAME = 'yutaopythonresourcetest1'
REGION = 'chinanorth' #option value: chinaeast, chinanorth

client_id = '16a94c84-c6ba-4f19-aa48-3353f8ffe18e'
client_secret = '123456'
tenant_id = 'b388b808-0ec9-4a09-a414-a7cbbd8b711b'
subscription_id = 'e0fbea86-6cf2-4b2d-81e2-9c59f4f96b33'

credentials = ServicePrincipalCredentials(client_id,client_secret,tenant=tenant_id,china='true')

resource_client = azure.mgmt.resource.ResourceManagementClient(credentials, subscription_id,base_url='https://management.chinacloudapi.cn')

resource_group_params = {'location':REGION}

# create a resource group
result = resource_client.resource_groups.create_or_update(
    GROUP_NAME,
    resource_group_params
)
```

## 账户名--密码方式

```
from azure.common.credentials import UserPassCredentials
from azure.mgmt.resource import ResourceManagementClient

credentials = UserPassCredentials('test@microsoftinternal.partner.onmschina.cn', '123456',china = 'true')
subscription_id = 'e0fbea86-6cf2-4b2d-81e2-9c59f4f96b33'

resource_client = ResourceManagementClient(credentials, subscription_id,base_url='https://management.chinacloudapi.cn')

resource_group_name = 'my_resource_group'

resource_client.resource_groups.create_or_update(
    resource_group_name,
    {
        'location':'chinanorth'
    }
)
```

## 更多信息参考链接

- [Azure SDK for Python](http://azure-sdk-for-python.readthedocs.io/en/latest/)
- [Azure SDK for Python - GitHub](https://github.com/Azure/azure-sdk-for-python)

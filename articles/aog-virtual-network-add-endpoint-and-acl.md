---
title: "经典虚拟机添加相应端口并配置 ACL"
description: "本页介绍如何为经典虚拟机添加相应端口并配置 ACL。"
author: JamborYao
resourceTags: ''
ms.service: virtual-network
wacn.topic: aog
ms.topic: article
ms.author: Jianbo.Yao
ms.date: 01/19/2018
wacn.date: 08/01/2016
---

# 经典虚拟机添加相应端口并配置 ACL

**本文包含以下内容**
- [相关概念](#concept)
- [详细步骤](#detail)
- [相关参考资料](#resource)

## <a id="concept"></a>相关概念

添加端口时我们需要输入公用端口和私有端口，下面是它们的相关概念：
- **公用端口**：Azure 负载均衡器使用公用端口侦听从 Internet 传入的虚拟机流量。
- **私有端口**：虚拟机使用私有端口侦听通常发送到虚拟机上运行的应用程序或服务的传入流量。<br>
    例如，配置数据库服务器，使得我们可以通过 Internet 连接到 SQL Server 数据库引擎，我们需要配置 1433 端口，假如我们设置公用端口为 57500，私有端口为 1433，在使用 SQL Server Management Studio 时我们需要指定连接地址为 **mycloudservice.chinacloudapp.cn:57500**，但是真正是通过私有端口 1433 访问的数据库服务器。
- **ACL**：我们可以创建、管理和删除允许或拒绝通过 Access Control 列表( ACL )访问虚拟机的端点的规则。<br>
    同样，如果我们为了保证数据库的安全，我们可以以 CIDR 格式来指定 IP 地址范围，这样可以使得只有在允许的 IP 范围的用户来访问，可以有效的避免数据的泄露。

## <a id="detail"></a>详细步骤

### 创建端口

1. 使用 [Azure门户](https://portal.azure.cn) 添加终结点：

    ![](./media/aog-virtual-network-add-endpoint-and-acl/endpoint_list.PNG)

2. 添加独立终结点：

    ![](./media/aog-virtual-network-add-endpoint-and-acl/add_endpoint.PNG) 

3. 添加的相应端口已在列表中：

    ![](./media/aog-virtual-network-add-endpoint-and-acl/endpoint_result_list.PNG)

### 管理 ACL

1. 选择需要修改 ACL 的终结点：

    ![](./media/aog-virtual-network-add-endpoint-and-acl/select_endpoint.PNG)

2. 添加允许或拒绝的远程子网：

    ![](./media/aog-virtual-network-add-endpoint-and-acl/edit_acl.PNG)

## <a id="resource"></a>相关参考资料

- [如何设置虚拟机的终结点](https://docs.azure.cn/virtual-machines/windows/classic/setup-endpoints)
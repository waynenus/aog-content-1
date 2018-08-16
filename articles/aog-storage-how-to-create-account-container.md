---
title: '如何创建存储账户及容器'
description: '本页介绍如何创建存储账户及容器。'
author: JamborYao
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: Jianbo.Yao
ms.date: 07/10/2018
wacn.date: 12/14/2015
---

# 如何创建存储账户及容器

## <a id="concept"></a>相关概念

**资源组**：一个容器，用于保存 Azure 解决方案的相关资源。 资源组可以包含解决方案的所有资源，也可以只包含以组的形式进行管理的资源。 根据对组织有利的原则，决定如何将资源分配到资源组。 请参阅 [资源组](https://docs.azure.cn/azure-resource-manager/resource-group-overview#resource-groups)。

**位置/地缘组**：中国版 Azure 有上海和北京中国东部，中国东部 2 ，中国北部和中国北部 2 四个数据中心，我们可以选择其中的一个作为我们存储的位置。

### 复制

- **本地冗余(LRS)**：本地冗余将在单个区域中的单个设施内复制三次，可以保护我们的数据免受普通硬件故障的损害。
- **地域冗余(GRS)**：地域冗余将在主区域中复制三次，然后再辅助区域内复制三次，一共维护六个副本，当主区域中发生故障时，Azure 存储空间将故障转移到辅助区域。
- **读取访问地域冗余(RA-GRS)**：读取访问地域冗余存储将你的数据复制到一个辅助地理位置，同时提供对你在辅助位置中的数据的读访问权限。读取访问地域冗余存储允许你从主位置或辅助位置访问数据，以防其中一个位置不可用。

更多详细请阅读[这篇文章](https://docs.azure.cn/storage/common/storage-redundancy)

### 存储和容器的命名规则

Azure 存储的命名规则可以阅读[这篇文章](https://docs.microsoft.com/zh-cn/azure/architecture/best-practices/naming-conventions)，文章包含了 Azure 存储中所涉及的 Blob、Table、Queue 的命名规则。

### 容器的公共访问级别

- **专用(无匿名访问)**：不允许匿名用户读取该容器中的 Blob
- **Blob(仅匿名读取访问 blob )**：匿名用户可以读取 Blob，无法列出容器下的所有 Blob
- **容器(匿名读取访问容器和 blob )**：匿名用户可以读取该容器中的 Blob，并且可以读取该容器，可以列出容器下的所有 Blob

## <a id="operation"></a>详细步骤

1. 登录 [Azure 门户](Https://portal.azure.cn), 依次点击存储账户-添加，在创建存储帐户页面输入存储账户信息，点击创建按钮

    ![](./media/aog-storage-how-to-create-account-container/create-account.PNG)

2. 创建成功后，在存储帐户列表中选中创建的存储账户，查看账户详细信息
    
    ![](./media/aog-storage-how-to-create-account-container/enter-storage.PNG)

    > [!NOTE]
    > 存储账号创建完成后我们可以通过最下方的“管理访问密钥”按钮获取或者更新密钥。

3.  在存储账户左侧导航栏中，选中容器，添加容器，输入容器名称，选择公共访问级别

    ![](./media/aog-storage-how-to-create-account-container/create-container.PNG)

4. 容器创建完成

    ![](./media/aog-storage-how-to-create-account-container/container-dashboard.PNG)

## <a id="tool"></a>推荐工具

Azure Store Explorer 是一个不错的工具，可以让我们很直观的管理我们的存储，请点击[此处](https://docs.azure.cn/zh-cn/vs-azure-tools-storage-manage-with-storage-explorer?toc=%2fstorage%2fblobs%2ftoc.json)下载。
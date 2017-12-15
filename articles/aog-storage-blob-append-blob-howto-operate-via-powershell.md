---
title: "如何使用 PowerShell 对 Azure 存储的追加 Blob (Append Blob) 进行读写"
description: "如何使用 PowerShell 对 Azure 存储的追加 Blob (Append Blob) 进行读写"
author: sscchh2001
resourceTags: 'Storage, Append Blob, PowerShell'
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: chesh
ms.date: 12/14/2017
wacn.date: 12/14/2017
---

# 如何使用 PowerShell 对 Azure 存储的追加 Blob (Append Blob) 进行读写

众所周知，Azure 存储服务提供了三种类型的 Blob，块 Blob，追加 Blob 和页 Blob（磁盘）。用户需要在创建 Blob 时指定 Blob 类型。一旦 Blob 被创建，其类型不能被改变，并且只能通过使用适合于该 Blob 类型的操作来更新它，即，将块或块列表写入块 Blob，将块追加到追加 Blob，以及将页面写到页 Blob。

块 Blob 是用于存储大量非结构化对象数据（例如文本或二进制数据）的服务；磁盘是作为页 Blob 存储在 Azure 存储中的固定格式 VHD，由虚拟机用来存储持久性数据。
追加 Blob 由块组成，并针对追加操作进行了优化。当您修改追加块时，块仅通过追加块操作添加到块的末尾。与块 Blob 不同，追加 Blob 不支持更新或删除现有的块，且不公开其块 ID。与块 Blob 相同的是，追加 Blob 中的每个块可以是不同的大小，最大为 4MB，最多可以包含 50,000 个块。因此追加 Blob 的最大尺寸略大于 195GB（4MB X 50,000 块）。

追加 Blob 提供了直接将数据写入 Blob 的功能，特别是使用 [Azure 自动化](https://docs.azure.cn/zh-cn/automation/) 执行脚本产生日志时，可以将内存变量中的日志数据直接写到追加 Blob 中。相比之下，块 Blob 需要生成本地文件后上传至存储，而页 Blob 只能通过 Azure 虚拟机加载后使用，追加 Blob 在此类情景下的优势可见一斑。

在操作追加 Blob 时，我们一般使用 Azure 存储 SDK 来执行，如 [.NET](https://docs.azure.cn/zh-cn/storage/blobs/storage-dotnet-how-to-use-blobs) 或 [Node.js](https://docs.azure.cn/zh-cn/storage/blobs/storage-nodejs-how-to-use-blob-storage)。使用 PowerShell，也可以通过调用.NET 对象的方法来达到相同的效果。

设置存储账号上下文并查找 Blob 容器：

```PowerShell
$Resourcegroup = "<yourResourceGroupName>"
$StorageAccountName = "<yourStorageAccountName>"

$StorageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $Resourcegroup -Name $StorageAccountName
$StorageContext = New-AzureStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey
$container = Get-AzureStorageContainer -Name $logcontainer -Context $StorageContext
```

配置追加 Blob reference：

```PowerShell
$appBlob = $container.CloudBlobContainer.GetAppendBlobReference("appendlog.log")
if (!$appBlob.Exists()) {
    $appBlob.CreateOrReplace();
}
```

> [!NOTE]
> 文件名所对应的文件可以不存在，也可以是已有的追加 Blob 文件。如果是已有的块 Blob 文件或页面 Blob，会出现 Blob 类型错误的报错。

将 String 写入追加 Blob：

```PowerShell
$String = "Hello World!"
$appBlob.AppendText([String]::Format("`n{0}",$String))
```

将字符串数组循环写入追加 Blob，并在不同字符串中插入换行：

```PowerShell
$Array = @("Hello World1!", "Hello World2!")
for ($i = 0; $i -lt $array.Count; $i++ ){
    $appBlob.AppendText([String]::Format("`n{0}",$array[$i]))
}
```

输出追加 Blob 的内容：

```PowerShell
$appBlob.DownloadText()
```

## 相关参考

- [通过 .NET 开始使用 Azure Blob 存储](https://docs.azure.cn/zh-cn/storage/blobs/storage-dotnet-how-to-use-blobs#writing-to-an-append-blob)
- [TechNet 博客 Azure Blob now has append!!!](https://blogs.technet.microsoft.com/thbrown/2015/08/26/azure-blob-now-has-append/)
- [Understanding Block Blobs, Append Blobs, and Page Blobs](https://docs.microsoft.com/zh-cn/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) 

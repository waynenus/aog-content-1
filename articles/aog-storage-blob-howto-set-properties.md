---
title: "如何对 Azure Storage Blob 属性进行设置"
description: "如何对 Azure Storage Blob 属性进行设置"
author: taroyutao
resourceTags: 'Storage, Blob, Properties'
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/8/2018
wacn.date: 11/8/2018
---

# 如何对 Azure Storage Blob 属性进行设置

## 概述

在使用 SDK 做 Blob 对象属性的获取或设置时，如果只是直接使用 get 或 set 方法，是无法成功获取或设置 blob 对象的属性。主要是因为在获取对象时，对象的属性默认并未被填充到对象，这就需要执行额外的方法将对象的属性填充给对象；而在设置 Blob 对象属性时，程序默认只是保存到了本地，并未提交到 Server 端，所以需要执行额外的方法将修改提交到 Server 端。

下面分别给出 JAVA 和 C# 的 SDK 获取、设置 Blob 对象属性的示例。

### JAVA 示例代码

```java
//get content type
blob2.downloadAttributes();
System.out.println(blob2.getProperties().getContentType());

//set content type
String contentType = "image"; //image/jpeg
blob2.getProperties().setContentType(contentType);
blob2.uploadProperties();
```

### C# 示例代码

```csharp
//get property
CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
blockBlob.FetchAttributes();
Console.WriteLine("ContentType: " + blockBlob.Properties.ContentType);

//set property
blockBlob.Properties.ContentType = "property test";
blockBlob.SetProperties();
```

### SDK参考

- [azure-storage-java](https://github.com/Azure/azure-storage-java)
- [azure-storage-net](https://github.com/Azure/azure-storage-net)
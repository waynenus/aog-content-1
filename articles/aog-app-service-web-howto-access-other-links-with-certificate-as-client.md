---
title: "Web 应用作为客户端携带证书访问其他链接办法"
description: "Web 应用作为客户端携带证书访问其他链接办法"
author: 123Jun321
resourceTags: 'App Service Web, Certificate'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-ciwu
ms.date: 12/20/2018
wacn.date: 12/20/2018
---

# Web 应用作为客户端携带证书访问其他链接办法

## 问题描述

有些时候应用服务可能需要作为客户端去访问其他自定义签名的 https 链接，或者在访问一些接口时需要携带证书去访问，关于公共证书 cer 文件您可以上传到 Azure 上面，然后通过更改应用程序设置加入环境变量 WEBSITE_LOAD_CERTIFICATES（值为 *）在代码中读取。

但是如果是 p12 这种在门户上面不支持添加的证书格式就需要您自己上传到磁盘下，再用代码读取了，这篇文档就是解决这个问题(也适用于 cer 证书)。

## 解决方案

您可以使用下面的代码去读取证书，然后将携带证书的请求发送出去：

```csharp
public async Task<HttpResponseMessage> PostAsync()
{
    try {
        string url = "您的应用程序要访问的链接";

        var handler = new WebRequestHandler();
        handler.ClientCertificateOptions = ClientCertificateOption.Manual;
                X509Certificate2 certificates = new X509Certificate2(@"您的证书路径", "证书密码", X509KeyStorageFlags.MachineKeySet);

        handler.ClientCertificates.Add(certificates);
        handler.ClientCertificates.Add(certificates);
        handler.ServerCertificateValidationCallback = delegate (object sender, System.Security.Cryptography.X509Certificates.X509Certificate certificate,
                                      System.Security.Cryptography.X509Certificates.X509Chain chain,
                                      System.Net.Security.SslPolicyErrors sslPolicyErrors) {return true; // **** Always accept
};

        using (var hc = new HttpClient(handler)) {
            var httpContent = new StringContent("证书密码", Encoding.UTF8, "application/xml");
            var response = await hc.PostAsync(url, httpContent);
            return response;
        }
    }
    catch (Exception e) {
    Console.WriteLine(e);
    return null;
    }
}
```

需要注意的是 'X509Certificate2 certificates = new X509Certificate2(@"您的证书路径", "证书密码", X509KeyStorageFlags.MachineKeySet);'。

此行代码中的 *X509KeyStorageFlags* 这个参数必须设置为 *X509KeyStorageFlags.MachineKeySet*。

>[!Note]
>以上代码只是为测试使用，并非必须写成上述格式，但是关于 *X509KeyStorageFlags* 的值必须为 *MachineKeySet*。
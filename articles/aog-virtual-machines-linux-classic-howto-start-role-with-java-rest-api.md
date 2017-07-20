---
title: Java 调用 Rest api 设置经典 Linux 虚拟机的实例启停
description: 如何使用 Java 调用 Rest api 设置经典 Linux 虚拟机的实例启停
services: Virtual Machines
documentationCenter: ''
author: xuhua4317
manager: ''
editor: ''
tags: 'Virtual Machines, Linux, ASM, Java, Rest API'

ms.service: virtual-machines
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 07/17/2017
wacn.date: 07/17/2017
---

# Java 调用 Rest api 设置经典 Linux 虚拟机的实例启停

## 现象描述

通过 Rest API 设置经典 Linux 虚拟机实例的启停。在调用此 API 时需要通过 Azure Active Directory(下文简称 AAD) 获取 Token，但是因为中国版 Azure 中通过 AAD 的 Application 获取到的 Token 无法操作经典 API，所以需要调用 PowerShell 的 Client ID 和管理员的用户名密码获取 Token 才可以。

## 前提条件

创建一台 Linux 经典虚拟机。

## 示例代码

``` Java
import java.io.DataOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.security.KeyManagementException;
import java.security.*;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.UnrecoverableKeyException;
import java.security.cert.X509Certificate;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import javax.net.ssl.*;
import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;
import org.codehaus.jackson.map.ObjectMapper;

public void GetToken() { 
    ExecutorService service = Executors.newFixedThreadPool(1);
    AuthenticationContext ac = new AuthenticationContext("https://login.chinacloudapi.cn/tenantID", true, service);	
    Future<AuthenticationResult> future = ac.acquireToken("https://management.core.chinacloudapi.cn/", 	"1950a258-227b-4e31-a9cf-717495945fc2", "username", "password", null);
	AuthenticationResult result = future.get();
	String token = result.getAccessToken();
	rest(token);
}

public static void rest(String accessToken) throws IOException{
    URL url = new URL(String.format("https://management.core.chinacloudapi.cn/subID/services/hostedservices/{hostedservices}/deployments/{deployments}/roleinstances/{roleinstancesName}/Operations"));
    HttpsURLConnection conn = (HttpsURLConnection)url.openConnection();
    conn.setRequestProperty("x-ms-version", "2013-06-01");
    conn.setRequestProperty("Authorization", "Bearer " + accessToken);
    conn.setRequestProperty("Content-Type", "application/xml");

    //StartRole
    String roleInstance = new String("<StartRoleOperation xmlns=\"http://schemas.microsoft.com/windowsazure\" xmlns:i=\"http://www.w3.org/2001/XMLSchema-instance\">\n" + "<OperationType>StartRoleOperation</OperationType>\n" +		                        "</StartRoleOperation>");
    //ShutdownRole             
    String roleins = new String( "<ShutdownRoleOperation xmlns=\"http://schemas.microsoft.com/windowsazure\" xmlns:i=\"http://www.w3.org/2001/XMLSchema-instance\">" +"<OperationType>ShutdownRoleOperation</OperationType>" +	                                         "<PostShutdownAction>StoppedDeallocated</PostShutdownAction>" +		                                       "</ShutdownRoleOperation>");
    byte[] data = roleInstance.getBytes();
    conn.setDoOutput(true);
    conn.setRequestMethod("POST");
    if (data != null)
    {
        DataOutputStream requestStream = new DataOutputStream(conn.getOutputStream());
        requestStream.write(data);
        requestStream.flush();
        requestStream.close();
    }
    String mess =  conn.getResponseMessage();
    int code = conn.getResponseCode(); 
    InputStream input = conn.getErrorStream();
    if (input == null)
    input = conn.getInputStream();
    String response = null;
    try (Scanner scanner = new Scanner(input)) {
        scanner.useDelimiter("\\Z");
        response = scanner.next();
        scanner.close();
        input.close();
    }
}
```

## 参考链接

[Virtual Machines (classic) REST API - Start Role](https://msdn.microsoft.com/en-us/library/jj157189.aspx)
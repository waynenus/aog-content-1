---
title: 如何预热 Azure Web 应用
description: 如何预热 Azure Web 应用
service: ''
resource: CDN
author: hello-azure
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'CDN, REST API'
cloudEnvironments: MoonCake

ms.service: cdn
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 09/22/2017
wacn.date: 09/22/2017
---
# CDN REST 接口调用及说明

最近，有些客户在尝试使用 REST 接口访问 CDN 服务期间，遇到了一些问题，导致调用失败。比如，在请求时不清楚 Authorization 的构建、或者是构建 Authorization 时仍出现各种异常，我们将对这些问题进行解答。

需要注意的是，目前 CDN 同样提供了两套 REST 接口，一套是 ASM（经典模式），另外一套是 ARM（资源管理器），API 接口地址参考：

- ASM CDN：https://docs.azure.cn/zh-cn/cdn/cdn-api 
- ARM CDN：https://docs.microsoft.com/zh-cn/rest/api/cdn/

如果对于 ARM CDN 的使用有更多问题，可以联系通过 400-085-0319 联系开发技术支持团队。本节我们重点介绍以下 3 个 ASM CDN REST 接口的使用：

- [创建节点 (JAVA 示例)](https://docs.azure.cn/zh-cn/cdn/cdn-api-create-endpoint)
- [节点管理 - 获取节点信息 (JAVA 示例)](https://docs.azure.cn/zh-cn/cdn/cdn-api-get-endpoint)
- [缓存刷新 (C#示例)](https://docs.azure.cn/zh-cn/cdn/cdn-api-add-purge)

## 创建节点调用详解

- Endpoint：

    `POST https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints?apiVersion=1.0`

- 请求参数：

    | 参数名称 | 参数值 |
    | ------- | ------ |
    | x-azurecdn-request-date | 必填。符合 yyyy-MM-dd hh:mm:ss 格式的 UTC 当前请求时间 |
    | Authorization | 必填。授权头请参考 [CDN API 签名机制](https://docs.azure.cn/zh-cn/cdn/cdn-api-signature) |
    | content-type |	必填。application/json |
    | Body | 比填。参考 API 连接中的解释 |

这里我们主要解释 Authorization 的生成，[CDN API 签名机制](https://docs.azure.cn/zh-cn/cdn/cdn-api-signature)中提供了不同语言生成认证头的代码示例，这里我们以 Java 作为说明，方法定义如下：

```Java
public static String calculateAuthorizationHeader(
	String requestURL, 
	String requestTime, 
	String keyID, 
	String keyValue, 
	String httpMethod) throws Exception {
…
}
```

- requestURL

    即填充后的 Endpoint 值，subscriptionId 需要设置为实际值，参考此[示例值](https://restapi.cdn.azure.cn/subscriptions/e0fbea86-6cf2-4b2d-81e2-9c59f4f96bcb/endpoints?apiVersion=1.0)。
    

- requestTime

    该参数的设置与请求中 `x-azurecdn-request-date` 的值一致，符合 `yyyy-MM-dd hh:mm:ss` 格式的 UTC 时间即可。参数值示例：

    `2017-09-01 09:00:00`

- keyID 与 keyValue

    通过 Azure 门户登录 CDN 节点管理门户，在 “**安全管理**” - “**秘钥管理**” 中获取秘钥，需要注意的是，秘钥的读写权限需要与请求的 URL 方法权限吻合。比如 “**Post**” 或 “**Put**” 类的请求，需要 “**写**” 权限的秘钥，对于 “**Get**” 请求则只需要 “**读**” 权限的秘钥即可。

    ![portal](media/aog-cdn-rest-api-interface-and-introduction/portal.png)

- httpMethod

    请求方法类型，参数值示例：`POST`。

## 示例代码

下载链接：[Azure CDN Demo](https://github.com/wacn/AOG-CodeSample/tree/master/cdn/Java/azure-cdn-demo-master)

## 测试用例

```Java
String requestURL = "https://restapi.cdn.azure.cn/subscriptions/e0fbea86-6cf2-4b2d-81e2-9c59f4f96bcb/endpoints?apiVersion=1.0";
 
String requestTimestamp = TimeUtil.getUTCTime();
String keyID = "cc65a046-2a32-4f7d-ab22-9ae49507d719";
String keyValue = "<cdn-key>";
String httpMethod = "POST";

CdnOperation cOperation = new CdnOperation();
String authorization = cOperation.calculateAuthorizationHeader(
            requestURL, 
            requestTimestamp,
            keyID, 
            keyValue, 
            httpMethod);
            
// 创建节点
String requestBody = Files.toString(
new File("D:\\workspace\\java\\azure-cdn-
demo\\src\\test\\java\\geo\\azure\\cdn\\request_body.json",
            Charsets.UTF_8);

String result = cOperation.postRequest(
            requestURL,
            authorization, 
            requestBody,
            requestTimestamp);
System.out.println(result);

2017-7-26 5:37:18
AzureCDN cc65a046-2a32-4f7d-ab22-9ae49507d719:CB82B4555573CCE40FF2BAB1C5AA256C35CFDA5175F4B3E7C5A4905A98E08BF9
{"EndpointId":"8137ecc4-71c4-11e7-8259-0017fa00a611","Settings":{"CustomDomain":"file.azurecloudapi.cn","Host":"file.azurecloudapi.cn","Origin":{"Addresses":["devstoragerm.blob.core.chinacloudapi.cn"]},"ICP":"?ICP?16013159?","ServiceType":"Download","AllowedScheme":0},"Status":{"Enabled":false,"IcpVerifyStatus":"IcpVerifying","LifetimeStatus":"Creating","CNameConfigured":false,"FreeTrialExpired":false,"TimeLastUpdated":"2017-07-26T05:37:47.4718609+00:00"}}
```

## 获取节点信息调用详解

- Endpoint

    `GET https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}?apiVersion=1.0`

- 请求参数

    | 参数名称 | 参数值 |
    | ------- | ------ |
    | x-azurecdn-request-date | 必填。符合yyyy-MM-dd hh:mm:ss格式的UTC当前请求时间 |
    | Authorization | 必填。授权头请参考 [CDN API 签名机制](https://docs.azure.cn/zh-cn/cdn/cdn-api-signature) |

## 示例代码

下载链接：[Azure CDN Demo](https://github.com/wacn/AOG-CodeSample/tree/master/cdn/Java/azure-cdn-demo-master)

## 测试用例

```Java
String requestURL = "https://restapi.cdn.azure.cn/subscriptions/e0fbea86-6cf2-4b2d-81e2-9c59f4f96bcb/endpoints/8137ecc4-71c4-11e7-8259-0017fa00a611?apiVersion=1.0";
 
String requestTimestamp = TimeUtil.getUTCTime();
String keyID = "cc65a046-2a32-4f7d-ab22-9ae49507d719";
String keyValue = "<cdn-key>";
String httpMethod = "GET";

CdnOperation cOperation = new CdnOperation();
String authorization = cOperation.calculateAuthorizationHeader(
            requestURL, 
            requestTimestamp,
            keyID, 
            keyValue, 
            httpMethod);
      
String result = cOperation.getRequest(
            requestURL,
            authorization, 
            requestTimestamp);
System.out.println(result);

AzureCDN cc65a046-2a32-4f7d-ab22-9ae49507d719:AE889EA579556251F5CA57B19361BE57C22550F0D8E4941E50819CB9F0296967
{"EndpointId":"8137ecc4-71c4-11e7-8259-0017fa00a611","Settings":{"CustomDomain":"file.azurecloudapi.cn","Host":"file.azurecloudapi.cn","Origin":{"Addresses":["devstoragerm.blob.core.chinacloudapi.cn"],"SchemePort":{"Scheme":"Http","HttpPort":80}},"ICP":"?ICP?16013159?","ServiceType":"Download","AllowedScheme":"Http","CName":"file.azurecloudapi.cn.mschcdn.com"},"Status":{"Enabled":true,"IcpVerifyStatus":"IcpVerified","LifetimeStatus":"Normal","CNameConfigured":false,"FreeTrialExpired":false,"TimeLastUpdated":"2017-07-26T05:46:00+00:00"}}
```

## 刷新缓存调用详解

- Endpoint：

    `POST https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/purges?apiVersion=1.0`

- 请求参数

    | 参数名称 | 参数值 |
    | ------- | ------ |
    | x-azurecdn-request-date | 必填。符合yyyy-MM-dd hh:mm:ss格式的UTC当前请求时间 |
    | Authorization | 必填。授权头请参考 [CDN API 签名机制](https://docs.azure.cn/zh-cn/cdn/cdn-api-signature) |
    | content-type | application/json |

## 示例代码

下载链接：[Azure CDN Demo](https://github.com/wacn/AOG-CodeSample/tree/master/cdn/Java/azure-cdn-demo-master)

## 测试用例

```Java
static void Main(string[] args)
{
// get
    var requestUrl = string.Format("https://restapi.cdn.azure.cn/subscriptions/{0}/endpoints?apiVersion=1.0", "e0fbea86-6cf2-4b2d-81e2-9c59f4f96bcb");
    var requestTime = DateTime.Now.AddHours(-8)
        .ToString("yyyy-MM-dd hh:mm:ss");
    var keyID = "cc65a046-2a32-4f7d-ab22-9ae49507d719";
    var keySecret = "秘钥";
    var requestAuth = CalculateAuthorizationHeader(
		requestUrl, requestTime, keyID, keySecret, "GET");
    Console.WriteLine(requestTime);
    Console.WriteLine(requestAuth);   
    get(requestUrl, requestAuth, requestTime);

    requestUrl = string.Format("https://restapi.cdn.azure.cn/subscriptions/{0}/endpoints/{1}/purges?apiVersion=1.0", "e0fbea86-6cf2-4b2d-81e2-9c59f4f96bcb", "4f7e4141-83d0-11e7-8259-0017fa00a611");
    requestTime = DateTime.Now.AddHours(-8)
		.ToString("yyyy-MM-dd hh:mm:ss");
    keyID = "cc65a046-2a32-4f7d-ab22-9ae49507d719";
    keySecret = "秘钥";
    requestAuth = CalculateAuthorizationHeader(
        requestUrl, requestTime, keyID, keySecret, "POST");
    var requestBody = "{\"Files\": [\"http://azurecloudapi.cn/wp-content/uploads/2016/09/logo2-1.png\"],\"Directories\": [ \"http://azurecloudapi.cn/wp-content/uploads/2016/09/\"]}";
    post(requestUrl, requestAuth, requestTime,requestBody);
    Console.ReadLine();
}

static  void get(string requestUrl, string requestAuth, string requestTime)
{

    Uri uri = new Uri(requestUrl);
    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(uri);
    request.Method = "GET";
    request.Headers["x-azurecdn-request-date"] = requestTime;
    request.Headers["Authorization"] = requestAuth;

    try
    {
        HttpWebResponse response = (HttpWebResponse)request.GetResponse();
        using (StreamReader reader = new 
        StreamReader(response.GetResponseStream()))
        {
            var result = reader.ReadToEnd();
            Console.WriteLine(result);
        }
        response.Close();

    }
    catch (WebException ex)
    {
        Console.WriteLine(ex);
    }
}

static async void post(string requestUrl, string requestAuth, string requestTime, string requestBody)
{
    Uri uri = new Uri(requestUrl);
    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(uri);
    request.Method = "POST";
    request.Headers["x-azurecdn-request-date"] = requestTime;
    request.Headers["Authorization"] = requestAuth;

    byte[] byteArray = Encoding.UTF8.GetBytes(requestBody);
    request.ContentLength = byteArray.Length;
    Stream newStream = request.GetRequestStream();
    newStream.Write(byteArray, 0, byteArray.Length);
    newStream.Close();

    try
    {
        HttpWebResponse response = (HttpWebResponse)request.GetResponse();

        using (StreamReader reader = new StreamReader(response.GetResponseStream()))
        {
            var result = reader.ReadToEnd();
            Console.WriteLine(result);
        }
        response.Close();
    }
    catch (WebException ex)
    {
        Console.WriteLine(ex);
    }
}
```
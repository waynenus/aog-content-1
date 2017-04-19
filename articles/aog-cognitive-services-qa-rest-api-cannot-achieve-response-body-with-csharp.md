<properties
    pageTitle="C# 基于 RestAPI 调用认知服务"
    description="如何解决 C# 调用认知服务 REST API 无法直接获取 Response Body 的问题"
    service=""
    resource="cognitiveservices"
    authors="Yu Tao"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Cognitive Services, REST API, C#"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="cognitive-services-aog"
    ms.date=""
    wacn.date="04/18/2017" />

# C# 基于 REST API 调用认知服务

## **问题描述**

使用 HttpClient 获取的 HttpResponseMessage 类型的 Response，但无法直接获取 Response Body。

## **问题分析**

参考[官方文档](https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)，java、PHP 等语言的参考示例均正常的打印输出请求的 Response Body，但是 C# 语言参考示例却无法直接打印输出 Response Body。

## **解决方法**

1. HttpResponseMessage.Content 属性声明 HTTP 响应的内容，HttpContent.ReadAsStringAsync 方法将 HTTP 内容作为异步操作写入到字符串中。

2. 示例代码：

        class Program
        {
            static void Main(string[] args)
            {
                MakeRequest();
                Console.WriteLine("Hit ENTER to exit...");
                Console.ReadLine();
            }

            static async void MakeRequest()
            {
                var client = new HttpClient();
                var queryString = HttpUtility.ParseQueryString(string.Empty);

                // Request headers
                client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "<key>");//Face API key

                // Request parameters
                queryString["returnFaceId"] = "true";
                queryString["returnFaceLandmarks"] = "false";
                queryString["returnFaceAttributes"] = "age";
                var uri = "https://api.cognitive.azure.cn/face/v1.0/detect?" + queryString;

                HttpResponseMessage response;

                // Request body
                byte[] byteData = Encoding.UTF8.GetBytes("{\"url\":\"http://imgsrc.baidu.com/baike/pic/item/4034970a304e251ff1e3819aa486c9177f3e53bf.jpg\"}");//Picture's URL

                using (var content = new ByteArrayContent(byteData))
                {
                    response = await client.PostAsync(uri, content);
                }

                //response result
                string result = await response.Content.ReadAsStringAsync();
                Console.WriteLine("response:" + result);
            }
        }

3. 认知服务更多调用信息[参考链接](/documentation/services/cognitive-services/)。


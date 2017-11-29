---
title: "使用 Azure Meida Player 无法在某些手机浏览器播放视频"
description: "使用 Azure Meida Player 无法在某些手机浏览器播放视频"
author: hahaxj
resourceTags: 'Media Services, Mobile'
ms.service: media-services
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/24/2017
wacn.date: 11/24/2017
---

# 使用 Azure Meida Player 无法在某些手机浏览器播放视频

## 问题现象

通过 Azure Media Player 播放媒体服务发布的视频，在 PC 端播放正常，在手机端某些浏览器会出现无法播放或者播放没有声音的情况。

此种情况可能是使用了 [Dynamic 模式](https://amp.azure.net/libs/amp/latest/samples/dynamic_setsource.html)加载视频，或者默认使用 HLS-V4 协议，导致某些只支持 HLS-V3 协议的手机浏览器无法兼容。

## 解决方案

使用 [Static 模式](https://amp.azure.net/libs/amp/latest/samples/videotag_setsource.html)进行播放，并配置 streamingFormats 属性来显示指定媒体服务可支持的播放协议。示例代码如下。

```XML
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Azure Media Player</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!--*****START OF Azure Media Player Scripts*****-->
    <!--Note: DO NOT USE the "latest" folder in production. Replace "latest" with a version number like "1.0.0"-->
    <!--EX:<script src="//amp.azure.net/libs/amp/1.0.0/azuremediaplayer.min.js"></script>-->
    <!--Azure Media Player versions can be queried from //aka.ms/ampchangelog-->
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <script src="//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
    <!--*****END OF Azure Media Player Scripts*****-->
  </head>
  <body>
    <h1>Sample: Clear</h1>
    <video id="azuremediaplayer" class="azuremediaplayer amp-default-skin amp-big-play-centered" controls autoplay width="640" height="400" poster="" data-setup='{}' tabindex="0">
      <source src="//amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest" type="application/vnd.ms-sstr+xml" data-setup='{"streamingFormats": ["DASH","SMOOTH","HLS-V3", "HLS-V4"] }'/>
      <p class="amp-no-js">To view this video please enable JavaScript, and consider upgrading to a web browser that supports HTML5 video</p>
    </video>
    <footer>
      <br />
      <p>© Microsoft Corporation 2016</p>
    </footer>
  </body>
</html>
```
## 参考文档

[Azure Media Player 的相关范例](https://amp.azure.net/libs/amp/latest/docs/samples.html)
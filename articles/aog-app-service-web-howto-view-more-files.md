---
title: "Kudu 的 Debug Console 窗口如何查看更多文件"
description: "Kudu 的 Debug Console 窗口如何查看更多文件"
author: 123Jun321
resourceTags: 'App Service Web, Kudu, Debug Console'
ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-ciwu
ms.date: 12/5/2018
wacn.date: 12/5/2018
---

# Kudu 的 Debug Console 窗口如何查看更多文件

## 问题描述

如果在 kudu 某个文件夹中出现文件过多的情况，无法在文件窗口都展示出来会报如下的错误，如果我们想展示更多的文件，可以通过修改 maxViewItems 来实现。

```
There are xxx items in this directory, but maxViewItems is set to 299. You can increase maxViewItems by setting it to a larger value in localStorage.
```

## 解决方法

1. 按 F12 按钮，出现浏览器的开发者工具（推荐 chrome）。
2. 在 Console 工具处输入命令 `*window.localStorage['maxViewItems'] = 350*`，敲回车键，之后回退到上一级文件夹，再次点击您要查看的文件夹即可。
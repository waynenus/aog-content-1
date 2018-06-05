---
title: "Azure Active Directory 中如何删除本机类型的应用"
description: "Azure Active Directory 中如何删除本机类型的应用"
author: taroyutao
resourceTags: 'Azure Active Directory'
ms.service: active-directory
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 5/31/2018
wacn.date: 5/31/2018
---

# Azure Active Directory 中如何删除本机类型的应用

## 问题描述

用户在使用 Azure Active Directory (以下简称：AAD) 时想要将应用想进行删除，却发现删除按钮无法使用。

![01](media/aog-active-directory-qa-remove-local-app/01.png)

## 解决方法

这是因为您要删除的应用类型是本机类型，本地应用在添加时默认是多租户类型的应用所以无法删除。<br>
只需要打开该应用的清单找到 `availableToOtherTenants` 设置为 `false`，保存后删除按钮就能够正常操作了。

![02](media/aog-active-directory-qa-remove-local-app/02.png)

保存后删除功能恢复正常:

![03](media/aog-active-directory-qa-remove-local-app/02.png)
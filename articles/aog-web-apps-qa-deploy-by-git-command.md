---
title: 如何使用 git submodule 命令部署 Azure Web 应用
description: 如何使用 git submodule 命令部署 Azure Web 应用
services: Web Apps
documentationCenter: ''
author: zhangyannan-yuki
manager: ''
editor: ''
tags: 'Web Apps, Git, Submodule'

ms.service: app-service-web
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 07/18/2017
wacn.date: 07/18/2017
---

# 如何使用 git submodule 命令部署 Azure Web 应用

## 问题描述

使用 git 进行部署的过程中在执行 git submodule add 的命令时，如果 URL 中不提供 FTP 的用户名以及密码，在部署的时候可能就会报 deployment failed 的错误。

## 解决方法

关于部署 git submodule 的步骤如下：

在使用 Web 应用站点作为 GIT repo 时，使用的 url 格式如下：https://username@sitemain.scm.chinacloudsites.cn:443/sitemain.git ( git remote add main https://username@sitemain.scm.chinacloudsites.cn:443/sitemain.git ) 这个 url 访问时是需要用户名密码的。 但是我们看到，这个 url 中并不包含密码信息。 

通常情况下，这样的 url 在 git 客户端直接用来 push 时，是没有问题的，因为 git 会弹出登录界面，在这种情况下就有机会输入密码。 

但是，在有 submodule 的场景中， 如果 Web 应用也作为 submodule repo，会有一些问题。 例如通过如下命令注册子模块：`git submodule add https://username@sitesubmodule.scm.chinacloudsites.cn:443/sitesubmodule.git libs` 在这种情况下我们是没有机会输入密码的。具体在于 push 主项目时，服务端也需要通过提供的子模块 git url 进行 updating sbumodules,  然而服务端是没有机会弹出界面让用户输入密码的， 于是就会报 deployment failed 这个错误。 对于这种情况建议在执行 git submodule add 命令时，url 中提供 ftp 用户名和 ftp 密码, 格式如下： `git submodule add https://username:MyPassword@sitename.scm.chinacloudsites.cn:443/sitename.git libs`

以下是成功部署 submodule 的命令列表 :

```
git init
git add -A  
git commit -m "init"
git remote add main https:// testftpname@ testwebapp1.scm.chinacloudsites.cn:443/ testwebapp1.git 
git push main master
git submodule add https://testftpname:MyPassword@ testwebapp2.scm.chinacloudsites.cn:443/ testwebapp2.git libs  
```

> [!TIP]
> 其中 testwebapp1 为主站点, testwebapp2 为子模块站点, FTP 用户名为 testftpname, FTP 密码为 MyPassword。

`git submodule add` 命令代表增加一个子模块 ，`https` 代表的是子模块的仓库地址，`libs` 代表将子模块放置在当前工程下的路径，`libs` 名称可以自定义。

```
git submodule update –remote //更新带有 submodule 的项目
git add -A  //-A 代表提交所有变化的文件
git commit -m "add submodule and change some files" 
```

`git commit` 代表将本地修改的内容提交到缓存中，一般是 `git commit –m “xxxx”` ,`xxx` 代表修改的内容，但这个只是提交到缓存中，`git push` 才代表真正提交到本地存储库。

```
git submodule update   
Git push main master
```

相应的输出结果：

![result](./media/aog-web-apps-qa-deploy-by-git-command/result.png)
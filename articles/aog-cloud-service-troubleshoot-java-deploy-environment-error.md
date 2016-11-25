# 关于部署在 cloud service 下的 Java 应用程序会出现 Staging 环境被 “同步” 到 Production 环境的问题 #

### 问题描述 ###

在使用 Java+eclipse 做应用程序开发时，会将开发好的应用程序 code 发布到 azure cloud service staging 环境做测试观察，如果运行稳定，可以将相关更新后的 code 发布到正式的 Production 环境中或者使用 swap 的功能切换环境。

然而在满足以下全部限定条件时，会出现 staging 环境中的 code “被同步” 到 Production 环境中的现象：

1. 使用 Java+eclipse 做应用程序开发，部署到 cloud service 中。
2. 在用 eclipse 发布相关应用到 staging 环境和 Production 环境中时使用的是同一个 storage account。
3. 将相关更新后的应用程序 code 发布到 staging 环境（Production 环境中依旧是旧版本的应用程序 code），并且重启 Production 环境中的实例。

### 问题分析 ###

当使用 .NET +Visual Studio 做开发，同样为 staging 环境和 Production 环境配置同一个 storage account 却不会出现同样的问题。因此，需要了解 .NET +Visual Studio 以及 Java+eclipse 在发布应用程序到 cloud service 时配置的 storage account 分别是做什么用的：

1. .NET +Visual Studio

 使用 Visual Studio 在部署应用时指定的 storage account 会存储相关的诊断数据，详细的说明可以参考[链接 1 ](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-dotnet-diagnostics-storage/)和[链接 2](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-dotnet-diagnostics/)。而应用程序的代码是不会存放在该 storage account 中的。 

2. Java+eclipse

 使用 Java+eclipse 在部署应用时指定的 storage account 则是用来存放 Java 应用程序生成的 .war 文件，而该文件里包含了使用 Java 开发的应用程序代码，简而言之，部署时指定的 storage account 是用来存放应用程序代码的。
 
 ![storage-account](./media/aog-cloud-service-java-deploy-environment-error/storage-account.jpg)
 
所以使用 Java+eclipse 做开发就会出现这样一个问题，如果 staging 和 Production 指定的是同一个 storage account，当我们向 staging 环境做新的部署时，原来旧版本的 .war 文件会被新的 .war 文件覆盖掉。这时如果 Production 环境的实例因为某些原因重启，在重启的过程中 cloud server 会为 Production 环境重新拿各种文件（包括应用 code），因此 Production 环境也会拿到从 staging 环境更新上来的 .war 文件，产生重启后“被同步的现象”。

而在使用 .NET +Visual Studio 发布应用程序时，客户的 code 会被 cloud service 放在 azure 后台单独的 storage account 中，针对不同的环境，该 storage account 会不同，因此不会出现 staging 环境的 code 跟 Production 环境中的 code 使用同一 storage account 的问题。

### 解决方法 ###

根据以上分析，可以了解到当使用 Java+eclipse 做开发时，staging 与 Production 环境共享同一 storage account，应用 code 会被重写，因此会有一定的几率出现“被同步”的问题。
针对这种情况，建议在使用 Java+eclipse 做开发部署到 cloud service 中时，为 staging 和 Production 环境配置两个不同的 storage account。

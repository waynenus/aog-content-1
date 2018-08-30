---
title: '如何使用 Blobfuse 将 Blob 存储装载为文件系统'
description: '如何使用 Blobfuse 将 Blob 存储装载为文件系统'
author: StellaZhou64 
resourceTags: 'Storage, Blob, Blobfuse, NTFS'
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: jieying.zhou
ms.date: 08/30/2018
wacn.date: 08/30/2018
---

# 如何使用 Blobfuse 将 Blob 存储装载为文件系统

本文以 Ubuntu18.04 为例，演示使用 Blobfuse 将 Blob 存储装载为文件系统。若要详细了解 Blobfuse，请阅读 [Blobfuse 存储库](https://github.com/Azure/azure-storage-fuse)中的详细信息。

## 安装 Blobfuse

```shell
wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install blobfuse
```

可将 URL 更改为 `.../ubuntu/16.04/...`，使之指向 Ubuntu 16.04 发行版。其他 Linux 版本请参考[这里](https://github.com/Azure/azure-storage-fuse/wiki/1.-Installation)。

## 准备装载

Blobfuse 要求文件系统中存在一个临时路径，用于缓冲和缓存任何打开的文件，以便提供类似本机的性能。 对于此临时路径，请选择性能最高的磁盘，或者使用 ramdisk 来获得最佳性能。请确保有足够的空间来容纳所有打开的文件。

### （可选）将 ramdisk 用于临时路径

以下示例创建 16 GB 的 ramdisk，并创建适用于 Blobfuse 的目录。 请根据需要选择大小。 此 ramdisk 允许 Blobfuse 打开多个文件，只要其大小总计不超过 16 GB 。

```shell
sudo mount -t tmpfs -o size=16g tmpfs /mnt/ramdisk
sudo mkdir /mnt/ramdisk/blobfusetmp
sudo chown <youruser> /mnt/ramdisk/blobfusetmp
```

### 将 SSD 用于临时路径

在 Azure 中，可以使用 VM 上提供的临时磁盘 (SSD)，为 Blobfuse 提供低延迟缓冲区。 此临时磁盘在 Ubuntu 发行版中装载在 `/mnt` 上。

请确保用户可以访问临时路径：

```shell
sudo mkdir /mnt/blobfusetmp
sudo chown <youruser> /mnt/blobfusetmp
```

### 配置存储帐户凭据

Blobfuse 要求将凭据采用以下格式存储在文本文件 fuse_connection.cfg 中：

```
accountName myaccount
accountKey myaccesskey==
containerName mycontainer
blobEndpoint myaccount.blob.core.chinacloudapi.cn
```

创建此文件以后，请确保限制对它的访问权限，防止其他用户读取它。

```shell
chmod 700 fuse_connection.cfg
```

创建装载用的空目录

```shell
mkdir ~/mycontainer
```

## 装载

请以用户身份运行以下命令。 此命令将 `/path/to/fuse_connection.cfg` 中指定的容器装载到 `/mycontainer` 位置。 `/path/to/fuse_connection.cfg` 请使用全路径。

```shell
blobfuse ~/mycontainer --tmp-path=/mnt/blobfusetmp  --config-file=/path/to/fuse_connection.cfg
```

### 参数说明：

- `--tmp-path=/path/to/cache` : 指定缓存文件的临时文件夹。为了更好的性能，建议配置 SSD 或 ramdisk。
- `--config-file=/path/to/connection.cfg` : 指定访问 Blob 账号信息的配置文件。
- [可选] `--use-https=true|false` : 对 Blob 访问是否启用 https。默认 `True`。
- [可选] `--file-cache-timeout-in-seconds=120` : 临时文件夹缓存内容过期时间。默认 `120` 秒。指定此选项后，在过期时间之内，Blobfuse 将读取缓存中内容而不是从 Blob 下载最新内容。到过期时间后，临时文件夹中的内容会被删除。请确保临时文件夹有足够的空间容纳 Blob 内容，或者将该参数设为 `0` 来加速缓存内容的清除。
- [可选] `--log-level=LOG_WARNING` : 可将日志写到 syslog。默认日志级别为 `LOG_WARNING by default`。可选日志级别为 `LOG_OFF|LOG_CRIT|LOG_ERR|LOG_WARNING|LOG_INFO|LOG_DEBUG`。

现在就可以通过常规文件系统 API 访问块 Blob。 请注意，装载的目录只能由装载它的用户访问，这样是为了确保访问安全。 如果希望所有用户都能够访问，可以通过选项 `-o allow_other` 进行装载。

```shell
cd ~/mycontainer
mkdir test
echo "hello world" > test/blob.txt
```

对于只读访问：

1. 由于 Blob 会在临时文件夹缓存并停留一段时间（`--file-cache-timeout-in-seconds` 指定），如果 Blob 资源被修改，这些改动将在缓存过期，且文件重新打开后获得。
2. 将 `--file-cache-timeout-in-seconds` 设置为 `0`，可以忽略缓存而得到最新的文件内容。

对于读写访问：

1. 在挂载 Blobfuse 后，不建议 编辑/ 修改/删除 临时文件夹中的内容。这么做有数据丢失风险。
2. 当 container 被挂载后，该 container 中的数据只能被 Blobfuse 处理。
3. 如果一个文件同时被多方编辑，每一方在关闭文件后都会将改动上传至 Blob 存储。
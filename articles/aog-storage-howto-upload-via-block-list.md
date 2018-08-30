---
title: 'Azure Storage 分块上传示例'
description: 'Azure Storage 分块上传示例'
author: taroyutao
resourceTags: 'Storage, Block List, Java, Python, PHP'
ms.service: storage
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 08/29/2018
wacn.date: 08/29/2018
---

# Azure Storage 分块上传示例

## 概述

Azure 存储提供三种类型的 Blob：块 Blob、页 Blob 和追加 Blob。其中，块 Blob 特别适用于存储短的文本或二进制文件，例如文档和媒体文件。

块 Blob 由块组成，每个块可以是不同的大小，最大为 100MB (对于 2016-05-31 之前 REST 版本的请求为 4MB )，块 Blob 最多可以包含 50,000 块。因此，块 Blob 的最大大小约为 4.75 TB (100MB X 50,000 块)。对于 2016-05-31 之前的 REST 版本，块 Blob 的最大大小约为 195 GB（4MB X 50,000 块），更多详细信息，请参阅[块 Blob、追加 Blob 以及页 Blob 介绍](https://docs.microsoft.com/zh-cn/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)。

在上传文件到 Azure Blob 存储时，Azure 支持两种方式，**整体上传** 和 **分块上传**。

- **整块上传**：当上传到块 Blob 的文件小于等于 [SingleBlobUploadThresholdInBytes](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.singleblobuploadthresholdinbytes?view=azure-dotnet) 属性（客户端可以通过设置该属性设置单个 Blob 上传的最大值，范围介于 1MB 和 256MB 之间）的值时，则可以采用整体上传的方式，调用 PutBlob 完整的上传 Blob ，更多详细信息，请参考 [PutBlob](https://docs.microsoft.com/zh-cn/rest/api/storageservices/put-blob) 。

- **分块上传**：当上传的块 Blob 的文件大于 SingleBlobUploadThresholdInBytes 属性的值时，存储客户端会根据 [StreamWriteSizeInBytes](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.blob.cloudblockblob.streamwritesizeinbytes?view=azure-dotnet) (客户端可以通过设置该属性设置单个分块 Blob 的大小，范围介于 16KB 和 100MB 之间) 的值将文件分解成块, 采用分块上传的方式上传文件，更多详细信息，请参考 [PubBlobList](https://docs.microsoft.com/zh-cn/rest/api/storageservices/put-block-list) 。

## 示例程序

关于 C# 的示例目前[官网文档](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/storage/aog-storage-blob-howto-upload-big-file-to-storage)已经给出，下面分别给出 JAVA、Python 及 PHP 语言的示例程序。

- JAVA Code Sample

    ```java
    import com.microsoft.azure.storage.CloudStorageAccount;
    import com.microsoft.azure.storage.StorageException;
    import com.microsoft.azure.storage.blob.BlockEntry;
    import com.microsoft.azure.storage.blob.CloudBlobClient;
    import com.microsoft.azure.storage.blob.CloudBlobContainer;
    import com.microsoft.azure.storage.blob.CloudBlockBlob;
    import java.io.FileInputStream;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.security.InvalidKeyException;
    import java.util.ArrayList;
    import java.util.Base64;

    public class uploadFileBlocksAsBlockBlob {

        public static void main(String[] args) throws IOException, StorageException, URISyntaxException, InvalidKeyException {

            FileInputStream fileInputStream = null;
            //存储连接字符串
            String storageConnectionString = "<storage connection string>";
            //上传容器的名称
            String containerName= "aaa1";
            // Parse the connection string and create a blob client to interact with Blob storage
            CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
            CloudBlobClient blobClient = storageAccount.createCloudBlobClient();
            CloudBlobContainer container = blobClient.getContainerReference(containerName);
            CloudBlockBlob blockBlob = container.getBlockBlobReference("testblockblob");//希望上传后文件的名称

            //本地文件的位置
            String filePath = "D:\\test.zip";

            try {
                // Open the file
                fileInputStream = new FileInputStream(filePath);
                // Split the file into 32K blocks (block size deliberately kept small for the demo) and upload all the blocks
                int blockNum = 0;
                String blockId = null;
                String blockIdEncoded = null;
                ArrayList<BlockEntry> blockList = new ArrayList<BlockEntry>();
                while (fileInputStream.available() > (32 * 1024)) {
                    blockId = String.format("%05d", blockNum);
                    blockIdEncoded = Base64.getEncoder().encodeToString(blockId.getBytes());
                    blockBlob.uploadBlock(blockIdEncoded, fileInputStream, (32 * 1024));
                    blockList.add(new BlockEntry(blockIdEncoded));
                    blockNum++;
                }
                blockId = String.format("%05d", blockNum);
                blockIdEncoded = Base64.getEncoder().encodeToString(blockId.getBytes());
                blockBlob.uploadBlock(blockIdEncoded, fileInputStream, fileInputStream.available());
                blockList.add(new BlockEntry(blockIdEncoded));

                // Commit the blocks
                blockBlob.commitBlockList(blockList);
            }
            catch (Throwable t) {
                throw t;
            }
            finally {
                // Close the file output stream writer
                if (fileInputStream != null) {
                    fileInputStream.close();
                }
            }
        }
    }
    ```

- Python Code Sample

    ```python
    from azure.storage.blob import BlockBlobService, PublicAccess
    from azure.storage.blob.models import BlobBlock
    from AzureStorageDemo.RandomData import RandomData

    # Block Blob Operations
    def block_blob_operations():
        # Define the upload file
        file_to_upload = "test.mp4"
        # Define the block size
        block_size = 1024*1024*4 #4MB

        # Create blockblob_service object
        blockblob_service = BlockBlobService(
            connection_string='<storage connection string>')

        # Define container name
        container_name = 'blockblobcontainer' + RandomData().get_random_name(6)
        print("Container name：" + container_name)

        try:
            # Create a new container
            print('1. Create a container with name - ' + container_name)
            blockblob_service.create_container(container_name)

            blocks = []

            # Read the file
            print('2. Upload file to block blob')
            with open(file_to_upload, "rb") as file:
                file_bytes = file.read(block_size)
                while len(file_bytes) > 0:
                    block_id = RandomData().get_random_name(32)
                    blockblob_service.put_block(container_name, file_to_upload, file_bytes, block_id)

                    blocks.append(BlobBlock(id=block_id))

                    file_bytes = file.read(block_size)

            blockblob_service.put_block_list(container_name, file_to_upload, blocks)

            print('3. Get the block list')
            blockslist = blockblob_service.get_block_list(container_name, file_to_upload, None, 'all')
            blocks = blockslist.committed_blocks

            print('4. Enumerate blocks in block blob')
            for block in blocks:
                print('Block ' + block.id)
        finally:
            print('5. Delete container')
            if blockblob_service.exists(container_name):
                blockblob_service.delete_container(container_name)

    # Main method.
    if __name__ == '__main__':
        block_blob_operations()
    ```

- PHP Code Sample

    ```php
    <?php
    require_once 'vendor\autoload.php';

    use MicrosoftAzure\Storage\Common\ServicesBuilder;
    use MicrosoftAzure\Storage\Common\ServiceException;
    use MicrosoftAzure\Storage\Blob\Models\BlockList;

    // Create blob REST proxy.
    $connectionString = 'BlobEndpoint=https://<storage account name>.blob.core.chinacloudapi.cn/;QueueEndpoint=https://<storage account name>.queue.core.chinacloudapi.cn/;TableEndpoint=https://<storage account name>.table.core.chinacloudapi.cn/;AccountName=<storage account name>;AccountKey=<storage account key>';
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);

    $container = "aaaa";
    $file_name = 'bigfile';
    $blob_name = basename($file_name);
    $block_list = new BlockList();
    define('CHUNK_SIZE', 4 * 1024 * 1024);#设置每个块的大小为 4MB
    try {
        $fptr = fopen($file_name, "rb");
        $index = 1;
        while (!feof($fptr)) {
            $block_id = base64_encode(str_pad($index, 6, "0", STR_PAD_LEFT));
            $block_list->addUncommittedEntry($block_id);
            $data = fread($fptr, CHUNK_SIZE);
            $blobRestProxy->createBlobBlock($container, $blob_name, $block_id, $data);
            ++$index;
        }
        $blobRestProxy->commitBlobBlocks($container, $blob_name, $block_list);
    } catch (ServiceException $e) {
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }
    echo 'upload big file success'
    ?>
    ```

## 参考连接

- [上传大文件到 Azure 存储块 Blob](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/storage/aog-storage-blob-howto-upload-big-file-to-storage)
- [azure-storage-java](https://github.com/Azure/azure-storage-java)
- [azure-storage-python](https://github.com/Azure/azure-storage-python)
- [Uploading Large File By Splitting Into Blocks In Windows Azure Blob Storage Using Windows Azure SDK For PHP](http://gauravmantri.com/2013/07/13/uploading-large-file-by-splitting-into-blocks-in-windows-azure-blob-storage-using-windows-azure-sdk-for-php/)
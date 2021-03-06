---
title: 存储和访问 Azure 存储空间中的数据
description: Azure 存储是一种可扩展的云存储解决方案，可以用于存储非结构化和结构化数据。 本文演示如何使用 Xamarin.Forms 将文本和二进制数据存储在 Azure 存储空间，以及如何访问数据。
ms.prod: xamarin
ms.assetid: 5B10D37B-839B-4CD0-9C65-91014A93F3EB
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 06/16/2017
ms.openlocfilehash: 63afeec81eff350b034e8dd3a13da52801937826
ms.sourcegitcommit: 945df041e2180cb20af08b83cc703ecd1aedc6b0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/04/2018
ms.locfileid: "30789867"
---
# <a name="storing-and-accessing-data-in-azure-storage"></a>存储和访问 Azure 存储空间中的数据

_Azure 存储是一种可扩展的云存储解决方案，可以用于存储非结构化和结构化数据。本文演示如何使用 Xamarin.Forms 将文本和二进制数据存储在 Azure 存储空间，以及如何访问数据。_

## <a name="overview"></a>概述

Azure 存储空间提供了四个存储服务：

- Blob 存储。 Blob 可以是文本或二进制数据，如备份、 虚拟机、 媒体文件或文档。
- 表存储是 NoSQL 键-属性存储。
- 队列存储是工作流处理和云服务之间的通信的消息传递服务。
- 文件存储提供了使用 SMB 协议的共享的存储。

有两种类型的存储帐户：

- 通过单个帐户情况下，通用的存储帐户提供对 Azure 存储服务的访问。
- Blob 存储帐户是专门的存储帐户用于存储 blob。 你只需以存储 blob 数据，则建议使用此帐户类型。

这篇文章，并随附的示例应用程序，演示到 blob 存储，并将其下载上载图像和文本的文件。 此外，它还演示从 blob 存储检索的文件的列表和删除文件。

有关 Azure 存储空间的详细信息，请参阅[存储空间简介](https://azure.microsoft.com/documentation/articles/storage-introduction/)。

## <a name="introduction-to-blob-storage"></a>Blob 存储空间简介

Blob 存储包含三个组件，如下图所示：

![](azure-storage-images/blob-storage.png "Blob 存储的概念")

到 Azure 存储空间的所有访问都都通过存储帐户。 存储帐户可以包含无限的数量的容器，并且一个容器可以存储无限的数量的 blob，直至达到存储帐户容量限制。

Blob 是任何类型和大小的文件。 Azure 存储空间支持三种不同的 blob 类型：

- 块 blob 进行了优化来流化和存储云对象，并且是一个不错的选择用于存储备份、 媒体文件和文档等。块 blob 可以是最多为 195 Gb 大小。
- 追加 blob 类似于块 blob，但针对进行了优化追加操作，例如日志记录。 追加 blob 可以是最多为 195 Gb 大小。
- 页 blob 为频繁的读/写操作进行了优化，并且通常用于存储虚拟机和其磁盘。 页 blob 可以达 1 Tb 的大小。

> [!NOTE]
> 请注意，blob 存储帐户支持块追加 blob，但不是页 blob。

Blob 上载到 Azure 存储空间，并从 Azure 存储空间下载的字节流的形式。 因此，文件必须转换为的上载，并为其原始的表示形式下载后转换的回之前的字节流。

Azure 存储中存储每个对象具有唯一的 URL 地址。 存储帐户名称构成该地址和子域和域名名称窗体组合子域*终结点*的存储帐户。 例如，如果你的存储帐户名为*mystorageaccount*，存储帐户的默认 blob 终结点`https://mystorageaccount.blob.core.windows.net`。

将存储帐户中的对象的位置附加到终结点生成用于访问存储帐户中的某个对象的 URL。 例如，blob 地址将具有格式`https://mystorageaccount.blob.core.windows.net/mycontainer/myblob`。

## <a name="setup"></a>安装

将 Azure 存储帐户集成到 Xamarin.Forms 应用程序的过程如下所示：

1. 创建存储帐户。 有关详细信息，请参阅[创建存储帐户](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/#create-a-storage-account)。
1. 添加[Azure Storage Client Library](https://www.nuget.org/packages/WindowsAzure.Storage/)到 Xamarin.Forms 应用程序。
1. 配置存储连接字符串。 有关详细信息，请参阅[连接到 Azure 存储](#connecting)。
1. 添加`using`指令`Microsoft.WindowsAzure.Storage`和`Microsoft.WindowsAzure.Storage.Blob`命名空间添加到将访问 Azure 存储空间的类。

> [!NOTE]
> 尽管此示例使用的共享访问项目，Azure 存储客户端库现在也支持从可移植类库 (PCL) 项目已使用。

<a name="connecting" />

## <a name="connecting-to-azure-storage"></a>连接到 Azure 存储

针对存储帐户资源发出的每个请求必须进行身份验证。 尽管 blob 可以配置为支持匿名身份验证，则有应用程序可用于使用存储帐户进行身份验证的两种主要方法：

- 共享的密钥。 此方法使用 Azure 存储帐户名称和帐户密钥访问存储服务。 存储帐户分配在创建可用于共享密钥身份验证的两个私有密钥。
- 共享的访问签名。 这是一个可以附加到 URL 的实现对存储资源的委托的访问令牌，，权限使用它，为指定的是有效的时间段。

可以指定连接字符串，包括从应用程序访问 Azure 存储资源所需的身份验证信息。 此外，还可以配置连接字符串，以从 Visual Studio 连接到 Azure 存储模拟器。

> [!NOTE]
> Azure 存储连接字符串中支持 HTTP 和 HTTPS。 但是，建议使用 HTTPS。

### <a name="connecting-to-the-azure-storage-emulator"></a>连接到 Azure 存储模拟器

Azure 存储模拟器提供了一个模拟 Azure blob、 队列和表服务以进行开发的本地环境。

应使用以下连接字符串连接到 Azure 存储模拟器：

```csharp
UseDevelopmentStorage=true
```

有关 Azure 存储模拟器的详细信息，请参阅[使用 Azure 存储模拟器进行开发和测试](https://azure.microsoft.com/documentation/articles/storage-use-emulator/)。

### <a name="connecting-to-azure-storage-using-a-shared-key"></a>连接到 Azure 存储空间中使用共享的密钥

以下连接字符串的格式应该用于连接到 Azure 存储具有共享密钥：

```csharp
DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey
```

`myAccountName` 应替换为你的存储帐户的名称和`myAccountKey`应使用两个帐户访问密钥之一进行替换。

> [!NOTE]
> 何时使用共享密钥身份验证，你的帐户名称和帐户密钥将分发到每个用户都使用你的应用程序，将提供完全读/写访问权限的存储帐户中。 因此，仅，用于测试目的使用共享密钥身份验证，并且永远不会将密钥分发给其他用户。

### <a name="connecting-to-azure-storage-using-a-shared-access-signature"></a>连接到 Azure 存储空间使用共享访问签名

以下连接字符串的格式应该用于连接到使用 SAS 的 Azure 存储：

`BlobEndpoint=myBlobEndpoint;SharedAccessSignature=mySharedAccessSignature`

`myBlobEndpoint` 应替换为你的 blob 终结点的 URL 和`mySharedAccessSignature`应替换为你的 SAS。 SAS 提供协议、 服务终结点和用于访问资源的凭据。

> [!NOTE]
> SAS 身份验证被建议用于生产应用程序。 但是，在生产应用程序应从后端服务按需，而不是与应用程序捆绑在正在检索 SAS。

有关共享访问签名的详细信息，请参阅[使用共享访问签名 (SAS)](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。

## <a name="creating-a-container"></a>创建容器

`GetContainer`方法用于检索对一个命名的容器，它随后可从容器中检索 blob 或将 blob 添加到容器的引用。 下面的代码示例演示`GetContainer`方法：

```csharp
static CloudBlobContainer GetContainer(ContainerType containerType)
{
  var account = CloudStorageAccount.Parse(Constants.StorageConnection);
  var client = account.CreateCloudBlobClient();
  return client.GetContainerReference(containerType.ToString().ToLower());
}
```

`CloudStorageAccount.Parse`方法分析连接字符串并返回`CloudStorageAccount`表示存储帐户的实例。 A`CloudBlobClient`实例，用于检索容器和 blob，然后由`CreateCloudBlobClient`方法。 `GetContainerReference`方法检索为指定的容器`CloudBlobContainer`实例，它将返回到调用方法之前。 在此示例中，容器名称是`ContainerType`枚举值，转换为小写的字符串。

> [!NOTE]
> 容器名称必须小写，并且必须以字母或数字开头。 此外，它们只能包含字母、 数字和短划线字符，并且必须介于 3 到 63 个字符之间。

`GetContainer`方法调用，如下所示：

```csharp
var container = GetContainer(containerType);
```

`CloudBlobContainer`然后可以使用实例创建一个容器，如果它尚不存在：

```csharp
await container.CreateIfNotExistsAsync();
```

默认情况下，新创建的容器是私有的。 这意味着，必须指定存储访问密钥容器中检索 blob。 有关如何使容器公开中的 blob 的信息，请参阅[创建容器](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/#create-a-container)。

## <a name="uploading-data-to-a-container"></a>将数据上载到容器

`UploadFileAsync`方法可用于上载流的字节数据到 blob 存储，并在下面的代码示例所示：

```csharp
public static async Task<string> UploadFileAsync(ContainerType containerType, Stream stream)
{
  var container = GetContainer(containerType);
  await container.CreateIfNotExistsAsync();

  var name = Guid.NewGuid().ToString();
  var fileBlob = container.GetBlockBlobReference(name);
  await fileBlob.UploadFromStreamAsync(stream);

  return name;
}
```

在检索之后容器引用，该方法创建的容器，如果它尚不存在。 一个新`Guid`然后创建以充当唯一 blob 名称，并为检索 blob 块引用`CloudBlockBlob`实例。 然后，数据的流上载到 blob 使用`UploadFromStreamAsync`方法，将创建 blob，如果它尚不存在，或如果它存在覆盖它。

将文件上载到 blob 存储使用此方法之前，它必须首先必须转换为字节流。 在下面的代码示例说明了这一点：

```csharp
var byteData = Encoding.UTF8.GetBytes(text);
uploadedFilename = await AzureStorage.UploadFileAsync(ContainerType.Text, new MemoryStream(byteData));
```

`text`数据转换为字节数组，然后传递给流的形式包装`UploadFileAsync`方法。

## <a name="downloading-data-from-a-container"></a>从容器下载数据

`GetFileAsync`方法可用于从 Azure 存储空间下载 blob 数据，并在下面的代码示例所示：

```csharp
public static async Task<byte[]> GetFileAsync(ContainerType containerType, string name)
{
  var container = GetContainer(containerType);

  var blob = container.GetBlobReference(name);
  if (await blob.ExistsAsync())
  {
    await blob.FetchAttributesAsync();
    byte[] blobBytes = new byte[blob.Properties.Length];

    await blob.DownloadToByteArrayAsync(blobBytes, 0);
    return blobBytes;
  }
  return null;
}
```

在检索之后容器引用，该方法将检索存储的数据的 blob 引用。 如果存在 blob，通过检索其属性`FetchAttributesAsync`方法。 将创建的正确大小的字节数组，并获取返回到调用方法的字节数组形式下载 blob。

在下载的 blob 字节数据之后, 它必须转换为其原始的表示形式。 在下面的代码示例说明了这一点：

```csharp
var byteData = await AzureStorage.GetFileAsync(ContainerType.Text, uploadedFilename);
string text = Encoding.UTF8.GetString(byteData);
```

从 Azure 存储空间中检索的字节数组`GetFileAsync`方法之前它将转换回为 UTF8, 编码的字符串。

## <a name="listing-data-in-a-container"></a>列出容器中的数据

`GetFilesListAsync`方法可用于检索的存储容器中的 blob 列表，并在下面的代码示例所示：

```csharp
public static async Task<IList<string>> GetFilesListAsync(ContainerType containerType)
{
  var container = GetContainer(containerType);

  var allBlobsList = new List<string>();
  BlobContinuationToken token = null;

  do
  {
    var result = await container.ListBlobsSegmentedAsync(token);
    if (result.Results.Count() > 0)
    {
      var blobs = result.Results.Cast<CloudBlockBlob>().Select(b => b.Name);
      allBlobsList.AddRange(blobs);
    }
    token = result.ContinuationToken;
  } while (token != null);

  return allBlobsList;
}
```

在检索之后容器引用，该方法使用容器的`ListBlobsSegmentedAsync`方法来检索对容器内 blob 的引用。 返回的结果`ListBlobsSegmentedAsync`枚举方法时`BlobContinuationToken`实例不是`null`。 每个 blob 被强制转换从返回`IListBlobItem`到`CloudBlockBlob`顺序访问中`Name`的 blob 之前它是值, 的属性添加到`allBlobsList`集合。 一次`BlobContinuationToken`实例是`null`，已返回最后一个的 blob 名称，并执行退出循环。

## <a name="deleting-data-from-a-container"></a>从容器中删除数据

`DeleteFileAsync`方法可用于在容器中，删除 blob，并在下面的代码示例所示：

```csharp
public static async Task<bool> DeleteFileAsync(ContainerType containerType, string name)
{
  var container = GetContainer(containerType);
  var blob = container.GetBlobReference(name);
  return await blob.DeleteIfExistsAsync();
}
```

在检索之后容器引用，该方法检索指定的 blob 的 blob 引用。 然后与删除的 blob`DeleteIfExistsAsync`方法。

## <a name="summary"></a>总结

本文演示如何使用 Xamarin.Forms 将文本和二进制数据存储在 Azure 存储空间，以及如何访问数据。 Azure 存储是一种可扩展的云存储解决方案，可以用于存储非结构化和结构化数据。


## <a name="related-links"></a>相关链接

- [Azure 存储 （示例）](https://developer.xamarin.com/samples/xamarin-forms/WebServices/AzureStorage/)
- [存储空间简介](https://azure.microsoft.com/documentation/articles/storage-introduction/)
- [如何通过 Xamarin 使用 Blob 存储](https://azure.microsoft.com/documentation/articles/storage-xamarin-blob-storage/)
- [使用共享的访问签名 (SAS)](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)
- [Windows Azure 存储空间](https://www.nuget.org/packages/WindowsAzure.Storage/)

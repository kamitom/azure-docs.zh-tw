---
title: Azure Blob 儲存輸入繫結以 Azure 函式
description: 瞭解如何向 Azure 函數提供 Azure Blob 存儲數據。
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.openlocfilehash: 33db9a8d86e02db2076cdb85170d466697930b96
ms.sourcegitcommit: d597800237783fc384875123ba47aab5671ceb88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2020
ms.locfileid: "80633890"
---
# <a name="azure-blob-storage-input-binding-for-azure-functions"></a>Azure Blob 儲存輸入繫結以 Azure 函式

輸入繫結允許您讀取 Blob 儲存資料作為 Azure 函數的輸入。

有關設定與設定詳細資訊的資訊,請參考[概述](./functions-bindings-storage-blob.md)。

## <a name="example"></a>範例

# <a name="c"></a>[C#](#tab/csharp)

下列範例是使用一個佇列觸發程序和一個輸入 Blob 繫結的 [C# 函式](functions-dotnet-class-library.md)。 佇列訊息包含 Blob 的名稱，而函式會記錄 Blob 的大小。

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

# <a name="c-script"></a>[C# 文稿](#tab/csharp-script)

<!--Same example for input and output. -->

下列範例所示範的是使用繫結之 function.json** 檔案，以及 [C# 指令碼 (.csx)](functions-reference-csharp.md) 程式碼中的 Blob 輸入和輸出繫結。 此函式會建立文字 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 blob 名為 *[原始 blobname] -複製*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[設定](#configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

<!--Same example for input and output. -->

下列範例所示範的是 *function.json* 檔案中的 blob 輸入和輸出繫結，以及使用該繫結的 [JavaScript 程式碼](functions-reference-node.md)。 此函式會建立 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 blob 名為 *[原始 blobname] -複製*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[設定](#configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

<!--Same example for input and output. -->

下列範例所示範的是使用繫結之 function.json** 檔案，以及 [Python 指令碼](functions-reference-python.md)程式碼中的 Blob 輸入和輸出繫結。 此函式會建立 Blob 的複本。 此函式是由佇列訊息 (包含要複製的 Blob 名稱) 觸發。 新的 blob 名為 *[原始 blobname] -複製*。

在 *function.json* 檔案中，`queueTrigger` 中繼資料屬性用於指定 `path` 屬性中的 Blob 名稱：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "dataType": "binary",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "$return",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

[設定](#configuration)章節會說明這些屬性。

以下是 Python 程式碼：

```python
import logging
import azure.functions as func


def main(queuemsg: func.QueueMessage, inputblob: func.InputStream) -> func.InputStream:
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    return inputblob
```

# <a name="java"></a>[Java](#tab/java)

本區段包含下列範例：

* [HTTP 觸發程序，從查詢字串中查閱 Blob 名稱](#http-trigger-look-up-blob-name-from-query-string)
* [佇列觸發程序，接收來自佇列訊息的 Blob 名稱](#queue-trigger-receive-blob-name-from-queue-message)

#### <a name="http-trigger-look-up-blob-name-from-query-string"></a>HTTP 觸發程序，從查詢字串中查閱 Blob 名稱

 下列範例示範使用 `HttpTrigger` 註解接收參數 (其中包含 Blob 儲存體容器中的檔案名稱) 的 Java 函式。 `BlobInput` 註解接著會讀取檔案，並將其內容傳遞給函式做為 `byte[]`。

```java
  @FunctionName("getBlobSizeHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage blobSize(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    final ExecutionContext context) {
      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### <a name="queue-trigger-receive-blob-name-from-queue-message"></a>佇列觸發程序，接收來自佇列訊息的 Blob 名稱

 下列範例示範使用 `QueueTrigger` 註解接收訊息 (其中包含 Blob 儲存體容器中的檔案名稱) 的 Java 函式。 `BlobInput` 註解接著會讀取檔案，並將其內容傳遞給函式做為 `byte[]`。

```java
  @FunctionName("getBlobSize")
  @StorageAccount("Storage_Account_Connection_String")
  public void blobSize(
    @QueueTrigger(
      name = "filename", 
      queueName = "myqueue-items-sample") 
    String filename,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{queueTrigger}") 
    byte[] content,
    final ExecutionContext context) {
      context.getLogger().info("The size of \"" + filename + "\" is: " + content.length + " bytes");
  }
```

在 [Java 函式執行階段程式庫](/java/api/overview/azure/functions/runtime)中，對其值來自 Blob 的參數使用 `@BlobInput` 註釋。  此註釋可以搭配原生 Java 類型、POJO 或使用 `Optional<T>` 的可為 Null 值使用。

---

## <a name="attributes-and-annotations"></a>屬性與註解

# <a name="c"></a>[C#](#tab/csharp)

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs)。

屬性的建構函式會採用 Blob 路徑和指示讀取或寫入的 `FileAccess` 參數，如下列範例所示：

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}

```

您可以設定 `Connection` 屬性來指定要使用的儲存體帳戶，如下列範例所示：

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read, Connection = "StorageConnectionAppSetting")] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

您可以使用 `StorageAccount` 屬性來指定類別、方法或參數層級的儲存體帳戶。 有關詳細資訊,請參閱[觸發器屬性和註解](./functions-bindings-storage-blob-trigger.md#attributes-and-annotations)。

# <a name="c-script"></a>[C# 文稿](#tab/csharp-script)

C# 文稿不支援屬性。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript 不支援屬性。

# <a name="python"></a>[Python](#tab/python)

Python 不支援屬性。

# <a name="java"></a>[Java](#tab/java)

該`@BlobInput`屬性允許您訪問觸發函數的 blob。 如果使用有屬性的位元陣陣陣,則設定為`dataType``binary`。 有關詳細資訊,請參閱[輸入範例](#example)。

---

## <a name="configuration"></a>組態

下表介紹了您在*函數.json*`Blob`檔和 屬性中設置的綁定配置屬性。

|function.json 屬性 | 屬性內容 |描述|
|---------|---------|----------------------|
|**型別** | n/a | 必須設為 `blob`。 |
|**direction** | n/a | 必須設為 `in`。 例外狀況在[使用方式](#usage)一節中會加以說明。 |
|**名稱** | n/a | 表示函式程式碼中 Blob 的變數名稱。|
|**path** |**BlobPath** | blob 的路徑。 |
|**連線** |**連線**| 包含要用於此繫定的[儲存連接字串](../storage/common/storage-configure-connection-string.md)的應用程式設定的名稱。 如果應用程式設定名稱是以「AzureWebJobs」開頭，於此僅能指定名稱的其餘部分。 例如,如果設置為`connection`「我的存儲」,函數運行時將查找名為「AzureWebJobsMyStorage」的應用設置。 如果您將 `connection` 保留空白，則函式執行階段會使用應用程式設定中名稱為 `AzureWebJobsStorage` 的預設儲存體連接字串。<br><br>連接字串必須為一般用途的儲存體帳戶，不可為[僅限 Blob 的儲存體帳戶](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|
|n/a | **存取** | 指出您是否將讀取或寫入。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>使用量

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-input-usage.md](../../includes/functions-bindings-blob-storage-input-usage.md)]

# <a name="c-script"></a>[C# 文稿](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-input-usage.md](../../includes/functions-bindings-blob-storage-input-usage.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

使用`context.bindings.<NAME>`與`<NAME>`*函數.json*中定義的值匹配的位置訪問 blob 數據。

# <a name="python"></a>[Python](#tab/python)

通過鍵入為[InputStream](https://docs.microsoft.com/python/api/azure-functions/azure.functions.inputstream?view=azure-python)的參數存取 blob 資料。 有關詳細資訊,請參閱[輸入範例](#example)。

# <a name="java"></a>[Java](#tab/java)

該`@BlobInput`屬性允許您訪問觸發函數的 blob。 如果使用有屬性的位元陣陣陣,則設定為`dataType``binary`。 有關詳細資訊,請參閱[輸入範例](#example)。

---

## <a name="next-steps"></a>後續步驟

- [Blob 儲存資料變更時執行函數](./functions-bindings-storage-blob-trigger.md)
- [從函數寫入 blob 儲存資料](./functions-bindings-storage-blob-output.md)

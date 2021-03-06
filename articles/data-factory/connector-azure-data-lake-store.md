---
title: 將資料複製到或從 Azure 資料存儲第 1 代
description: 了解如何使用 Data Factory 將資料從支援的來源資料存放區複製到 Azure Data Lake Store，或從 Data Lake Store 複製到支援的接收存放區。
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/12/2020
ms.openlocfilehash: a8ba8b212a504a8f8e4e29fbd50126189998e81a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80065473"
---
# <a name="copy-data-to-or-from-azure-data-lake-storage-gen1-using-azure-data-factory"></a>使用 Azure 資料工廠將資料複製到或從 Azure 資料存儲第 1 代

> [!div class="op_single_selector" title1="選擇正在使用的 Azure 資料工廠版本："]
> * [版本 1](v1/data-factory-azure-datalake-connector.md)
> * [當前版本](connector-azure-data-lake-store.md)

本文概述了如何將資料複製到 Azure 資料存儲第 1 代。 若要了解 Azure Data Factory，請閱讀[簡介文章](introduction.md)。

## <a name="supported-capabilities"></a>支援的功能

此 Azure 資料存儲庫第 1 代連接器支援以下活動：

- 使用[支援的源/接收器矩陣](copy-activity-overview.md)[複製活動](copy-activity-overview.md) 
- [映射資料流程](concepts-data-flow-overview.md)
- [查找活動](control-flow-lookup-activity.md)
- [獲取中繼資料活動](control-flow-get-metadata-activity.md)
- [刪除活動](delete-activity.md)

具體來說，使用此連接器，您可以：

- 使用以下身份驗證方法之一複製檔：Azure 資源的服務主體或託管標識。
- 複製檔作為或解析或生成檔與[支援的檔案格式和壓縮編解碼器](supported-file-formats-and-compression-codecs.md)。
- 複製到 Azure 資料湖存儲 Gen2 時[保留 ACL。](#preserve-acls-to-data-lake-storage-gen2)

> [!IMPORTANT]
> 如果使用自託管的集成運行時複製資料，請將公司防火牆配置為允許進出站流量到`<ADLS account name>.azuredatalakestore.net`埠 443`login.microsoftonline.com/<tenant>/oauth2/token`和埠 443 上。 後者是 Azure 安全性權杖服務，整合執行階段需要與之通訊才能取得權杖。

## <a name="get-started"></a>開始使用

> [!TIP]
> 有關如何使用 Azure 資料湖存儲連接器的演練，請參閱[將資料載入到 Azure 資料湖存儲中](load-azure-data-lake-store.md)。

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

以下各節提供有關用於定義特定于 Azure 資料湖存儲的資料工廠實體的屬性的資訊。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Azure Data Lake Store 已連結服務支援的屬性：

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | `type` 屬性必須設為 **AzureDataLakeStore**。 | 是 |
| dataLakeStoreUri | Azure Data Lake Store 帳戶相關資訊。 此資訊會採用下列其中一種格式：`https://[accountname].azuredatalakestore.net/webhdfs/v1` 或 `adl://[accountname].azuredatalakestore.net/`。 | 是 |
| subscriptionId | Data Lake Store 帳戶所屬的 Azure 訂用帳戶識別碼。 | 接收 (Sink) 的必要項目 |
| resourceGroupName | Data Lake Store 帳戶所屬的 Azure 資源群組名稱。 | 接收 (Sink) 的必要項目 |
| connectVia | 用於連接到資料存儲的[集成運行時](concepts-integration-runtime.md)。 如果資料存儲位於私人網路絡中，則可以使用 Azure 集成運行時或自託管集成運行時。 如果未指定此屬性，則使用預設的 Azure 集成運行時。 |否 |

### <a name="use-service-principal-authentication"></a>使用服務主體驗證

要使用服務主體身份驗證，請按照以下步驟操作。

1. 在 Azure 活動目錄中註冊應用程式實體，並授予其對資料湖存儲的存取權限。 如需詳細的步驟，請參閱[服務對服務驗證](../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。 請記下以下的值，您可以使用這些值來定義連結服務：

    - 應用程式識別碼
    - 應用程式金鑰
    - 租用戶識別碼

2. 授予服務主體適當的許可權。 請參閱[有關 Azure 資料湖存儲 Gen1 中訪問控制項](../data-lake-store/data-lake-store-access-control.md#common-scenarios-related-to-permissions)中許可權在資料湖存儲 Gen1 中的工作方式的示例。

    - **作為源**：在**資料資源管理器** > **訪問**中，至少授予所有上游資料夾（包括根）**的執行**許可權，以及檔案複製的**讀取**許可權。 您可以選擇新增到 [此資料夾及所有子系]**** 以供遞迴，並新增成為**存取權限和預設權限項目**。 對帳戶級存取控制 （IAM） 沒有要求。
    - **作為接收器**：在**資料資源管理器** > **訪問**中，至少對所有上游資料夾（包括根 **）授予執行**許可權，以及接收器資料夾的**寫入**許可權。 您可以選擇新增到 [此資料夾及所有子系]**** 以供遞迴，並新增成為**存取權限和預設權限項目**。 如果使用 Azure 集成運行時進行複製（源和接收器都在雲中），則至少授予**Reader**角色，以便讓資料工廠檢測資料湖存儲的區域。 如果您想要避免此 IAM 角色，請以 Data Lake Store 的位置明確地[建立 Azure 整合執行階段](create-azure-integration-runtime.md#create-azure-ir)。 例如，如果資料湖存儲位於西歐，請創建 Azure 集成運行時，位置設置為"西歐"。 將它們關聯到資料湖存儲連結服務中，如以下示例所示。

以下是支援的屬性：

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| servicePrincipalId | 指定應用程式的用戶端識別碼。 | 是 |
| servicePrincipalKey | 指定應用程式的金鑰。 將此欄位標記為 `SecureString`，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| tenant | 指定應用程式所在的租戶資訊，如功能變數名稱或租戶 ID。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | 是 |

**例子：**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a>使用 Azure 資源的受控識別驗證

資料處理站可與 [Azure 資源的受控識別](data-factory-service-identity.md)相關聯，後者表示特定的資料處理站。 您可以直接使用此託管標識進行資料湖存儲身份驗證，類似于使用您自己的服務主體。 這可以讓這個指定的處理站存取及複製資料至 Data Lake Store，或從 Data Lake Store 存取及複製資料。

要使用託管標識進行 Azure 資源身份驗證，請按照以下步驟操作。

1. 通過複製與工廠一起生成的"服務標識應用程式 ID"的值來[檢索資料工廠託管標識資訊](data-factory-service-identity.md#retrieve-managed-identity)。

2. 授予對資料湖存儲的託管標識存取權限。 請參閱[有關 Azure 資料湖存儲 Gen1 中訪問控制項](../data-lake-store/data-lake-store-access-control.md#common-scenarios-related-to-permissions)中許可權在資料湖存儲 Gen1 中的工作方式的示例。

    - **作為源**：在**資料資源管理器** > **訪問**中，至少授予所有上游資料夾（包括根）**的執行**許可權，以及檔案複製的**讀取**許可權。 您可以選擇新增到 [此資料夾及所有子系]**** 以供遞迴，並新增成為**存取權限和預設權限項目**。 對帳戶級存取控制 （IAM） 沒有要求。
    - **作為接收器**：在**資料資源管理器** > **訪問**中，至少對所有上游資料夾（包括根 **）授予執行**許可權，以及接收器資料夾的**寫入**許可權。 您可以選擇新增到 [此資料夾及所有子系]**** 以供遞迴，並新增成為**存取權限和預設權限項目**。 如果使用 Azure 集成運行時進行複製（源和接收器都在雲中），則至少授予**Reader**角色，以便讓資料工廠檢測資料湖存儲的區域。 如果您想要避免此 IAM 角色，請以 Data Lake Store 的位置明確地[建立 Azure 整合執行階段](create-azure-integration-runtime.md#create-azure-ir)。 將它們關聯到資料湖存儲連結服務中，如以下示例所示。

在 Azure Data Factory 中，除了 Data Lake Store 的一般資訊，您不需要在連結服務中指定任何屬性。

**例子：**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

在基於格式的資料集中的設置下`location`，Azure 資料湖存儲 Gen1 支援以下屬性：

| 屬性   | 描述                                                  | 必要 |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | 資料集`location`下的類型屬性必須設置為**AzureDataLakeStoreLocation**。 | 是      |
| folderPath | 資料夾的路徑。 如果要使用萬用字元篩選資料夾，請跳過此設置並在活動源設置中指定它。 | 否       |
| fileName   | 給定資料夾 Path 下的檔案名。 如果要使用萬用字元篩選檔，請跳過此設置並在活動源設置中指定它。 | 否       |

**例子：**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<ADLS Gen1 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureDataLakeStoreLocation",
                "folderPath": "root/folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)。 本節提供 Azure Data Lake Store 來源和接收器所支援的屬性清單。

### <a name="azure-data-lake-store-as-source"></a>Azure Data Lake Store 作為來源

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

在基於格式的副本源中的設置下`storeSettings`，Azure 資料湖存儲 Gen1 支援以下屬性：

| 屬性                 | 描述                                                  | 必要                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | 下`storeSettings`的類型屬性必須設置為**AzureDataLakeStoreRead設置**。 | 是                                           |
| 遞迴                | 指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 當遞迴設置為 true 並且接收器是檔案為儲存基礎時，不會在接收器上複製或創建空資料夾或子資料夾。 允許的值**為 true（** 預設值）和**false**。 | 否                                            |
| 萬用字元資料夾路徑       | 具有萬用字元的資料夾路徑，用於篩選源資料夾。 <br>允許的通配`*`符是（匹配零個或多個字元）`?`和（匹配零個或多個字元）。 用於`^`轉義，如果實際資料夾名稱有萬用字元或此逸出字元裡面。 <br>如需更多範例，請參閱[資料夾和檔案篩選範例](#folder-and-file-filter-examples)。 | 否                                            |
| 萬用字元檔案名稱         | 給定資料夾 Path/萬用字元下具有萬用字元的檔案名用於篩選原始檔案。 <br>允許的通配`*`符是（匹配零個或多個字元）`?`和（匹配零個或多個字元）。 用於`^`轉義，如果實際資料夾名稱有萬用字元或此逸出字元裡面。 如需更多範例，請參閱[資料夾和檔案篩選範例](#folder-and-file-filter-examples)。 | `fileName`如果資料集中未指定，則為"是" |
| modifiedDatetimeStart    | 基於屬性"上次修改"的檔篩選器。 如果檔上次修改的時間在 和`modifiedDatetimeStart``modifiedDatetimeEnd`之間的時間範圍內，則選擇這些檔。 時間以"2018-12-01T05：00：00：00Z"的形式應用於 UTC 時區。 <br> 屬性可以是 Null，這意味著沒有檔案屬性篩選器應用於資料集。 當`modifiedDatetimeStart`具有日期時間值但`modifiedDatetimeEnd`為 Null 時，表示將選擇其上次修改屬性大於或等於日期時間值的檔。 當`modifiedDatetimeEnd`具有日期時間值但`modifiedDatetimeStart`為 Null 時，表示選擇上次修改屬性小於日期時間值的檔。 | 否                                            |
| modifiedDatetimeEnd      | 同上。                                               | 否                                            |
| maxConcurrentConnections | 要同時連接到存儲存儲的連接數。 僅當要將併發連接限制到資料存儲時，才指定。 | 否                                            |

**例子：**

```json
"activities":[
    {
        "name": "CopyFromADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AzureDataLakeStoreReadSettings",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-data-lake-store-as-sink"></a>Azure Data Lake Store 作為接收器

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

在基於格式的副本接收器中的設置下`storeSettings`，Azure 資料湖存儲 Gen1 支援以下屬性：

| 屬性                 | 描述                                                  | 必要 |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | 下`storeSettings`的類型屬性必須設置為**AzureDataLakeStoreWrite設置**。 | 是      |
| copyBehavior             | 當來源是來自檔案型資料存放區的檔案時，會定義複製行為。<br/><br/>允許的值包括：<br/><b>- 保留層次結構（預設）：</b>保留目的檔案夾中的檔層次結構。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><b>- FlattenHierarchy</b>：來自來源資料夾的所有檔案都在目標資料夾的第一層中。 目標檔案會有自動產生的名稱。 <br/><b>- 合併檔</b>：將源資料夾中的所有檔合併到一個檔。 若已指定檔案名稱，合併檔案的名稱會是指定的名稱。 否則，就會是自動產生的檔案名稱。 | 否       |
| 到期日期時間 | 指定寫入檔的過期時間。 時間以"2020-03-01T08：00：00：00Z"的格式應用於 UTC 時間。 預設情況下，它是 Null，這意味著寫入的檔永遠不會過期。 | 否 |
| maxConcurrentConnections | 要同時連接到資料存儲的連接數。 僅當要將併發連接限制到資料存儲時，才指定。 | 否       |

**例子：**

```json
"activities":[
    {
        "name": "CopyToADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Parquet output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "ParquetSink",
                "storeSettings":{
                    "type": "AzureDataLakeStoreWriteSettings",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>資料夾和檔案篩選範例

本節描述含有萬用字元篩選之資料夾路徑和檔案名稱所產生的行為。

| folderPath | fileName | 遞迴 | 源資料夾結構和篩選結果（檢索**粗體**檔）|
|:--- |:--- |:--- |:--- |
| `Folder*` | （空，使用預設值） | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | （空，使用預設值） | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**檔3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**檔3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="examples-of-behavior-of-the-copy-operation"></a>複製作業的行為範例

本節說明 `recursive` 和 `copyBehavior` 值在不同組合的情況下，複製作業所產生的行為。

| 遞迴 | copyBehavior | 來源資料夾結構 | 產生的目標 |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以與來源相同的結構，建立目標 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File1 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File2 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File3 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File4 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File5 自動產生的名稱 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 = File2 = File3 = File4 = File5 內容合併到一個檔中，並帶有一個自動生成的檔案名。 |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>未選取包含 File3、File4 和 File5 的子資料夾1。 |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File1 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File2 自動產生的名稱<br/><br/>未選取包含 File3、File4 和 File5 的子資料夾1。 |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 = File2 內容合併到具有自動生成的檔案名的一個檔中。 針對 File1 自動產生的名稱<br/><br/>未選取包含 File3、File4 和 File5 的子資料夾1。 |

## <a name="preserve-acls-to-data-lake-storage-gen2"></a>將 ACL 保留到資料存儲庫第 2 代

>[!TIP]
>要將資料從 Azure 資料存儲 Gen1 複製到 Gen2，請參閱[將資料從 Azure 資料存儲第 1 代複製到 Azure 資料工廠](load-azure-data-lake-storage-gen2-from-gen1.md)的第 2 代資料，瞭解演練和最佳實踐。

如果要在從資料存儲庫第 1 代升級到資料存儲第 1 代時複製存取控制清單 （ACL） 以及資料檔案，請參閱[保留來自資料存儲湖存儲 Gen1 的 ACL。](copy-activity-preserve-metadata.md#preserve-acls)

## <a name="mapping-data-flow-properties"></a>映射資料流程屬性

在映射資料流程中轉換資料時，可以從 Azure 資料存儲第 1 代（JSON、Avro、分隔文本或 Parquet 格式）讀取和寫入檔。 有關詳細資訊，請參閱映射資料流程功能中的[源轉換](data-flow-source.md)和[接收器轉換](data-flow-sink.md)。

### <a name="source-transformation"></a>源轉換

在源轉換中，可以從 Azure 資料湖存儲 Gen1 中的容器、資料夾或單個檔進行讀取。 "**源"選項**選項卡允許您管理檔讀取的方式。 

![源選項](media/data-flow/sourceOptions1.png "源選項")

**萬用字元路徑：** 使用萬用字元模式將指示 ADF 在單個源轉換中逐一查看每個匹配的資料夾和檔。 這是處理單個流中的多個檔的有效方法。 使用將滑鼠懸停在現有萬用字元模式上時出現的 + 符號添加多個萬用字元匹配模式。

從源容器中選擇一系列與模式匹配的檔。 只能在資料集中指定容器。 因此，萬用字元路徑還必須包括根資料夾中的資料夾路徑。

萬用字元示例：

* ```*```表示任意一組字元
* ```**```表示遞迴目錄嵌套
* ```?```替換一個字元
* ```[]```匹配括弧中更多字元之一

* ```/data/sales/**/*.csv```獲取 /資料/銷售下的所有 csv 檔
* ```/data/sales/20??/**/```獲取 20 世紀的所有檔
* ```/data/sales/*/*/*.csv```獲取 csv 檔下 /資料/銷售兩個級別
* ```/data/sales/2004/*/12/[XY]1?.csv```在 2004 年 12 月獲取所有 csv 檔，以 X 或 Y 開頭，以兩位數位開頭

**分區根路徑：** 如果檔源中的資料夾已採用```key=value```格式（例如，year_2019）進行分區，則可以將該分區資料夾樹的頂層分配給資料流程資料流程中的列名稱。

首先，設置萬用字元以包括分區資料夾的所有路徑以及您希望讀取的葉檔。

![分區原始檔案設置](media/data-flow/partfile2.png "分區檔設置")

使用分區根路徑設置定義資料夾結構的頂層。 當您通過資料預覽查看資料內容時，您將看到 ADF 將添加在每個資料夾級別中找到的已解析分區。

![分區根路徑](media/data-flow/partfile1.png "分區根路徑預覽")

**檔案清單：** 這是一個檔集。 創建包含要處理的相對路徑檔的清單的文字檔。 指向此文字檔。

**要存儲檔案名的列：** 將原始檔案的名稱存儲在資料中的列中。 在此處輸入新的列名以存儲檔案名字串。

**完成後：** 選擇在資料流程運行後對原始檔案不執行任何操作、刪除原始檔案或移動原始檔案。 移動的路徑是相對的。

要將原始檔案移到處理後的另一個位置，首先選擇檔操作的"移動"。 然後，設置"from"目錄。 如果您沒有為路徑使用任何萬用字元，則"from"設置將與源資料夾位於同一資料夾。

如果您有具有萬用字元的源路徑，則語法如下所示：

```/data/sales/20??/**/*.csv```

您可以將"從"指定為

```/data/sales```

和"到"作為

```/backup/priorSales```

在這種情況下，在 /data/sales 下獲取的所有檔將移動到 /備份/優先銷售。

> [!NOTE]
> 僅當從管道運行（管道調試或執行運行）啟動資料流程時，檔操作才會運行，該流使用管道中的執行資料流程活動。 檔操作*不在*資料流程偵錯模式下運行。

**按上次修改進行篩選：** 您可以通過指定上次修改檔的日期範圍來篩選處理的檔。 所有日期時間均以 UTC 表示。 

### <a name="sink-properties"></a>接收器屬性

在接收器轉換中，可以寫入 Azure 資料湖存儲 Gen1 中的容器或資料夾。 通過 **"設置"** 選項卡，您可以管理檔的寫入方式。

![接收器選項](media/data-flow/file-sink-settings.png "接收器選項")

**清除資料夾：** 確定在寫入資料之前是否清除目的檔案夾。

**檔案名選項：** 確定目的檔案在目的檔案夾中的命名方式。 檔案名選項為：
   * **預設值**：允許 Spark 根據 PART 預設值命名檔。
   * **模式**：輸入一種模式，以枚舉每個分區的輸出檔案。 例如，**貸款\n\csv**將創建貸款1.csv，貸款2.csv，等等。
   * **每個分區**：每個分區輸入一個檔案名。
   * **作為列中的資料**：將輸出檔案設置為列的值。 路徑相對於資料集容器，而不是目的檔案夾。 如果資料集中包含資料夾路徑，則該路徑將被覆蓋。
   * **輸出到單個檔**：將分區的輸出檔案合併到單個命名檔中。 路徑與資料集資料夾相關。 請注意，te 合併作業可能會根據節點大小失敗。 不建議對大型資料集使用此選項。

**報價所有：** 確定是否將報價中的所有值括起來

## <a name="lookup-activity-properties"></a>查找活動屬性

要瞭解有關屬性的詳細資訊，請檢查[查找活動](control-flow-lookup-activity.md)。

## <a name="getmetadata-activity-properties"></a>獲取中繼資料活動屬性

要瞭解有關屬性的詳細資訊，請選中[獲取中繼資料活動](control-flow-get-metadata-activity.md) 

## <a name="delete-activity-properties"></a>刪除活動屬性

要瞭解有關屬性的詳細資訊，請選中["刪除活動](delete-activity.md)"

## <a name="legacy-models"></a>舊模型

>[!NOTE]
>以下型號仍支援以下型號，以便進行向後相容性。 建議您今後使用上述各節中提及的新模型，並且 ADF 創作 UI 已切換到生成新模型。

### <a name="legacy-dataset-model"></a>舊資料集模型

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設置為**AzureDataLakeStoreFile**。 |是 |
| folderPath | Data Lake Store 中的資料夾路徑。 若未指定，它會指向根。 <br/><br/>支援萬用字元篩選器。 允許的通配`*`符是（匹配零個或多個字元）`?`和（匹配零個或多個字元）。 用於`^`轉義，如果實際資料夾名稱有萬用字元或此逸出字元裡面。 <br/><br/>例如：根資料夾/子資料夾/。 如需更多範例，請參閱[資料夾和檔案篩選範例](#folder-and-file-filter-examples)。 |否 |
| fileName | 指定"資料夾路徑"下檔的名稱或萬用字元篩選器。 若未指定此屬性的值，資料集就會指向資料夾中的所有檔案。 <br/><br/>對於篩選器，允許的萬用字元`*`為（匹配零個或多個字元）和`?`（匹配零個或多個字元）。<br/>- 範例 1：`"fileName": "*.csv"`<br/>- 範例 2：`"fileName": "???20180427.txt"`<br/>用於`^`轉義，如果實際檔案名有萬用字元或此逸出字元裡面。<br/><br/>當未為輸出資料集指定檔案名並在活動接收器中未指定**保留層次結構**時，複製活動將自動生成具有以下模式的檔案名："*資料"。活動運行 ID GUID]。[GUID 如果扁平層次結構]。[格式（如果已配置）。"[壓縮如果配置]*"，例如，"資料.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz"。 如果使用表名而不是查詢從表格源複製，則名稱模式為 *"[表名]"。格式*。[壓縮（如果配置）"，"* 例如，"MyTable.csv"。 |否 |
| modifiedDatetimeStart | 基於屬性"上次修改"的檔篩選器。 如果檔上次修改的時間在 和`modifiedDatetimeStart``modifiedDatetimeEnd`之間的時間範圍內，則選擇這些檔。 時間以"2018-12-01T05：00：00：00Z"的形式應用於 UTC 時區。 <br/><br/> 當您想要使用大量檔進行檔篩選器時，啟用此設置會影響資料移動的總體性能。 <br/><br/> 屬性可以是 Null，這意味著沒有檔案屬性篩選器應用於資料集。 當`modifiedDatetimeStart`具有日期時間值但`modifiedDatetimeEnd`為 Null 時，表示將選擇其上次修改屬性大於或等於日期時間值的檔。 當`modifiedDatetimeEnd`具有日期時間值但`modifiedDatetimeStart`為 Null 時，表示選擇上次修改屬性小於日期時間值的檔。| 否 |
| modifiedDatetimeEnd | 基於屬性"上次修改"的檔篩選器。 如果檔上次修改的時間在 和`modifiedDatetimeStart``modifiedDatetimeEnd`之間的時間範圍內，則選擇這些檔。 時間以"2018-12-01T05：00：00：00Z"的形式應用於 UTC 時區。 <br/><br/> 當您想要使用大量檔進行檔篩選器時，啟用此設置會影響資料移動的總體性能。 <br/><br/> 屬性可以是 Null，這意味著沒有檔案屬性篩選器應用於資料集。 當`modifiedDatetimeStart`具有日期時間值但`modifiedDatetimeEnd`為 Null 時，表示將選擇其上次修改屬性大於或等於日期時間值的檔。 當`modifiedDatetimeEnd`具有日期時間值但`modifiedDatetimeStart`為 Null 時，表示選擇上次修改屬性小於日期時間值的檔。| 否 |
| format | 如果要在基於檔的商店（二進位副本）之間複製檔，請跳過輸入資料集和輸出資料集定義中的格式部分。<br/><br/>如果您想要以特定格式來剖析或產生檔案，以下是支援的檔案格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat** 和 **ParquetFormat**。 將 [format]**** 下的 [type]**** 屬性設定為下列其中一個值。 如需詳細資訊，請參閱[文字格式](supported-file-formats-and-compression-codecs-legacy.md#text-format)、[JSON 格式](supported-file-formats-and-compression-codecs-legacy.md#json-format)、[Avro 格式](supported-file-formats-and-compression-codecs-legacy.md#avro-format)、[Orc 格式](supported-file-formats-and-compression-codecs-legacy.md#orc-format)和 [Parquet 格式](supported-file-formats-and-compression-codecs-legacy.md#parquet-format)小節。 |否 (僅適用於二進位複製案例) |
| compression | 指定此資料的壓縮類型和層級。 如需詳細資訊，請參閱[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs-legacy.md#compression-support)。<br/>支援的類型為：GZip****、Deflate****、BZip2**** 及 ZipDeflate****。<br/>支援的層級為 **Optimal** 和 **Fastest**。 |否 |

>[!TIP]
>若要複製資料夾下的所有檔案，請只指定 **folderPath**。<br>要複製具有特定名稱的單個檔，請指定資料夾部分的**資料夾 Path**和具有檔案名**的檔案名**。<br>要在資料夾下複製檔子集，請使用資料夾部件指定**資料夾 Path，** 使用萬用字元篩選器**指定檔案名**。 

**例子：**

```json
{
    "name": "ADLSDataset",
    "properties": {
        "type": "AzureDataLakeStoreFile",
        "linkedServiceName":{
            "referenceName": "<ADLS linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "datalake/myfolder/",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

### <a name="legacy-copy-activity-source-model"></a>舊複製活動源模型

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 必須`type`將複製活動源的屬性設置為**AzureDataLakeStoreSource**。 |是 |
| 遞迴 | 指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 當`recursive`設置為 true 並且接收器是檔案為儲存基礎時，不會在接收器上複製或創建空資料夾或子資料夾。 允許的值**為 true（** 預設值）和**false**。 | 否 |
| maxConcurrentConnections | 要同時連接到資料存儲的連接數。 僅當要將併發連接限制到資料存儲時，才指定。 | 否 |

**例子：**

```json
"activities":[
    {
        "name": "CopyFromADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ADLS Gen1 input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureDataLakeStoreSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="legacy-copy-activity-sink-model"></a>舊複製活動接收器模型

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 複製`type`活動接收器的屬性必須設置為**AzureDataLakeStoreSink**。 |是 |
| copyBehavior | 當來源是來自檔案型資料存放區的檔案時，會定義複製行為。<br/><br/>允許的值包括：<br/><b>- 保留層次結構（預設）：</b>保留目的檔案夾中的檔層次結構。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><b>- FlattenHierarchy</b>：來自來源資料夾的所有檔案都在目標資料夾的第一層中。 目標檔案會有自動產生的名稱。 <br/><b>- 合併檔</b>：將源資料夾中的所有檔合併到一個檔。 若已指定檔案名稱，合併檔案的名稱會是指定的名稱。 否則，會自動產生檔案名稱。 | 否 |
| maxConcurrentConnections | 要同時連接到資料存儲的連接數。 僅當要將併發連接限制到資料存儲時，才指定。 | 否 |

**例子：**

```json
"activities":[
    {
        "name": "CopyToADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<ADLS Gen1 output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDataLakeStoreSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

## <a name="next-steps"></a>後續步驟

如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。

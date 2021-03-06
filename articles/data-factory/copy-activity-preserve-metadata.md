---
title: 使用 Azure 資料工廠中的複製活動保留中繼資料和 ACL
description: 瞭解如何使用 Azure 資料工廠中的複製活動在複製期間保留中繼資料和 ACL。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 03/24/2020
ms.author: jingwang
ms.openlocfilehash: b73cd73a18d286f221c7be2c624719e1d23d7c06
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80153823"
---
#  <a name="preserve-metadata-and-acls-using-copy-activity-in-azure-data-factory"></a>使用 Azure 資料工廠中的複製活動保留中繼資料和 ACL

使用 Azure 資料工廠複製活動將資料從源複製到接收器時，在以下方案中，還可以保留中繼資料和 ACL。

## <a name="preserve-metadata-for-lake-migration"></a><a name="preserve-metadata"></a>保留湖泊遷移的中繼資料

將資料從一個資料湖遷移到另一個資料湖（包括[Amazon S3、Azure](connector-amazon-simple-storage-service.md) [Blob](connector-azure-blob-storage.md)和[Azure 資料存儲 Gen2）](connector-azure-data-lake-storage.md)時，可以選擇保留檔中繼資料和資料。

複製活動支援在資料複製期間保留以下屬性：

- **所有客戶指定的中繼資料** 
- 和以下**五個數據存儲內置系統屬性**`contentType`： （`contentLanguage`亞馬遜S3除外）， `contentEncoding` `contentDisposition`， `cacheControl`。

當您按原樣從 Amazon S3/Azure 資料存儲湖存儲 Gen2/Azure Blob 複製到具有二進位格式的 Azure 資料湖存儲 Gen2/Azure Blob 時，可以在"**複製活動** > **設置"** 選項卡上找到活動創作中的 **"保留**"選項或"複製資料工具中的**設置"** 頁。

![複製活動保留中繼資料](./media/copy-activity-preserve-metadata/copy-activity-preserve-metadata.png)

下面是複製活動 JSON 配置的示例（請參見`preserve`）： 

```json
"activities":[
    {
        "name": "CopyAndPreserveMetadata",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AmazonS3ReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "Attributes"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset Amazon S3/Azure Blob/ADLS Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset for Azure Blob/ADLS Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="preserve-acls-from-data-lake-storage-gen1gen2-to-gen2"></a><a name="preserve-acls"></a>保留從資料存儲湖存儲第 1 代/第 2 代到第 2 代的 ACL

當您從 Azure 資料存儲庫第 1 代升級到第 2 代或在 ADLS Gen2 之間複製資料時，可以選擇保留 POSIX 存取控制清單 （ACL） 以及資料檔案。 有關存取控制的詳細資訊，請參閱[Azure 資料湖存儲 Gen1 中的存取控制](../data-lake-store/data-lake-store-access-control.md)以及[Azure 資料湖存儲 Gen2 中的存取控制](../storage/blobs/data-lake-storage-access-control.md)。

複製活動支援在資料複製期間保留以下類型的 ACL。 您可以選擇一個或多個類型：

- **ACL**：複製並保留檔和目錄上的 POSIX 存取控制清單。 它將完整的現有 ACL 從源複製到接收器。 
- **擁有者**：複製並保留檔和目錄的擁有使用者。 需要超級使用者訪問接收器資料存儲第 2 代。
- **組**：複製並保留擁有的檔和目錄組。 需要超級使用者訪問接收器資料存儲 Gen2 或擁有的使用者（如果擁有的使用者也是目標群組的成員）。

如果指定從資料夾複製，資料工廠將複製該給定資料夾的 ACL 以及該資料夾下的檔和目錄（如果`recursive`設置為 true）。 如果指定從單個檔案複製，則複製該檔上的 ACL。

>[!NOTE]
>當您使用 ADF 保留從資料存儲湖存儲 Gen1/Gen2 到 Gen2 的 ACL 時，接收器 Gen2 對應資料夾/檔上的現有 ACL 將被覆蓋。

>[!IMPORTANT]
>當您選擇保留 ACL 時，請確保授予足夠高的許可權，以便資料工廠對接收器資料存儲 Gen2 帳戶進行操作。 例如，使用帳戶金鑰身份驗證或將存儲 Blob 資料擁有者角色指派給服務主體或託管標識。

當您將源配置為具有二進位格式或二進位副本選項的資料存儲湖存儲 Gen1/Gen2，並且以二進位格式或二進位複製選項作為資料存儲湖存儲 Gen2 接收器時，您可以在"複製資料工具**中的設置"** 頁上或活動創作的 **"複製活動** > **設置"** 選項卡上找到 **"保留**"選項。

![資料湖存儲第 1 代/第 2 代保留 ACL](./media/connector-azure-data-lake-storage/adls-gen2-preserve-acl.png)

下面是複製活動 JSON 配置的示例（請參見`preserve`）： 

```json
"activities":[
    {
        "name": "CopyAndPreserveACLs",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AzureDataLakeStoreReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "ACL",
                "Owner",
                "Group"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen1/Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="next-steps"></a>後續步驟

請參閱其他複製活動文章：

- [複製活動概述](copy-activity-overview.md)
- [複製活動效能](copy-activity-performance.md)

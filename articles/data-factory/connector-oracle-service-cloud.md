---
title: 從 Oracle 服務雲複製資料(預覽)
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 Oracle Service Cloud 複製到支援的接收資料存放區。
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2019
ms.openlocfilehash: e8e8b24aba5daa421c1840bb7164ba09e683981a
ms.sourcegitcommit: a53fe6e9e4a4c153e9ac1a93e9335f8cf762c604
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/09/2020
ms.locfileid: "80991853"
---
# <a name="copy-data-from-oracle-service-cloud-using-azure-data-factory-preview"></a>使用 Azure Data Factory 從 Oracle Service Cloud 複製資料 (預覽)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Oracle Service Cloud 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

> [!IMPORTANT]
> 此連接器目前為預覽版。 您可以親身體驗並提供意見反應。 如果您需要依賴解決方案中的預覽連接器，請連絡 [Azure 支援](https://azure.microsoft.com/support/)。

## <a name="supported-capabilities"></a>支援的功能

以下活動支援此 Oracle 服務雲端連線器:

- 使用[支援的來源/接收器矩陣](copy-activity-overview.md)[複製活動](copy-activity-overview.md)
- [尋找活動](control-flow-lookup-activity.md)

您可以將資料從 Oracle Service Cloud 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Oracle Service Cloud 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是支援 Oracle Service Cloud 連結服務的屬性：

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**OracleServiceCloud** | 是 |
| 主機 | Oracle Service Cloud 執行個體的 URL。  | 是 |
| username | 您用來存取 Oracle Service Cloud 伺服器的使用者名稱。  | 是 |
| 密碼 | 密碼，與您在使用者名稱金鑰中提供的使用者名稱相對應。 您可以選擇將這個欄位標記為 SecureString 以將它安全地儲存在 ADF，或將密碼儲存在 Azure Key Vault，然後在執行複製資料時，讓 ADF 複製活動從該處提取 - 請參閱[將認證儲存在 Key Vault](store-credentials-in-key-vault.md) 以進一步了解。 | 是 |
| useEncryptedEndpoints | 指定是否使用 HTTPS 來加密資料來源端點。 預設值為 true。  | 否 |
| useHostVerification | 指定在透過 TLS 連接時,是否要求伺服器證書中的主機名與伺服器的主機名匹配。 預設值為 true。  | 否 |
| usePeerVerification | 指定在透過 TLS 連接時是否驗證伺服器的標識。 預設值為 true。  | 否 |

**範例:**

```json
{
    "name": "OracleServiceCloudLinkedService",
    "properties": {
        "type": "OracleServiceCloud",
        "typeProperties": {
            "host" : "<host>",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            },
            "useEncryptedEndpoints" : true,
            "useHostVerification" : true,
            "usePeerVerification" : true,
        }
    }
}

```

## <a name="dataset-properties"></a>資料集屬性

有關可用於定義數據集的節和屬性的完整清單,請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Oracle Service Cloud 資料集所支援的屬性清單。

若要從 Oracle Service Cloud 複製資料，請將資料集的類型屬性設定為 **OracleServiceCloudObject**。 以下是支援的屬性：

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為 **:OracleServiceCloudObject** | 是 |
| tableName | 資料表的名稱。 | 否 (如果已指定活動來源中的「查詢」) |

**範例**

```json
{
    "name": "OracleServiceCloudDataset",
    "properties": {
        "type": "OracleServiceCloudObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<OracleServiceCloud linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}

```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Oracle Service Cloud 來源所支援的屬性清單。

### <a name="oracle-service-cloud-as-source"></a>以 Oracle Service Cloud 作為來源

若要從 Oracle Service Cloud 複製資料，請將複製活動中的來源類型設定為 **OracleServiceCloudSource**。 複製活動**來源**部份支援以下屬性:

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**OracleServiceCloudSource** | 是 |
| 查詢 | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM MyTable"` 。 | 否 (如果已指定資料集中的 "tableName") |

**範例:**

```json
"activities":[
    {
        "name": "CopyFromOracleServiceCloud",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<OracleServiceCloud input dataset name>",
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
                "type": "OracleServiceCloudSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>尋找活動屬性

要瞭解有關屬性的詳細資訊,請檢查[。](control-flow-lookup-activity.md)


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。

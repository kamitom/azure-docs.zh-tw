---
title: 使用 Azure Data Factory 從 Presto 複製資料 (預覽)
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 Presto 複製到支援的接收資料存放區。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: jingwang
ms.openlocfilehash: 8364468277123205d967871ab7bf2d048db64a82
ms.sourcegitcommit: a53fe6e9e4a4c153e9ac1a93e9335f8cf762c604
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/09/2020
ms.locfileid: "80991785"
---
# <a name="copy-data-from-presto-using-azure-data-factory-preview"></a>使用 Azure Data Factory 從 Presto 複製資料 (預覽)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Presto 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

> [!IMPORTANT]
> 此連接器目前為預覽版。 您可以親身體驗並提供意見反應。 如果您需要依賴解決方案中的預覽連接器，請連絡 [Azure 支援](https://azure.microsoft.com/support/)。

## <a name="supported-capabilities"></a>支援的功能

以下活動支援此 Presto 連接器:

- 使用[支援的來源/接收器矩陣](copy-activity-overview.md)[複製活動](copy-activity-overview.md)
- [尋找活動](control-flow-lookup-activity.md)

您可以將資料從 Presto 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Presto 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Presto 已連結服務支援的屬性：

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | Type 屬性必須設定為：**Presto** | 是 |
| 主機 | Presto 伺服器的 IP 位址或主機名稱。 (亦即 192.168.222.160)  | 是 |
| serverVersion | Presto 伺服器的版本。 (亦即 0.148-t)  | 是 |
| catalog | 對伺服器之所有要求的目錄內容。  | 是 |
| 連接埠 | Presto 伺服器用來接聽用戶端連線的 TCP 連接埠。 預設值為 8080。  | 否 |
| authenticationType | 用來連線到 Presto 伺服器的驗證機制。 <br/>允許的值為：**Anonymous**、**LDAP** | 是 |
| username | 用來連線到 Presto 伺服器的使用者名稱。  | 否 |
| 密碼 | 對應到使用者名稱的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 否 |
| enableSsl | 指定與伺服器的連接是否使用 TLS 進行加密。 預設值為 false。  | 否 |
| trustedCertPath | 包含受信任 CA 憑證的 .pem 檔的完整路徑,用於在透過 TLS 連接時驗證伺服器。 僅當在自託管 IR 上使用 TLS 時,才能設置此屬性。 預設值為隨 IR 安裝的 cacerts.pem 檔案。  | 否 |
| useSystemTrustStore | 指定是否使用來自系統信任存放區或來自指定 PEM 檔案的 CA 憑證。 預設值為 false。  | 否 |
| allowHostNameCNMismatch | 指定在透過 TLS 連接時是否需要 CA 頒發的 TLS/SSL 憑證名稱與伺服器的主機名匹配。 預設值為 false。  | 否 |
| allowSelfSignedServerCert | 指定是否允許來自伺服器的自我簽署憑證。 預設值為 false。  | 否 |
| timeZoneID | 連線所使用的本機時區。 這個選項的有效值均指定於 IANA 時區資料庫中。 預設值為系統時區。  | 否 |

**範例:**

```json
{
    "name": "PrestoLinkedService",
    "properties": {
        "type": "Presto",
        "typeProperties": {
            "host" : "<host>",
            "serverVersion" : "0.148-t",
            "catalog" : "<catalog>",
            "port" : "<port>",
            "authenticationType" : "LDAP",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            },
            "timeZoneID" : "Europe/Berlin"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

有關可用於定義數據集的節和屬性的完整清單,請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Presto 資料集所支援的屬性清單。

若要從 Presto 複製資料，請將資料集的 type 屬性設定為 **PrestoObject**。 以下是支援的屬性：

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為 **:PrestoObject** | 是 |
| 結構描述 | 結構描述的名稱。 |否 (如果已指定活動來源中的「查詢」)  |
| 資料表 | 資料表的名稱。 |否 (如果已指定活動來源中的「查詢」)  |
| tableName | 具有架構的表的名稱。 此屬性支援向後相容性。 用於`schema`新`table`工作負載。 | 否 (如果已指定活動來源中的「查詢」) |

**範例**

```json
{
    "name": "PrestoDataset",
    "properties": {
        "type": "PrestoObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Presto linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Presto 來源所支援的屬性清單。

### <a name="presto-as-source"></a>Presto 作為來源

若要從 Presto 複製資料，請將複製活動中的來源類型設定為 **PrestoSource**。 複製活動**來源**部份支援以下屬性:

| 屬性 | 描述 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為：**PrestoSource** | 是 |
| 查詢 | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM MyTable"` 。 | 否 (如果已指定資料集中的 "tableName") |

**範例:**

```json
"activities":[
    {
        "name": "CopyFromPresto",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Presto input dataset name>",
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
                "type": "PrestoSource",
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

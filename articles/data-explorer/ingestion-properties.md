---
title: Azure 資料資源管理器的資料引入屬性
description: 瞭解 Azure 資料資源管理器支援的各種資料引入屬性。
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/19/2020
ms.openlocfilehash: 48bce1bf03a0a4c8d81fff7ae54e0b22261b70f9
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80109572"
---
# <a name="azure-data-explorer-data-ingestion-properties"></a>Azure 資料資源管理器資料引入屬性 

資料引入是將資料添加到表中並可在 Azure 資料資源管理器中查詢的過程。 在`with`關鍵字之後向引入命令添加屬性。

## <a name="ingestion-properties"></a>內嵌屬性

下表列出了 Azure 資料資源管理器支援的屬性，介紹了這些屬性，並提供了示例： 

|屬性              |描述                                              |範例                                             |
|----------------------|---------------------------------------------------------|----------------------------------------------------|
|`ingestionMapping`    |字串值，該值指出如何將來源檔案的資料對應到資料表中的實際資料行。 使用相關`format`映射類型定義值。 請參閱[資料對應](/azure/kusto/management/mappings)。|`with (format="json", ingestionMapping = "[{\"column\":\"rownumber\", \"Properties\":{\"Path\":\"$.RowNumber\"}}, {\"column\":\"rowguid\", \"Properties\":{\"Path\":\"$.RowGuid\"}}]")`<br>(已淘汰： `avroMapping`、`csvMapping`、`jsonMapping`) |
|`ingestionMappingReference`|字串值，該值指出如何使用具名對應原則物件，將來源檔案的資料對應到資料表中的實際資料行。 使用相關`format`映射類型定義值。 請參閱[資料對應](/azure/kusto/management/mappings)。|`with (format="csv", ingestionMappingReference = "Mapping1")`<br>(已淘汰： `avroMappingReference`、`csvMappingReference`、`jsonMappingReference`)|
|`creationTime` |在引入的資料範圍的創建時要使用的日期時間值（格式化為 ISO8601 字串）。 如未指定，則會使用目前的值 (`now()`)。 在引入較舊的資料時，重寫預設值非常有用，以便正確應用保留原則。|`with (creationTime="2017-02-13T11:09:36.7992775Z")`|
|`extend_schema`|布林值，若已指定，則指示命令擴充資料表的結構描述 (預設為 `false`)。 此選項僅適用於 `.append` 和 `.set-or-append` 命令。 唯一允許的結構描述擴充會在資料表結尾新增其他資料行。|如果原始表架構為`(a:string, b:int)`，則有效的架構擴展將無效`(a:string, b:int, c:datetime, d:string)` `(a:string, c:datetime)`|
|`folder` |對於[從查詢引入](/azure/kusto/management/data-ingestion/ingest-from-query)的命令，要分配給表的資料夾。 如果表已存在，則此屬性將覆蓋表的資料夾。|`with (folder="Tables/Temporary")`|
|`format` |資料格式（請參閱[支援的資料格式](ingestion-supported-formats.md)）。|`with (format="csv")`|
|`ingestIfNotExists`|字串值，若已指定，則會在資料表中已經有具相同值和 `ingest-by:` 標記的資料時，防止從後續內嵌。 如此可確保資料以等冪方式擷取。 有關詳細資訊，請參閱[通過： 標記](/azure/kusto/management/extents-overview#ingest-by-extent-tags)。|這些屬性`with (ingestIfNotExists='["Part0001"]', tags='["ingest-by:Part0001"]')`指示，如果具有標記`ingest-by:Part0001`的資料已存在，則不要完成當前引入。 如果不存在，則此新引入應設置此標記（以防將來再次引入相同的資料。|
|`ignoreFirstRecord` |布林值，若設為 `true`，則表示內嵌應該忽略每個檔案的第一筆記錄。 如果檔中的第一個記錄是`CSV`列名，則此屬性對於格式和類似格式的檔很有用。 預設情況下，`false`假定為。|`with (ignoreFirstRecord=false)`|
|`persistDetails` |布林值，若已指定，則表示命令應該保存詳細的結果 (即使成功)，以便 [.show operation details](/azure/kusto/management/operations#show-operation-details) 命令擷取它們。 預設為 `false`。|`with (persistDetails=true)`|
|`policy_ingestiontime`|布林值，若已指定，則描述是否要在此命令所建立的資料表上啟用[內嵌時間](/azure/kusto/management/ingestiontimepolicy)原則。 預設值為 `true`。|`with (policy_ingestiontime=false)`|
|`recreate_schema` |布林值，若已指定，則描述此命令是否可以重建資料表的結構描述。 此屬性僅適用于命令`.set-or-replace`。 如果同時設置兩個屬性`extend_schema`，則此屬性優先于該屬性。|`with (recreate_schema=true)`|
|`tags`|要與引入的資料關聯的標記清單，該[標記](/azure/kusto/management/extents-overview#extent-tagging)的格式為 JSON 字串 |`with (tags="['Tag1', 'Tag2']")`|
|`validationPolicy`|指出要在內嵌期間執行哪些驗證的 JSON 字串。 有關不同選項的說明，請參閱[資料引入](/azure/kusto/management/data-ingestion/)。| `with (validationPolicy='{"ValidationOptions":1, "ValidationImplications":1}')`（這實際上是預設策略）|
|`zipPattern`|從具有 ZIP 存檔的存儲中引入資料時，請使用此屬性。 這是一個字串值，指示在選擇要引入的 ZIP 存檔中的檔的正則運算式。  封存中的所有其他檔案都會被忽略。|`with (zipPattern="*.csv")`|

## <a name="next-steps"></a>後續步驟

* 瞭解有關[資料引入](/azure/data-explorer/ingest-data-overview)的更多
* 瞭解有關[受支援資料格式](ingestion-supported-formats.md)的資訊

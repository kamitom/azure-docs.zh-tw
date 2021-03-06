---
title: 在 Azure 資料工廠中使用 Azure 資料資源管理器控制命令
description: 在本主題中，在 Azure 資料工廠中使用 Azure 資料資源管理器控制項命令
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/15/2019
ms.openlocfilehash: 20da2d54ea54674656b2c1006d094c63133baf79
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "72264483"
---
# <a name="use-azure-data-factory-command-activity-to-run-azure-data-explorer-control-commands"></a>使用 Azure 資料工廠命令活動運行 Azure 資料資源管理器控制命令

[Azure 資料工廠](/azure/data-factory/)（ADF） 是一種基於雲的資料整合服務，允許您對資料執行活動組合。 使用 ADF 創建資料驅動的工作流，以便協調和自動化資料移動和資料轉換。 Azure**資料資源管理器命令**活動在 Azure 資料工廠中使您能夠在 ADF 工作流中運行[Azure 資料資源管理器控制項命令](/azure/kusto/concepts/#control-commands)。 本文介紹如何創建具有查找活動和 ForEach 活動包含 Azure 資料資源管理器命令活動的管道。

## <a name="prerequisites"></a>Prerequisites

* 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費 Azure 帳戶](https://azure.microsoft.com/free/)。
* [Azure 資料資源管理器群集和資料庫](create-cluster-database-portal.md)
* 資料來源。
* [資料工廠](data-factory-load-data.md#create-a-data-factory)

## <a name="create-a-new-pipeline"></a>建立新管線

1. 選擇 **"作者**"鉛筆工具。 
1. 通過選擇**+** 然後從下拉清單中選擇**管道**來創建新管道。

   ![創建新管道](media/data-factory-command-activity/create-pipeline.png)

## <a name="create-a-lookup-activity"></a>創建查找活動

[查找活動](/azure/data-factory/control-flow-lookup-activity)可以從任何 Azure 資料工廠支援的資料來源中檢索資料集。 查找活動的輸出可用於 ForEach 或其他活動。

1. 在"**活動"** 窗格中，在 **"常規**"下，選擇 **"查找**"活動。 將其拖放到右側的主畫布中。
 
    ![選擇查找活動](media/data-factory-command-activity/select-activity.png)

1. 畫布現在包含您創建的查找活動。 使用畫布下方的選項卡更改任何相關參數。 **通常**，重命名活動。 

    ![編輯查找活動](media/data-factory-command-activity/edit-lookup-activity.PNG)

    > [!TIP]
    > 按一下空畫布區域以查看管道屬性。 使用 **"常規**"選項卡重具名管道。 我們的管道被命名為*管道-4-docs。*

### <a name="create-an-azure-data-explorer-dataset-in-lookup-activity"></a>在查找活動中創建 Azure 資料資源管理器資料集

1. 在 **"設置"中**，選擇預先創建的 Azure 資料資源管理器**源資料集**，或選擇 **" 新建"** 以創建新資料集。
 
    ![在查找設置中添加資料集](media/data-factory-command-activity/lookup-settings.png)

1. 從 **"新資料集"** 視窗選擇**Azure 資料資源管理器（庫托）** 資料集。 選擇 **"繼續**添加新資料集"。

   ![選擇新資料集](media/data-factory-command-activity/select-new-dataset.png) 

1. 新的 Azure 資料資源管理器資料集參數在 **"設置"** 中可見。 要更新參數，請選擇 **"編輯**"。

    ![使用 Azure 資料資源管理器資料集查找設置](media/data-factory-command-activity/lookup-settings-with-adx-dataset.png)

1. **AzureDataExplorerTable**新選項卡將在主畫布中打開。 
    * 選擇 **"常規"** 並編輯資料集名稱。 
    * 選擇 **"連接**"以編輯資料集屬性。 
    * 從下拉清單中選擇 **"連結服務**"，或選擇 **"新建"** 以創建新的連結服務。

    ![編輯 Azure 資料資源管理器資料集屬性](media/data-factory-command-activity/adx-dataset-properties-edit-connections.png)

1. 創建新連結服務時，將打開 **"新連結服務（Azure 資料資源管理器）"** 頁：

    ![ADX新連結服務](media/data-factory-command-activity/adx-new-linked-service.png)

   * 為 Azure 資料資源管理器連結服務選擇**名稱**。 如果需要 **，添加說明**。
   * 在**通過集成運行時連接**時，如果需要，更改當前設置。 
   * 在 **"科目選擇"方法**中，使用以下兩種方法之一選擇群集： 
        * 選擇"**從 Azure 訂閱**單選"按鈕，然後選擇**Azure 訂閱**帳戶。 然後，**選擇群集**。 請注意，下拉清單將僅列出屬於使用者的群集。
        * 相反，選擇 **"輸入手動**單選"按鈕並輸入**終結點**（群集 URL）。
    * 指定**租戶**。
    * 輸入**服務主體 ID**。 根據所使用的命令所需的權限等級，主體 ID 必須具有足夠的許可權。
    * 選擇**服務主體鍵**按鈕並輸入**服務主體金鑰**。
    * 從下拉式功能表中選擇**資料庫**。 或者，選擇 **"編輯"** 核取方塊並輸入資料庫名稱。
    * 選擇 **"測試連接**"以測試您創建的連結服務連接。 如果可以連接到您的設置，將顯示綠色核取記號**連接成功**。
    * 選擇 **"完成"** 以完成連結的服務創建。

1. 設置連結服務後，在**AzureDataExplorerTable** > **連接**中，添加**表**名稱。 選擇**預覽資料**，以確保資料正確顯示。

   資料集現已準備就緒，您可以繼續編輯管道。

### <a name="add-a-query-to-your-lookup-activity"></a>向查找活動添加查詢

1. 在**管道 4 文檔** > **設置**中，在**查詢**文字方塊中添加查詢，例如：

    ```kusto
    ClusterQueries
    | where Database !in ("KustoMonitoringPersistentDatabase", "$systemdb")
    | summarize count() by Database
    ```

1. 根據需要更改**查詢超時**或**無截斷****和第一行屬性**。 在此流中，我們保留預設**查詢超時**並取消選中核取方塊。 

    ![查找活動的最終設置](media/data-factory-command-activity/lookup-activity-final-settings.png)

## <a name="create-a-for-each-activity"></a>為每個活動創建一個 

[For-每個](/azure/data-factory/control-flow-for-each-activity)活動用於反覆運算集合並在迴圈中執行指定的活動。 

1. 現在，您將"每個"活動添加到管道中。 此活動將處理從查找活動返回的資料。 
    * 在 **"活動"** 窗格中，在 **"反覆運算&條件**"下，選擇**ForEach**活動並將其拖放到畫布中。
    * 在"查找"活動的輸出和畫布中 ForEach 活動的輸入之間繪製一條線以連接它們。

        ![ForEach 活動](media/data-factory-command-activity/for-each-activity.png)

1.  選擇畫布中的"ForEach"活動。 在下面的 **"設置"** 選項卡中：
    * 選中 **"順序**"核取方塊，瞭解查找結果的連續處理，或取消選中該核取方塊以創建並行處理。
    * 設置**批次處理計數**。
    * 在**Items 中**，提供以下對輸出值的引用*@activity：（"查找 1"）。輸出值。*

       ![ForEach 活動設定](media/data-factory-command-activity/for-each-activity-settings.png)

## <a name="create-an-azure-data-explorer-command-activity-within-the-foreach-activity"></a>在 ForEach 活動中創建 Azure 資料資源管理器命令活動

1. 按兩下畫布中的 ForEach 活動以在新畫布中打開該活動，以指定 ForEach 中的活動。
1. 在 **"活動"** 窗格中，在**Azure 資料資源管理器**下，選擇**Azure 資料資源管理器命令**活動並將其拖放到畫布中。

    ![Azure 資料資源管理器命令活動](media/data-factory-command-activity/adx-command-activity.png)

1.  在 **"連接"** 選項卡中，選擇以前創建的相同連結服務。

    ![azure 資料資源管理器命令活動連接選項卡](media/data-factory-command-activity/adx-command-activity-connection-tab.png)

1. 在 **"命令"** 選項卡中，提供以下命令：

    ```kusto
    .export
    async compressed
    into csv h"http://<storageName>.blob.core.windows.net/data/ClusterQueries;<storageKey>" with (
    sizeLimit=100000,
    namePrefix=export
    )
    <| ClusterQueries | where Database == "@{item().Database}"
    ```

    該**命令**指示 Azure 資料資源管理器以壓縮格式將給定查詢的結果匯出到 Blob 存儲中。 它非同步運行（使用非同步修改器）。
    查詢處理"查找"活動結果中每行的資料庫列。 **命令逾時**可以保持不變。

    ![命令活動](media/data-factory-command-activity/command.png)   

    > [!NOTE]
    > 命令活動具有以下限制：
    > * 大小限制：1 MB 回應大小
    > * 時間限制：20 分鐘（預設值），1 小時（最大值）。
    > * 如果需要，可以使用[AdminThenQuery](/azure/kusto/management/index#combining-queries-and-control-commands)將查詢追加到結果，以減少結果的大小/時間。

1.  現在管道已經準備好了。 您可以通過按一下管道名稱返回主管道視圖。

    ![Azure 資料資源管理器命令管道](media/data-factory-command-activity/adx-command-pipeline.png)

1. 在發佈管道之前選擇**調試**。 可以在 **"輸出"** 選項卡中監視管道進度。

    ![azure 資料資源管理器命令活動輸出](media/data-factory-command-activity/command-activity-output.png)

1. 您可以**發佈全部內容**，然後**添加觸發器**以運行管道。 

## <a name="control-command-outputs"></a>控制命令輸出

命令活動輸出的結構詳見下文。 此輸出可以由管道中的下一個活動使用。

### <a name="returned-value-of-a-non-async-control-command"></a>非同步控制命令的傳回值

在非同步控制命令中，傳回值的結構類似于查找活動結果的結構。 此欄位`count`指示返回的記錄數。 固定陣列欄位`value`包含記錄清單。 

```json
{ 
    "count": "2", 
    "value": [ 
        { 
            "ExtentId": "1b9977fe-e6cf-4cda-84f3-4a7c61f28ecd", 
            "ExtentSize": 1214.0, 
            "CompressedSize": 520.0 
        }, 
        { 
            "ExtentId": "b897f5a3-62b0-441d-95ca-bf7a88952974", 
            "ExtentSize": 1114.0, 
            "CompressedSize": 504.0 
        } 
    ] 
} 
```
 
### <a name="returned-value-of-an-async-control-command"></a>非同步控制命令的傳回值

在非同步控制命令中，活動在後臺輪詢動作表，直到非同步作業完成或超時。因此，返回的值將包含給定`.show operations OperationId`**的"操作 Id"** 屬性的結果。 檢查**狀態**和**狀態**屬性的值，以驗證操作的完成成功。

```json
{ 
    "count": "1", 
    "value": [ 
        { 
            "OperationId": "910deeae-dd79-44a4-a3a2-087a90d4bb42", 
            "Operation": "TableSetOrAppend", 
            "NodeId": "", 
            "StartedOn": "2019-06-23T10:12:44.0371419Z", 
            "LastUpdatedOn": "2019-06-23T10:12:46.7871468Z", 
            "Duration": "00:00:02.7500049", 
            "State": "Completed", 
            "Status": "", 
            "RootActivityId": "f7c5aaaf-197b-4593-8ba0-e864c94c3c6f", 
            "ShouldRetry": false, 
            "Database": "MyDatabase", 
            "Principal": "<some principal id>", 
            "User": "<some User id>" 
        } 
    ] 
}
``` 

## <a name="next-steps"></a>後續步驟

* 瞭解如何[使用 Azure 資料工廠將資料複製到 Azure 資料資源管理器](data-factory-load-data.md)。
* 瞭解如何使用[Azure 資料工廠範本將批量複製到 Azure 資料資源管理器](data-factory-template.md)。

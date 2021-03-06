---
title: Durable Functions 中的效能和級別 - Azure
description: Azure Functions 的 Durable Functions 擴充簡介。
author: cgillum
ms.topic: conceptual
ms.date: 11/03/2019
ms.author: azfuncdf
ms.openlocfilehash: 260811c4ae15b45de6f7bc1b22e3ed6dcea44259
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79277904"
---
# <a name="performance-and-scale-in-durable-functions-azure-functions"></a>Durable Functions (Azure Functions) 中的效能和級別

若要最佳化效能和延展性，務必了解 [Durable Functions](durable-functions-overview.md) 獨特的調整規模特性。

若要了解調整規模行為，您必須了解基礎 Azure 儲存體提供者的一些詳細資料。

## <a name="history-table"></a>記錄資料表

[歷程記錄]**** 資料表是 Azure 儲存體資料表，含有工作中樞內所有協調流程執行個體的歷程記錄事件。 此資料表的名稱格式為 *TaskHubName*History。 隨著執行個體執行，此資料表中會新增資料列。 此資料表的資料分割索引鍵衍生自協調流程的執行個體識別碼。 在大部分情況下，執行個體識別碼都是隨機的，可確保 Azure 儲存體中的內部資料分割有最佳的分佈。

當協調流程執行個體需要執行時，系統會將 [歷程記錄] 資料表的適當資料列載入記憶體中。 這些「歷程記錄事件」** 會接著重新顯示為協調器函式程式碼，使其回到先前的檢查點狀態。 以這種方式使用執行歷程記錄重建狀態，會受[事件來源模式](https://docs.microsoft.com/azure/architecture/patterns/event-sourcing)所影響。

## <a name="instances-table"></a>執行個體資料表

**實例**表是另一個 Azure 存儲表，其中包含任務中心內所有業務流程和實體實例的狀態。 隨著執行個體的建立，此資料表中會新增資料列。 此表的分區鍵是業務流程實例 ID 或實體鍵，行鍵是固定常量。 每個業務流程或實體實例有一行。

此表用於滿足`GetStatusAsync`來自 （.NET） 和`getStatus`（JavaScript） API 以及[狀態查詢 HTTP API](durable-functions-http-api.md#get-instance-status)的實例查詢請求。 它終於與先前所述的 [歷程記錄]**** 資料表內容保持一致。 以這種方式使用不同的 Azure 儲存體資料表有效地滿足執行個體查詢作業，會受到[命令和查詢責任隔離 (CQRS) 模式](https://docs.microsoft.com/azure/architecture/patterns/cqrs)所影響。

## <a name="internal-queue-triggers"></a>內部佇列觸發程序

協調器函式和活動函式都是由函式應用程式的工作中樞內的內部佇列所觸發。 以這種方式使用佇列會提供可靠的「至少一次」訊息傳遞保證。 Durable Functions 中有兩種佇列：**控制佇列**和**工作項目佇列**。

### <a name="the-work-item-queue"></a>工作項目佇列

在 Durable Functions 中，每個工作中樞各有一個工作項目佇列。 這是基本佇列，運作方式類似於 Azure Functions 中的其他任何 `queueTrigger` 佇列。 此佇列用來觸發無狀態「活動函式」**，做法是一次從佇列中清除一則訊息。 上述每則訊息都包含活動函式輸入和其他中繼資料，例如要執行哪個函式。 當 Durable Functions 應用程式向外延展至多個虛擬機器時，這些虛擬機器全部會從工作項目佇列中爭奪工作。

### <a name="control-queues"></a>控制佇列

在 Durable Functions 中，每個工作中樞都有多個「控制佇列」**。 「控制佇列」** 比簡單的工作項目佇列更複雜。 控制項佇列用於觸發有狀態的協調器和實體函數。 由於協調器和實體函數實例是狀態單例，因此無法使用競爭消費者模型跨 VM 分配負載。 相反，協調器和實體消息在控制項佇列中負載平衡。 後續各節對此行為有更詳細的說明。

控制佇列包含各種不同的協調流程生命週期訊息類型。 例如，[協調器控制訊息](durable-functions-instance-management.md)、活動函式「回應」** 訊息，以及計時器訊息。 將有 32 則訊息會在單一輪詢中從控制佇列中清除。 這些訊息包含承載資料以及中繼資料，包括適用於哪個協調流程執行個體。 如果有多則已從佇列中清除的訊息適用於相同的協調流程執行個體，系統會將這些訊息當作一個批次處理。

### <a name="queue-polling"></a>佇列輪詢

持久的任務擴展實現隨機指數回退演算法，以減少空閒佇列輪詢對存儲事務成本的影響。 找到消息後，運行時會立即檢查另一條消息;當找不到消息時，它會等待一段時間，然後再重試。 在隨後嘗試獲取佇列消息失敗後，等待時間繼續增加，直到達到預設為 30 秒的最大等待時間。

最大輪詢延遲可通過`maxQueuePollingInterval`[host.json 檔中](../functions-host-json.md#durabletask)的屬性進行配置。 將此屬性設置為較高的值可能會導致消息處理延遲增加。 只有在不活動期之後，預計更高的延遲率才會增加。 將此屬性設置為較低的值可能會導致存儲成本增加，因為存儲事務增加。

> [!NOTE]
> 在 Azure 函數消耗和高級計畫中運行時[，Azure 函數縮放控制器](../functions-scale.md#how-the-consumption-and-premium-plans-work)將每隔 10 秒輪詢每個控制項和工作項佇列一次。 此附加輪詢對於確定何時啟動函數應用實例和做出縮放決策是必需的。 在編寫本文時，此 10 秒間隔是恒定的，無法配置。

## <a name="storage-account-selection"></a>儲存體帳戶選取

持久函數使用的佇列、表和 Blob 在配置的 Azure 存儲帳戶中創建。 可以使用**host.json**檔中的`durableTask/storageProvider/connectionStringName`設置（或`durableTask/azureStorageConnectionStringName`持久函數 1.x 中的設置）指定要使用的帳戶。

### <a name="durable-functions-2x"></a>持久功能 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "connectionStringName": "MyStorageAccountAppSetting"
      }
    }
  }
}
```

### <a name="durable-functions-1x"></a>持久功能 1.x

```json
{
  "extensions": {
    "durableTask": {
      "azureStorageConnectionStringName": "MyStorageAccountAppSetting"
    }
  }
}
```

如果未指定，則會使用預設 `AzureWebJobsStorage` 儲存體帳戶。 不過，對於重視效能的工作負載，建置您設定非預設的儲存體帳戶。 Durable Functions 會密集使用 Azure 儲存體，而使用專用儲存體帳戶會隔離 Durable Functions 儲存體使用量與 Azure Functions 主機的內部使用量。

## <a name="orchestrator-scale-out"></a>協調器向外延展

活動函式是無狀態，且經由新增虛擬機器而自動相應放大。 另一方面，協調器函數和實體在一個或多個控制佇列中*分區*。 控制佇列數目會定義於 **host.json** 檔案中。 下面的示例 host.json 程式碼片段`durableTask/storageProvider/partitionCount`將屬性（`durableTask/partitionCount`或持久函數 1.x）`3`集到 。

### <a name="durable-functions-2x"></a>持久功能 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
          "partitionCount": 3
      }
    }
  }
}
```

### <a name="durable-functions-1x"></a>持久功能 1.x

```json
{
  "extensions": {
    "durableTask": {
      "partitionCount": 3
    }
  }
}
```

一個工作中樞可設定為有 1 到 16 個資料分割。 若未指定，則預設資料分割計數為 **4**。

向外延展至多個函式主機執行個體時 (通常在不同的虛擬機器上)，每個執行個體取得其中一個控制佇列的鎖定。 這些鎖在內部實現為 blob 存儲租約，並確保業務流程實例或實體一次僅在單個主機實例上運行。 如果任務中心配置了三個控制佇列，則業務流程實例和實體可以在多達三個 VM 之間進行負載平衡。 您可以新增更多虛擬機器，以增加容量來執行活動函式。

下圖說明在向外延展環境中，Azure Functions 主機與儲存體實體之間的互動方式。

![級別圖表](./media/durable-functions-perf-and-scale/scale-diagram.png)

如上圖所示，所有虛擬機器會爭奪工作項目佇列上的訊息。 不過，只有三個虛擬機器可以從控制佇列中取得訊息，每個虛擬機器會鎖定單一控制佇列。

業務流程實例和實體分佈在所有控制項佇列實例中。 分發是通過散列業務流程的實例 ID或機構名稱和鍵對來完成的。 預設情況下，業務流程實例 I 是隨機 GUID，可確保實例在所有控制佇列中平均分佈。

一般而言，協調器函式是輕巧的設計，應該不需要大量運算能力。 因此，無需創建大量控制佇列分區即可獲得出色的業務流程輸送量。 大部分繁重的工作應在無狀態活動函式中進行，這還可以無限制地相應放大。

## <a name="auto-scale"></a>自動調整規模

與消耗和彈性高級計畫中運行的所有 Azure 函數一樣，持久函數支援通過 Azure[函數縮放控制器](../functions-scale.md#runtime-scaling)自動縮放。 縮放控制器藉由定期發出 _peek_ 命令來監視所有佇列的延遲。 縮放控制器會以所查看訊息的延遲為基礎，決定是要新增或移除 VM。

如果縮放控制器判斷控制佇列訊息延遲太高，它會新增 VM 執行個體，直到訊息延遲減少到可接受的程度或達到控制佇列資料分割計數。 同樣地，如果工作項目佇列延遲偏高，不論資料分割計數為何，縮放控制器都會持續新增 VM 執行個體。

> [!NOTE]
> 從持久功能 2.0 開始，可以將函數應用配置為在彈性高級計畫中受 VNET 保護的服務終結點內運行。 在此配置中，持久函數觸發啟動縮放請求，而不是縮放控制器。

## <a name="thread-usage"></a>執行緒使用方式

協調器函式會在單一執行緒上執行，確保執行作業可透過多次重新執行來下決定。 由於此單一執行緒執行作業的緣故，所以協調器函式執行緒切勿執行需要大量 CPU 的工作、進行 I/O 或因故封鎖。 任何可能需要 I/O、封鎖或多個執行緒的工作，都應該移入活動函式中。

活動函式和一般佇列觸發的函式有完全相同的行為。 它們可以安全地執行 I/O、執行需要大量 CPU 的作業，以及使用多個執行緒。 因為活動觸發程序是無狀態，所以能夠自由地向外延展至不限數量的虛擬機器。

實體函數也在單一執行緒上執行，操作一次處理。 但是，實體函數對可執行檔代碼類型沒有任何限制。

## <a name="concurrency-throttles"></a>並行節流

Azure Functions 支援在單一應用程式執行個體中同時執行多個函式。 此並行執行作業有助於提升平行處理原則，且盡可能減少典型應用程式在一段時間內會遇到的「冷啟動」數目。 但是，高併發可能會耗盡每個 VM 系統資源，如網路連接或可用記憶體。 視函式應用程式的需求而定，每個執行個體並行處理可能必須進行節流，以避免在高負載情況下記憶體不足的可能性。

可以在**host.json**檔中配置活動、協調器和實體函數併發限制。 相關設置用於`durableTask/maxConcurrentActivityFunctions`活動函數以及`durableTask/maxConcurrentOrchestratorFunctions`協調器和實體函數。

### <a name="functions-20"></a>功能 2.0

```json
{
  "extensions": {
    "durableTask": {
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10
    }
  }
}
```

### <a name="functions-1x"></a>Functions 1.x

```json
{
  "durableTask": {
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10
  }
}
```

在前面的示例中，最多 10 個協調器或實體函數和 10 個活動函數可以同時在單個 VM 上運行。 如果未指定，則併發活動和業務流程或實體函數執行的數量上限為 VM 上內核數的 10 倍。

> [!NOTE]
> 這些設定很有用，有助於管理單一 VM 上的記憶體和 CPU 使用量。 但是，當跨多個 VM 進行擴展時，每個 VM 都有自己的限制集。 這些設置不能用於在全域級別控制併發性。

## <a name="extended-sessions"></a>擴展會話

擴展會話是一種設置，即使在業務流程和實體完成處理消息後，也會保留在記憶體中。 啟用擴充工作階段的典型效果，就是減少對 Azure 儲存體帳戶的 I/O 及提升整體輸送量。

您可以通過在**host.json** `true`檔中設置為`durableTask/extendedSessionsEnabled`啟用擴展會話。 該`durableTask/extendedSessionIdleTimeoutInSeconds`設置可用於控制空閒會話在記憶體中保持的時間：

**功能 2.0**
```json
{
  "extensions": {
    "durableTask": {
      "extendedSessionsEnabled": true,
      "extendedSessionIdleTimeoutInSeconds": 30
    }
  }
}
```

**功能 1.0**
```json
{
  "durableTask": {
    "extendedSessionsEnabled": true,
    "extendedSessionIdleTimeoutInSeconds": 30
  }
}
```

此設置有兩個潛在的缺點需要注意：

1. 功能應用記憶體使用量總體增加。
2. 如果存在許多併發、短壽命的業務流程協調器或實體函數執行，則輸送量可能會總體下降。

例如，如果`durableTask/extendedSessionIdleTimeoutInSeconds`設置為 30 秒，則在不到 1 秒內執行的短壽命協調器或實體函數單集仍將佔用記憶體 30 秒。 它還對前面提到的`durableTask/maxConcurrentOrchestratorFunctions`配額進行計數，可能會阻止其他協調器或實體函數運行。

下一節將介紹擴展會話對協調器和實體函數的具體影響。

### <a name="orchestrator-function-replay"></a>協調器函式重新執行

如先前所述，使用 [歷程記錄]**** 資料表的內容可重新執行協調器函式。 根據預設，每次從控制佇列中清除一批訊息時，就會重新執行協調器函式程式碼。 即使您使用的是扇出、扇出模式，並且正在等待所有任務完成（例如，`Task.WhenAll`在 .NET 或`context.df.Task.all`JavaScript 中使用），也會隨著任務回應的批次處理而發生重播。 啟用擴展會話後，協調器函數實例在記憶體中保留的時間更長，並且無需完全歷史記錄重播即可處理新消息。

擴展會話的性能改進最常在以下情況下觀察到：

* 當同時運行的業務流程實例數量有限時。
* 當業務流程具有大量快速完成的順序操作（例如數百個活動函式呼叫）時。
* 當業務流程扇出和扇出大量操作時，這些操作大約在同一時間完成。
* 當協調器函數需要處理大消息或執行任何 CPU 密集型資料處理時。

在所有其他情況下，協調器函數通常沒有可觀察到的性能改進。

> [!NOTE]
> 只有在協調器函式經過完整開發及測試之後，才能使用這些設定。 預設主動重播行為可用於在開發時檢測[業務流程協調器函數代碼約束](durable-functions-code-constraints.md)衝突，因此預設情況下禁用。

### <a name="entity-function-unloading"></a>實體函數卸載

實體函數在單個批次處理中處理多達 20 個操作。 一旦實體完成處理一批操作，它就會保持其狀態並從記憶體中卸載。 您可以使用擴展會話設置延遲從記憶體卸載實體。 實體繼續像以前那樣保留其狀態更改，但在配置的時間段內保留在記憶體中，以減少 Azure 存儲的載入次數。 Azure 存儲負載的減少可以提高頻繁訪問的實體的總體輸送量。

## <a name="performance-targets"></a>效能目標

在規劃將 Durable Functions 用於生產應用程式時，請務必在規劃初期考慮效能需求。 本節涵蓋一些基本使用案例，以及預期的最大輸送量數字。

* **循序活動執行**：這個案例說明的協調器函式會一個接一個執行一系列活動函式。 它最類似於[函式鏈結](durable-functions-sequence.md)範例。
* **平行活動執行**：這個案例說明的協調器函式會使用[展開傳送、收合傳送](durable-functions-cloud-backup.md)模式平行執行許多活動函式。
* **平行處理回應**：這個案例是[展開傳送、收合傳送](durable-functions-cloud-backup.md)模式的第二個部份。 它著重於收合傳送的效能。 請務必請注意，不同於展開傳送，收合傳送是由單一協調器函式執行個體進行，因此只能在單一 VM 上執行。
* **外部事件處理**：這個案例代表為[外部事件](durable-functions-external-events.md)服務的單一協調器函式執行個體 (一次一個)。
* **實體操作處理**：此方案測試_單個_[計數器實體](durable-functions-entities.md)處理恒定操作流的速度。

> [!TIP]
> 不同於展開傳送，收合傳送作業受限制於單一 VM。 如果您的應用程式使用展開傳送、收合傳送模式，而且您很在意收合傳送效能，請考慮將活動函式展開傳送細分到多個[子協調流程](durable-functions-sub-orchestrations.md)。

下表顯示先前所述案例的預期「最大」** 輸送量數字。 「執行個體」是指在 Azure App Service 中單一小型 ([A1](../../virtual-machines/sizes-previous-gen.md)) VM 上執行之協調器函式的單一執行個體。 在所有情況下，假設已啟用[擴充工作階段](#orchestrator-function-replay)。 實際結果可能會因函式程式碼所執行的 CPU 或 I/O 工作而有所不同。

| 狀況 | 最大輸送量 |
|-|-|
| 循序活動執行 | 每個執行個體每秒 5 個活動 |
| 平行活動執行 (展開傳送) | 每個執行個體每秒 100 個活動 |
| 平行回應處理 (收合傳送) | 每個執行個體每秒 150 個回應 |
| 外部事件處理 | 每個執行個體每秒 50 個事件 |
| 實體操作處理 | 每秒 64 個操作 |

> [!NOTE]
> 這些數字是 Durable Functions 擴充功能 v1.4.0 (GA) 版的現行資料。 隨著功能更臻成熟及最佳化的進行，這些數字可能會隨時間改變。

如果您未看見您預期的輸送量數字，而且您的 CPU 和記憶體使用狀況似乎良好，請查看原因是否與[儲存體帳戶的健康情況](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance)相關。 Durable Functions 擴充功能可能為 Azure 儲存體帳戶帶來大量負載，而極高的負載可能導致儲存體帳戶節流。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解災害復原和地理分佈](durable-functions-disaster-recovery-geo-distribution.md)

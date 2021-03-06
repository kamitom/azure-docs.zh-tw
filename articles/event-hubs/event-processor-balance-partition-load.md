---
title: 平衡跨多個實例的分區負載 - Azure 事件中心 |微軟文件
description: 介紹如何使用事件處理器和 Azure 事件中心 SDK 平衡應用程式多個實例的分區負載。
services: event-hubs
documentationcenter: .net
author: ShubhaVijayasarathy
editor: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/16/2020
ms.author: shvija
ms.openlocfilehash: bf90120157bf64bd62a3b5ec9d8a6b2c6260e024
ms.sourcegitcommit: 632e7ed5449f85ca502ad216be8ec5dd7cd093cb
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/30/2020
ms.locfileid: "80398304"
---
# <a name="balance-partition-load-across-multiple-instances-of-your-application"></a>平衡應用程式多個實體的分割區負載
要擴展事件處理應用程式,可以運行應用程式的多個實例,並讓它平衡其之間的負載。 在舊版本中[,EventProcessorHost](event-hubs-event-processor-host.md)允許您在接收時平衡程式的多個實例和檢查點事件之間的負載。 在較新版本(5.0 以後)、**事件處理器用戶端**(.NET 和 Java)或**EventHub消費者用戶端**(Python 和 JavaScript)中,允許您執行相同的操作。 通過使用事件使開發模型變得更簡單。 通過註冊事件處理程式訂閱您感興趣的事件。

本文介紹一個示例方案,用於使用多個實例從事件中心讀取事件,然後提供有關事件處理器用戶端功能的詳細資訊,該功能允許您一次從多個分區接收事件,並與使用同一事件中心和消費組的其他使用者進行負載平衡。

> [!NOTE]
> 分割取用者概念是針對事件中樞調整大小的關鍵。 與[競爭取用者](https://msdn.microsoft.com/library/dn568101.aspx)模式相比，分割取用者模式可藉由消除爭奪瓶頸，以及加速端對端平行處理原則來具有高度縮放能力。

## <a name="example-scenario"></a>範例案例

針對範例案例，請設想有一家監視 100,000 戶家庭的住家安全公司。 每分鐘,它都會從安裝在每個家庭的各種感測器(如運動探測器、門/窗打開感測器、玻璃破碎探測器等)獲得數據。 該公司會提供網站給住戶，讓他們可近乎即時地監視房子內的活動。

每個感應器會將資料推送至事件中樞。 事件中樞已設定 16 個分割區。 在使用端,您需要一種機制來讀取這些事件、合併這些事件(篩選器、聚合等)並將聚合轉儲到存儲 Blob,然後投影到使用者友好的網頁。

## <a name="write-the-consumer-application"></a>撰寫取用者應用程式

在分散式環境中設計取用者時，案例必須處理下列需求：

1. **縮放：** 建立多個取用者，而每個取用者都會負責從幾個事件中樞分割區讀取資料。
2. **負載平衡：** 以動態方式增加或減少取用者。 例如，對每個家庭新增感應器類型 (例如一氧化碳偵測器) 時，事件數目就會增加。 在此情況下，操作人員會增加取用者執行個體的數目。 然後，取用者集區可以重新平衡他們擁有的分割區數目，讓新增的取用者共同分攤負載。
3. **容錯時無縫復原:** 如果消費者 (**消費者 A**) 發生故障 (例如,託管消費者的虛擬機突然崩潰), 則其他消費者可以選取消費者**A**擁有的分區並繼續。 此外，接續點 (稱為「檢查點」** 或「位移」**) 應剛好位在**取用者 A** 失敗的位置，或在此稍微前面一點的位置。
4. **使用事件:** 雖然前三點涉及消費者的管理,但必須有代碼來使用事件並對其進行一些有用的操作。 例如,聚合它並將其上載到 Blob 存儲。

## <a name="event-processor-or-consumer-client"></a>事件處理器或使用者用戶端

您無需構建自己的解決方案來滿足這些要求。 Azure 事件中心 SDK 提供此功能。 在 .NET 或 Java SDK 中,您可以使用事件處理器用戶端 (事件處理器用戶端),在 Python 和 Java 文稿 SDK 中,您可以使用 EventHub 消費者用戶端。 在舊版本的 SDK 中,支援這些功能的事件處理器主機 (EventProcessor Host) 是。

對於大多數生產方案,我們建議您使用事件處理器客戶端讀取和處理事件。 處理器用戶端旨在以執行性和容錯的方式跨事件中心的所有分區處理事件,同時提供檢查其進度的方法, 提供可靠的體驗。 事件處理器用戶端還能夠在給定事件中心的消費者組上下文中協同工作。 當實例為組可用或不可用時,用戶端將自動管理工作的分發和平衡。

## <a name="partition-ownership-tracking"></a>分割區的擁有權追蹤

事件處理器實例通常擁有和處理來自一個或多個分區的事件。 分區的擁有權在與事件中心和消費群體組合關聯的所有活動事件處理器實例之間均勻分佈。 

每個事件處理器都獲得一個唯一標識符,並通過在檢查點存儲中添加或更新條目來聲明分區的擁有權。 所有事件處理器實例定期與此存儲通信,以更新其自己的處理狀態,並瞭解其他活動實例。 然後,此數據用於平衡活動處理器之間的負載。 新實例可以聯接處理池以進行擴展。 當實例出現故障時(由於失敗或縮減,分區擁有權將正常傳輸到其他活動處理器)。

檢查點儲存中的分區擁有權記錄跟蹤事件中心命名空間、事件中心名稱、消費者組、事件處理器標識符(也稱為擁有者)、分區 ID 和上次修改時間。



| 事件中樞命名空間               | 事件中樞名稱 | **消費者組** | 擁有者                                | 資料分割識別碼 | 上次變更時間  |
| ---------------------------------- | -------------- | :----------------- | :----------------------------------- | :----------- | :------------------ |
| mynamespace.servicebus.windows.net | 邁文丘布     | 我的消費者團體    | 3be3f9d3-9d9e-4c50-9491-85ece8334ff6 | 0            | 2020-01-15T01:22:15 |
| mynamespace.servicebus.windows.net | 邁文丘布     | 我的消費者團體    | f5cc5176-ce96-4bb4-bbaa-a0e3a9054ecf | 1            | 2020-01-15T01:22:17 |
| mynamespace.servicebus.windows.net | 邁文丘布     | 我的消費者團體    | 72b980e9-2efc-4ca7-ab1b-ffd7bece8472 | 2            | 2020-01-15T01:22:10 |
|                                    |                | :                  |                                      |              |                     |
|                                    |                | :                  |                                      |              |                     |
| mynamespace.servicebus.windows.net | 邁文丘布     | 我的消費者團體    | 844bd8fb-1f3a-4580-984d-6324f9e208af | 15           | 2020-01-15T01:22:00 |

每個事件處理器實例獲取分區的擁有權,並開始從上次已知的[檢查點](# Checkpointing)處理分區。 如果處理器發生故障(VM 關閉),則其他實例通過查看上次修改時間來檢測此情況。 其他實例嘗試獲取以前由非活動實例擁有的分區的擁有權,並且檢查點存儲保證只有一個實例能夠成功聲明分區的擁有權。 因此,在任何給定的時間點,最多有一個處理器從分區接收事件。

## <a name="receive-messages"></a>接收訊息

建立事件處理器時,指定將處理事件和錯誤的函數。 對處理事件的函數的每個調用都會從特定分區傳遞單個事件。 你有責任處理這件事。 如果要確保消費者至少處理每條消息一次,則需要使用重試邏輯編寫自己的代碼。 但請留意有害訊息。

我們建議您相對較快地做事。 也就是說,盡可能少地處理。 如果需要寫入存儲並執行一些路由,最好使用兩個消費者組並具有兩個事件處理器。

## <a name="checkpointing"></a>檢查點

*檢查點是*事件處理器標記或提交分區中最後一個成功處理的事件的位置的過程。 標記檢查點通常在處理事件的函數中完成,並在消費者組中按分區進行。 

如果事件處理器與分區斷開連接,則另一個實例可以在該使用者組中該分區的最後一個處理器以前提交的檢查點繼續處理分區。 當處理器連接時,它將偏移傳遞給事件中心,以指定開始讀取的位置。 這樣,可以使用檢查點將事件標記為下游應用程式為"已完成",並在事件處理器關閉時提供恢復能力。 從這個檢查點處理程序中指定較低的位移，即可回到較舊的資料。 

執行檢查點以將事件標記為已處理時,將添加或使用事件的偏移量和序列號更新檢查點存儲中的條目。 用戶應決定更新檢查點的頻率。 在每個成功處理的事件之後進行更新都會對性能和成本產生影響,因為它會觸發對基礎檢查點存儲的寫入操作。 此外,檢查每個事件都指示佇列消息模式,為此,服務總線佇列可能比事件中心更好選擇。 事件中樞背後的構想是，在大型規模中您能取得「至少一次」的傳遞。 藉由讓您的下游系統具有等冪性，即可輕鬆地從失敗中復原，或以接收多次的相同事件重新啟動該結果。

> [!NOTE]
> 如果在支援與 Azure 上通常可用的版本的儲存 Blob SDK 不同的環境中使用 Azure Blob 儲存作為檢查點存儲,則需要使用代碼將儲存服務 API 版本更改為該環境支援的特定版本。 例如,如果您在[Azure 堆疊中心版本 2002 上運行事件中心](https://docs.microsoft.com/azure-stack/user/event-hubs-overview),則存儲服務的最高可用版本是版本 2017-11-09。 在這種情況下,您需要使用代碼將儲存服務 API 版本定位到 2017-11-09。 有關如何定位特定存儲 API 版本的範例,請參閱 GitHub 上的以下範例: 
> - [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/Sample10_RunningWithDifferentStorageVersion.cs). 
> - [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithOlderStorageVersion.java)
> - [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/receiveEventsWithDownleveledStorage.js)或[型態文稿](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/receiveEventsWithDownleveledStorage.ts)
> - [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/event_processor_blob_storage_example_with_storage_api_version.py)

## <a name="thread-safety-and-processor-instances"></a>執行緒安全性和處理器執行個體

默認情況下,事件處理器或消費者是線程安全的,並且以同步的方式運行。 當事件到達分區時,將調用處理事件的函數。 後續消息和對此函數的調用在後台排隊,因為消息泵在其他線程的後台繼續運行。 此執行緒安全性不需要安全執行緒集合，並可大幅提升效能。

## <a name="next-steps"></a>後續步驟
請參考以下快速入門:

- [.NET Core](get-started-dotnet-standard-send-v2.md)
- [Java](event-hubs-java-get-started-send.md)
- [Python](get-started-python-send-v2.md)
- [JavaScript](get-started-node-send-v2.md)

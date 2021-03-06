---
title: 故障排除指南 - Azure 事件中心 |微軟文檔
description: 本文提供 Azure 事件中樞傳訊例外狀況和建議的動作清單。
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.custom: seodec18
ms.date: 01/16/2020
ms.author: shvija
ms.openlocfilehash: de5b95bd10bf72f60ba5d63c4b3a799556fcce33
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76309773"
---
# <a name="troubleshooting-guide-for-azure-event-hubs"></a>Azure 事件中心故障排除指南
本文提供了事件中心 .NET 框架 API 生成的一些 .NET 異常，以及故障排除問題的其他提示。 

## <a name="event-hubs-messaging-exceptions---net"></a>事件中心消息異常 - .NET
本節列出了 .NET 框架 API 生成的 .NET 異常。 

### <a name="exception-categories"></a>例外狀況類別

事件中心 .NET API 生成的異常可以分為以下類別，以及您可以嘗試修復它們的關聯操作。

1. 使用者編碼錯誤：[系統.參數異常](https://msdn.microsoft.com/library/system.argumentexception.aspx)，[系統.無效操作異常](https://msdn.microsoft.com/library/system.invalidoperationexception.aspx)，[系統.操作取消異常](https://msdn.microsoft.com/library/system.operationcanceledexception.aspx)，[系統.運行時.序列化.序列化異常](https://msdn.microsoft.com/library/system.runtime.serialization.serializationexception.aspx)。 一般動作：請先嘗試修正此程式碼，再繼續執行。
2. 設定/組態錯誤：[Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagingentitynotfoundexception)、[Microsoft.Azure.EventHubs.MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.eventhubs.messagingentitynotfoundexception)、[System.UnauthorizedAccessException](https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx)。 一般動作：檢閱您的組態並視需要進行變更。
3. 暫時性例外狀況：[Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception)、[Microsoft.ServiceBus.Messaging.ServerBusyException](#serverbusyexception)、[Microsoft.Azure.EventHubs.ServerBusyException](#serverbusyexception)、[Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)。 一般動作：重試此操作或通知使用者。
4. 其他例外情況：[系統.事務.事務異常](https://msdn.microsoft.com/library/system.transactions.transactionexception.aspx)，[系統.超時例外](#timeoutexception)，[微軟.服務匯流排.消息.消息鎖定異常](/dotnet/api/microsoft.servicebus.messaging.messagelocklostexception)，[微軟.服務匯流排.消息.會話鎖定異常](/dotnet/api/microsoft.servicebus.messaging.sessionlocklostexception)。 一般動作︰依例外狀況類型而異；請參閱下一節中的表格。 

### <a name="exception-types"></a>例外狀況類型
下表列出傳訊例外狀況類型及其原因，並指出您可以採取的建議動作。

| 例外狀況類型 | 描述/原因/範例 | 建議的動作 | 自動/立即重試的注意事項 |
| -------------- | -------------------------- | ---------------- | --------------------------------- |
| [超時例外](https://msdn.microsoft.com/library/system.timeoutexception.aspx) |伺服器未在指定時間內回應請求的操作，該操作由[操作超時](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings)控制。 伺服器可能已完成要求的作業。 此異常可能由於網路或其他基礎結構延遲而發生。 |檢查系統狀態的一致性，並視需要重試。<br /> 請參閱 [TimeoutException](#timeoutexception)。 | 在某些情況下，重試也許有幫助；將重試邏輯新增至程式碼。 |
| [InvalidOperationException](https://msdn.microsoft.com/library/system.invalidoperationexception.aspx) |伺服器或服務中不允許請求的使用者操作。 如需詳細資訊，請參閱例外狀況訊息。 例如，如果是在 [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 模式收到訊息， [Complete](/dotnet/api/microsoft.servicebus.messaging.receivemode) 將會產生這個例外狀況。 | 檢查程式碼和文件。 確定要求的作業無效。 | 重試無濟於事。 |
| [OperationCanceledException](https://msdn.microsoft.com/library/system.operationcanceledexception.aspx) | 嘗試在已關閉、中止或處置的物件上叫用作業。 極少數的情況下，環境交易是已處置狀態。 | 檢查代碼並確保它不會調用已釋放物件的操作。 | 重試無濟於事。 |
| [UnauthorizedAccessException](https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx) | [權杖提供程式](/dotnet/api/microsoft.servicebus.tokenprovider)物件無法獲取權杖，權杖無效，或者權杖不包含執行操作所需的聲明。 | 確定權杖提供者是以正確的值建立。 檢查存取控制服務的配置。 | 在某些情況下，重試也許有幫助；將重試邏輯新增至程式碼。 |
| [ArgumentException](https://msdn.microsoft.com/library/system.argumentexception.aspx)<br /> [ArgumentNullException](https://msdn.microsoft.com/library/system.argumentnullexception.aspx)<br />[ArgumentOutOfRangeException](https://msdn.microsoft.com/library/system.argumentoutofrangeexception.aspx) | 提供給方法的一個或多個引數無效。 提供給 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 的 URI 包含路徑區段。 提供給 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 的 URI 配置無效。 屬性值大於 32 KB。 | 檢查呼叫程式碼，並確定引數正確無誤。 | 重試將無助益。 |
| [Microsoft.ServiceBus.Messaging MessagingEntityNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagingentitynotfoundexception) <br /><br/> [Microsoft.Azure.EventHubs MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.eventhubs.messagingentitynotfoundexception) | 與作業相關聯的實體不存在或已被刪除。 | 確定實體已存在。 | 重試將無助益。 |
| [MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) | 用戶端無法建立事件中樞連線。 |確定提供的主機名稱正確，且主機可以連線。 | 如果有間歇性的連線問題，重試也許有幫助。 |
| [微軟.服務匯流排.消息伺服器忙例外](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) <br /> <br/>[Microsoft.Azure.EventHubs ServerBusyException](/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) | 服務目前無法處理要求。 | 用戶端可以等待一段時間，然後再重試作業。 <br /> 請參閱 [ServerBusyException](#serverbusyexception)。 | 用戶端可以在特定間隔後重試。 如果重試產生不同的例外狀況，請檢查該例外狀況的重試行為。 |
| [MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception) | 可能會在下列情況中擲回的一般傳訊例外狀況：利用屬於不同實體類型 (例如主題) 的名稱或路徑嘗試建立 [QueueClient](/dotnet/api/microsoft.servicebus.messaging.queueclient) 。 嘗試傳送大於 1 MB 的訊息。 處理要求時伺服器或服務發生錯誤。 如需詳細資訊，請參閱例外狀況訊息。 此例外狀況通常是暫時性例外狀況。 | 查看程式碼，並確定訊息內文只使用可序列化的物件 (或使用自訂序列化程式)。 查看文件來了解支援的屬性值類型，並且只使用支援的類型。 查看 [IsTransient](/dotnet/api/microsoft.servicebus.messaging.messagingexception) 屬性。 如果該屬性為 **True**，您就可以重試作業。 | 重試行為未定義，而且可能沒有幫助。 |
| [MessagingEntityAlreadyExistsException](/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) | 嘗試在該服務命名空間中以另一個實體已在使用的名稱建立實體。 | 刪除現有的實體，或選擇不同的名稱來建立實體。 | 重試將無助益。 |
| [QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) | 傳訊實體已達到允許的大小上限。 如果在個別取用者群組層級開啟的接收者數目已達到上限 (5)，便可能發生此例外狀況。 | 從實體或其子佇列接收訊息，在實體中建立空間。 <br /> 請參閱[配額超過例外](#quotaexceededexception) | 如果在此同時已移除訊息，重試可能會有幫助。 |
| [MessagingEntityDisabledException](/dotnet/api/microsoft.servicebus.messaging.messagingentitydisabledexception) | 在停用的實體上要求執行階段作業。 |啟用實體。 | 如實體在過渡期間被啟用，重試可能會有幫助。 |
| [Microsoft.ServiceBus.Messaging MessageSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) <br /><br/> [Microsoft.Azure.EventHubs MessageSizeExceededException](/dotnet/api/microsoft.azure.eventhubs.messagesizeexceededexception) | 消息負載超過 1 MB 限制。 此 1 MB 限制用於總消息，其中可以包括系統屬性和任何 .NET 開銷。 | 減少訊息裝載大小，然後再重試作業。 |重試將無助益。 |

### <a name="quotaexceededexception"></a>QuotaExceededException
[QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) 指出已超過某特定實體的配額。

如果在個別取用者群組層級開啟的接收者數目已達到上限 (5)，便可能發生此例外狀況。

#### <a name="event-hubs"></a>事件中樞
每一個事件中樞都有 20 個用戶群組的限制。 當您嘗試建立更多時，您會收到 [QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception)。 

### <a name="timeoutexception"></a>TimeoutException
[TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx) 表示使用者啟始作業所用的時間長過作業逾時。 

事件中樞的逾時是指定為連接字串的一部分，或透過 [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.servicebus.servicebusconnectionstringbuilder)指定。 錯誤訊息本身可能不盡相同，但它一定會包含目前作業的指定逾時值。 

#### <a name="common-causes"></a>常見的原因
這個錯誤有兩個常見的原因︰設定不正確或暫時性服務錯誤。

1. **設定不正確** ：操作條件的作業逾時可能太小。 用戶端 SDK 的作業逾時預設值為 60 秒。 請檢查程式碼是否將值設定過小。 網路和 CPU 使用率的條件可能會影響特定操作完成所需的時間，因此操作超時不應設置為小值。
2. **暫時性服務錯誤** ：有時事件中樞服務在處理要求時會遇到延遲，例如，高流量的時段。 在這種情況下，您可以在延遲後重試作業，直到作業成功為止。 如果多次嘗試同一作業之後仍然失敗，請瀏覽 [Azure 服務狀態網站](https://azure.microsoft.com/status/)，看看是否有任何已知的服務中斷。

### <a name="serverbusyexception"></a>ServerBusyException

[Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) 或 [Microsoft.Azure.EventHubs.ServerBusyException](/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) 表示伺服器已超載。 此例外狀況有兩個相關的錯誤碼。

#### <a name="error-code-50002"></a>錯誤碼 50002

此錯誤會因為兩個原因其中之一而發生：

1. 負載不均勻分佈于事件中心上的所有分區，並且一個分區達到本地輸送量單元限制。
    
    解決方式︰修改資料分割散發策略，或嘗試 [EventHubClient.Send(eventDataWithOutPartitionKey)](/dotnet/api/microsoft.servicebus.messaging.eventhubclient)，可能會有幫助。

2. 事件中心命名空間沒有足夠的輸送量單位（您可以檢查[Azure 門戶](https://portal.azure.com)中事件中心命名空間視窗中的 **"指標"** 螢幕以進行確認）。 門戶顯示聚合（1 分鐘）資訊，但我們即時測量輸送量 ，因此這只是一個估計值。

    解決方式︰增加命名空間的輸送量單位可能會有幫助。 您可以透過入口網站，在 [事件中樞命名空間] 畫面的 [調整]**** 視窗中執行此作業。 您也可以使用[自動擴充](event-hubs-auto-inflate.md)。

#### <a name="error-code-50001"></a>錯誤碼 50001

此錯誤應該很少會發生。 它會在執行您命名空間的程式碼之容器的 CPU 不足時發生 – 就在事件中樞負載平衡器開始之前幾秒鐘。

#### <a name="limit-on-calls-to-the-getruntimeinformation-method"></a>對獲取時間資訊方法的調用限制
Azure 事件中心每秒最多支援 50 次每秒到 GetRuntimeInfo 的呼叫。 達到限制後，您可能會收到類似于以下異常的異常：

```
ExceptionId: 00000000000-00000-0000-a48a-9c908fbe84f6-ServerBusyException: The request was terminated because the namespace 75248:aaa-default-eventhub-ns-prodb2b is being throttled. Error code : 50001. Please wait 10 seconds and try again.
```

## <a name="connectivity-certificate-or-timeout-issues"></a>連接、證書或超時問題
以下步驟可以説明您解決 *.servicebus.windows.net 下所有服務的連接/證書/超時問題。 

- 流覽到 或[wget](https://www.gnu.org/software/wget/) `https://<yournamespacename>.servicebus.windows.net/`。 它有助於檢查您是否存在 IP 篩選或虛擬網路或憑證連結問題（在使用 java SDK 時最為常見）。

    成功消息的示例：
    
    ```xml
    <feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
    ```
    
    失敗錯誤訊息的示例：

    ```json
    <Error>
        <Code>400</Code>
        <Detail>
            Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
        </Detail>
    </Error>
    ```
- 運行以下命令以檢查防火牆上是否有任何埠被阻止。 使用的埠為 443 （HTTPS）、5671 （AMQP） 和 9093 （卡夫卡）。 根據您使用的庫，也會使用其他埠。 下面是檢查 5671 埠是否被阻止的示例命令。

    ```powershell
    tnc <yournamespacename>.servicebus.windows.net -port 5671
    ```

    在 Linux 上：

    ```shell
    telnet <yournamespacename>.servicebus.windows.net 5671
    ```
- 當出現間歇性連接問題時，運行以下命令以檢查是否有任何丟棄的資料包。 此命令將嘗試每 1 秒與服務建立 25 個不同的 TCP 連接。 然後，您可以檢查它們中有多少成功/失敗，還可以查看 TCP 連接延遲。 你可以`psping`[從這裡](/sysinternals/downloads/psping)下載該工具。

    ```shell
    .\psping.exe -n 25 -i 1 -q <yournamespacename>.servicebus.windows.net:5671 -nobanner     
    ```
    如果使用其他工具（如`tnc`、`ping`等），則可以使用等效命令。 
- 如果前面的步驟不起作用，請獲取網路跟蹤，並使用[Wireshark](https://www.wireshark.org/)等工具對其進行分析。 如果需要[，請與微軟支援部門](https://support.microsoft.com/)聯繫。 

## <a name="next-steps"></a>後續步驟

您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [創建事件中心](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

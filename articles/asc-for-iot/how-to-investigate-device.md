---
title: 用於物聯網設備的 Azure 安全中心調查指南*微軟文檔
description: 如何指導如何使用 IoT Azure 安全中心使用日誌分析調查可疑的 IoT 設備。
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: b18b48ae-b445-48f8-9ac0-365d6e065b64
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/23/2019
ms.author: mlottner
ms.openlocfilehash: 8d2fe8d63c7ece6f3b3426d8fc5a3454a61826f8
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "68596247"
---
# <a name="investigate-a-suspicious-iot-device"></a>調查可疑的 IoT 裝置

Azure IoT 服務警報安全中心在懷疑 IoT 設備參與可疑活動或存在設備受到威脅的跡象時提供明確指示。 

在本指南中，使用提供的調查建議來説明確定組織的潛在風險，決定如何修復，併發現防止將來發生類似攻擊的最佳方式。  

> [!div class="checklist"]
> * 尋找您的裝置資料
> * 使用 kql 查詢進行調查


## <a name="how-can-i-access-my-data"></a>如何訪問我的資料？

預設情況下，適用于 IoT 的 Azure 安全中心會在日誌分析工作區中存儲安全警報和建議。 您也可以選擇儲存未經處理的安全性資料。

要查找用於資料存儲的日誌分析工作區，請執行以下操作：

1. 開啟 IoT 中樞， 
1. 在 **"安全"** 下，按一下 **"概述**"，然後選擇 **"設置**"。
1. 變更 Log Analytics 工作區的組態詳細資料。 
1. 按一下 [儲存]****。 

下列組態，執行下列步驟來存取儲存在 Log Analytics 工作區中的資料：

1. 選擇並按一下 IoT 中心中的 IoT 警報 Azure 安全中心。 
1. 按一下 [進一步調查]****。 
1. 選取 [若要查看哪些裝置有此警示，請按一下這裡並檢視 DeviceId 資料行]****。

## <a name="investigation-steps-for-suspicious-iot-devices"></a>可疑 IoT 裝置的調查步驟

要查看有關 IoT 設備的見解和原始資料，請轉到日誌分析工作區[以訪問資料](#how-can-i-access-my-data)。

請參閱下面的示例 kql 查詢，以便開始調查設備上的警報和活動。

### <a name="related-alerts"></a>相關警報

若要了解同一時間前後是否觸發了其他警示，請使用下列 kql 查詢：

  ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityAlert
  | where ExtendedProperties contains device and ResourceId contains tolower(hub)
  | project TimeGenerated, AlertName, AlertSeverity, Description, ExtendedProperties
  ```

### <a name="users-with-access"></a>具有存取權限的使用者

若要了解哪些使用者可以存取此裝置，請使用下列 kql 查詢： 

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "LocalUsers"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     GroupNames=extractjson("$.GroupNames", EventDetails, typeof(string)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string))
  | summarize FirstObserved=min(TimestampLocal) by GroupNames, UserName
 ```
請使用這項資料來了解︰ 
- 哪些使用者可以存取此裝置？
- 具有存取權限的使用者具有預期的權限等級嗎？

### <a name="open-ports"></a>開啟連接埠

要瞭解設備中的哪些埠當前正在使用或使用，請使用以下 kql 查詢： 

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ListeningPorts"
     and extractjson("$.LocalPort", EventDetails, typeof(int)) <= 1024 // avoid short-lived TCP ports (Ephemeral)
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Protocol=extractjson("$.Protocol", EventDetails, typeof(string)),
     LocalAddress=extractjson("$.LocalAddress", EventDetails, typeof(string)),
     LocalPort=extractjson("$.LocalPort", EventDetails, typeof(int)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     RemotePort=extractjson("$.RemotePort", EventDetails, typeof(string))
  | summarize MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), AllowedRemoteIPAddress=makeset(RemoteAddress), AllowedRemotePort=makeset(RemotePort) by Protocol, LocalPort
 ```

請使用這項資料來了解︰
- 裝置上目前作用中的接聽通訊端有哪些？
- 是否應允許當前處於活動狀態的偵聽通訊端？
- 是否有任何可疑的遠端位址連接到設備？

### <a name="user-logins"></a>使用者登錄

要查找登錄到設備的使用者，請使用以下 kql 查詢： 
 
 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "Login"
     // filter out local, invalid and failed logins
     and EventDetails contains "RemoteAddress"
     and EventDetails !contains '"RemoteAddress":"127.0.0.1"'
     and EventDetails !contains '"UserName":"(invalid user)"'
     and EventDetails !contains '"UserName":"(unknown user)"'
     //and EventDetails !contains '"Result":"Fail"'
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string)),
     LoginHandler=extractjson("$.Executable", EventDetails, typeof(string)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     Result=extractjson("$.Result", EventDetails, typeof(string))
  | summarize CntLoginAttempts=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), CntIPAddress=dcount(RemoteAddress), IPAddress=makeset(RemoteAddress) by UserName, Result, LoginHandler
 ```

請使用查詢結果來了解︰
- 有哪些使用者登入過此裝置？
- 登錄的使用者應該登錄嗎？
- 登入過的使用者是從預期還是非預期的 IP 位址進行連線的？
  
### <a name="process-list"></a>流程清單

要瞭解流程清單是否按預期，請使用以下 kql 查詢： 

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ProcessCreate"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Executable=extractjson("$.Executable", EventDetails, typeof(string)),
     UserId=extractjson("$.UserId", EventDetails, typeof(string)),
     CommandLine=extractjson("$.CommandLine", EventDetails, typeof(string))
  | join kind=leftouter (
     // user UserId details
     SecurityIoTRawEvent
     | where
        DeviceId == device and AssociatedResourceId contains tolower(hub)
        and RawEventName == "LocalUsers"
     | project
        UserId=extractjson("$.UserId", EventDetails, typeof(string)),
        UserName=extractjson("$.UserName", EventDetails, typeof(string))
     | distinct UserId, UserName
  ) on UserId
  | extend UserIdName = strcat("Id:", UserId, ", Name:", UserName)
  | summarize CntExecutions=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), ExecutingUsers=makeset(UserIdName), ExecutionCommandLines=makeset(CommandLine) by Executable
```

請使用查詢結果來了解︰

- 裝置上是否執行了任何可疑的處理序？
- 這些處理序是不是由適當的使用者來執行？
- 是否有任何命令列執行包含正確且預期中的引數？

## <a name="next-steps"></a>後續步驟

在調查裝置並更加了解您的風險之後，請考慮[設定自訂警示](quickstart-create-custom-alerts.md)來改善您的 IoT 解決方案安全性態勢。 如果您還沒有裝置代理程式，請考慮[部署安全性代理程式](how-to-deploy-agent.md)或[變更現有裝置代理程式的組態](how-to-agent-configuration.md)，以改善您的結果。 

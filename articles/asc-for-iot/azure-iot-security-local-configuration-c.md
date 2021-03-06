---
title: 瞭解 C 的代理本地配置的 Azure 安全中心 |微軟文檔
description: 瞭解用於 C 的代理本地配置的 Azure 安全中心。
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 2cf6a49b-5d35-491f-abc3-63ec24eb4bc2
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/26/2019
ms.author: mlottner
ms.openlocfilehash: 2725a824da26dafcbc215e4c302ec38ad4b5a699
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "68600536"
---
# <a name="understanding-the-localconfigurationjson-file---c-agent"></a>了解 LocalConfiguration.json 檔案 - C 代理程式

IoT 安全代理的 Azure 安全中心使用本地設定檔中的配置。
安全代理在代理啟動時讀取配置一次。
本地設定檔中的配置包含身份驗證配置和其他代理相關配置。
該檔包含 JSON 標記法中的"鍵值"對中的配置，並在安裝代理時填充配置。 

預設情況下，該檔位於：/var/ASCIoTAgent/本地配置.json

重新開機代理時，將更改設定檔。 

## <a name="security-agent-configurations-for-c"></a>C 的安全代理配置
| 設定名稱 | 可能值 | 詳細資料 | 
|:-----------|:---------------|:--------|
| AgentId | GUID | 代理唯一識別碼 |
| 觸發事件間隔 | ISO8601 字串 | 觸發事件集合的計畫程式間隔 |
| ConnectionTimeout | ISO8601 字串 | 連接到 IoThub 的時間段超時 |
| 驗證 | JsonObject | 身份驗證配置。 此物件包含針對 IoTHub 進行身份驗證所需的所有資訊 |
| 身分識別 | "DPS"、"安全模組"、"設備" | 身份驗證標識 - 通過 DPS 進行身份驗證的 DPS，如果使用設備憑據進行身份驗證，則通過安全模組憑據或設備進行身份驗證時，使用安全模組模組 |
| 身份驗證方法 | "簽名"，"自簽章憑證" | 使用者金鑰進行身份驗證 - 如果使用金鑰是對稱金鑰，請選擇 SasToken，如果金鑰是自簽章憑證，請選擇自簽章憑證  |
| FilePath | 檔路徑（字串） | 包含身份驗證金鑰的檔的路徑 |
| HostName | 字串 | Azure iot 中心的主機名稱。 通常<我的樞紐>.azure-devices.net |
| deviceId | 字串 | 設備的 ID（在 Azure IoT 中心註冊） |
| DPS | JsonObject | DPS 相關配置 |
| IDScope | 字串 | DPS 的 ID 範圍 |
| RegistrationId | 字串  | DPS 設備註冊 ID |
| 記錄 | JsonObject | 代理記錄器相關配置 |
| 系統記錄最小嚴重性 | 0 <= 數位 <= 4 | 等於且高於此嚴重性的日誌消息將記錄到 /var/log/syslog（0 是最低嚴重性） |
| 診斷事件最小嚴重性 | 0 <= 數位 <= 4 | 等於且高於此嚴重性的日誌消息將作為診斷事件發送（0 是最低嚴重性） |

## <a name="security-agent-configurations-code-example"></a>安全代理配置代碼示例
```JSON
{
    "Configuration" : {
        "AgentId" : "b97faf0a-0f57-471f-9dab-46a8e1764946",
        "TriggerdEventsInterval" : "PT2M",
        "ConnectionTimeout" : "PT30S",
        "Authentication" : {
            "Identity" : "Device",
            "AuthenticationMethod" : "SasToken",
            "FilePath" : "/path/to/my/SymmetricKey",
            "DeviceId" : "my-device",
            "HostName" : "my-iothub.azure-devices.net",
            "DPS" : {
                "IDScope" : "",
                "RegistrationId" : ""
            }
        },
        "Logging": {
            "SystemLoggerMinimumSeverity": 0,
            "DiagnoticEventMinimumSeverity": 2
        }
    }
}
```

## <a name="next-steps"></a>後續步驟
- 閱讀 Azure 安全中心，瞭解 IoT 服務[概述](overview.md)
- 瞭解有關 IoT[體系結構](architecture.md)的 Azure 安全中心
- 啟用適用于 IoT[服務的](quickstart-onboard-iot-hub.md)Azure 安全中心
- 閱讀 Azure 安全中心，瞭解 IoT 服務[常見問題解答](resources-frequently-asked-questions.md)
- 了解如何存取[未經處理的安全性資料](how-to-security-data-access.md)
- 了解[建議](concept-recommendations.md)
- 瞭解安全[警報](concept-security-alerts.md)
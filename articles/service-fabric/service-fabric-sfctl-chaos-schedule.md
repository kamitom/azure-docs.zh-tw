---
title: Azure 服務結構 CLI-sfctl 混亂計畫
description: 瞭解 sfctl，Azure 服務結構命令列介面。 包括用於混沌調度的命令清單。
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 29b365c48e405830e238945f1d94156f477c15b4
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76906181"
---
# <a name="sfctl-chaos-schedule"></a>sfctl chaos schedule
取得並設定 chaos 排程。

## <a name="commands"></a>命令

|Command|描述|
| --- | --- |
| get | 取得定義 Chaos 執行時間及執行方式的 Chaos 排程。 |
| set | 設定 Chaos 使用的排程。 |

## <a name="sfctl-chaos-schedule-get"></a>sfctl chaos schedule get
取得定義 Chaos 執行時間及執行方式的 Chaos 排程。

取得使用中的 Chaos 排程版本，以及定義 Chaos 執行時間及執行方式的 Chaos 排程。

### <a name="arguments"></a>引數

|引數|描述|
| --- | --- |
| --timeout -t | 伺服器超時，用於在幾秒鐘內執行操作。 此超時指定用戶端願意等待請求的操作完成的時間持續時間。 此參數的預設值為 60 秒。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|描述|
| --- | --- |
| --debug | 增加記錄詳細資訊，以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細資訊。 使用 --debug 來取得完整偵錯記錄。 |

## <a name="sfctl-chaos-schedule-set"></a>sfctl chaos schedule set
設定 Chaos 使用的排程。

Chaos 會根據 Chaos 排程自動排定執行。 如果提供的版本與伺服器上的版本匹配，則混沌計畫將更新。 更新混沌計畫時，伺服器上的版本將遞增 1。 到達大量數位後，伺服器上的版本將換回 0。 如果 Chaos 在進行此呼叫時正在執行，此呼叫將會失敗。

### <a name="arguments"></a>引數

|引數|描述|
| --- | --- |
| --chaos-parameters-dictionary | 針對字串名稱對應於作業所要使用的 ChaosParameters，代表對應關係的 JSON 編碼清單。 |
| --expiry-date-utc | 何時該停止使用排程來排定 Chaos 的日期和時間。  預設值\: 9999-12-31T23\:59\:59.999Z。 |
| --jobs | 代表何時該執行 Chaos 及要搭配哪些參數來執行 Chaos 的 ChaosScheduleJobs JSON 編碼清單。 |
| --start-date-utc | 何時該開始使用排程來排定 Chaos 的日期和時間。  預設值\: 1601-01-01T00\:00\:00.000Z。 |
| --timeout -t | 預設值\: 60。 |
| --version | 排程的版本號碼。 |

### <a name="global-arguments"></a>全域引數

|引數|描述|
| --- | --- |
| --debug | 增加記錄詳細資訊，以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細資訊。 使用 --debug 來取得完整偵錯記錄。 |

### <a name="examples"></a>範例

下列命令會將排程設定為 (假設目前的排程版本為 0) 在 2016-01-01 開始並在 2038-01-01 到期，一週 7 天、每天 24 小時執行 Chaos。
Chaos 將會排定在該時間於叢集上執行。
```
sfctl chaos schedule set --version 0 --start-date-utc "2016-01-01T00:00:00.000Z" --expiry-date-utc "2038-01-01T00:00:00.000Z"
    --chaos-parameters-dictionary
    [
    {
        "Key":"adhoc",
        "Value":{
            "MaxConcurrentFaults":3,
            "EnableMoveReplicaFaults":true,
            "ChaosTargetFilter":{
                "NodeTypeInclusionList":[
                "N0010Ref",
                "N0020Ref",
                "N0030Ref",
                "N0040Ref",
                "N0050Ref"
                ]
            },
            "MaxClusterStabilizationTimeoutInSeconds":60,
            "WaitTimeBetweenIterationsInSeconds":15,
            "WaitTimeBetweenFaultsInSeconds":30,
            "TimeToRunInSeconds":"600",
            "Context":{
                "Map":{
                "test":"value"
                }
            },
            "ClusterHealthPolicy":{
                "MaxPercentUnhealthyNodes":0,
                "ConsiderWarningAsError":true,
                "MaxPercentUnhealthyApplications":0
            }
        }
    }
    ]
    --jobs
    [
    {
        "ChaosParameters":"adhoc",
        "Days":{
            "Sunday":true,
            "Monday":true,
            "Tuesday":true,
            "Wednesday":true,
            "Thursday":true,
            "Friday":true,
            "Saturday":true
        },
        "Times":[
            {
                "StartTime":{
                "Hour":0,
                "Minute":0
                },
                "EndTime":{
                "Hour":23,
                "Minute":59
                }
            }
        ]
    }
    ]
```



## <a name="next-steps"></a>後續步驟
- [設置](service-fabric-cli.md)服務結構 CLI。
- 了解如何使用[範例指令碼](/azure/service-fabric/scripts/sfctl-upgrade-application)來使用 Service Fabric CLI。

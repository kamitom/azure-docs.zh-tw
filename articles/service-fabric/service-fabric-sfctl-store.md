---
title: Azure 服務結構 CLI-sfctl 存儲
description: 瞭解 sfctl，Azure 服務結構命令列介面。 包括用於在群集映射存儲上執行檔級操作的命令清單。
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 75c62b54ff3aa7f3af344aa3e1ca81d431ae0ab2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76905620"
---
# <a name="sfctl-store"></a>sfctl store
在叢集映像存放區上執行基本檔案層級作業。

## <a name="commands"></a>命令

|Command|描述|
| --- | --- |
| delete | 刪除現有映像存放區內容。 |
| root-info | 取得映像存放區根的內容資訊。 |
| stat | 取得映像存放區內容資訊。 |

## <a name="sfctl-store-delete"></a>sfctl store delete
刪除現有映像存放區內容。

刪除在指定映像存放區相對路徑內找到的現有映像存放區內容。 此命令可以用來刪除已上傳且佈建的應用程式套件。

### <a name="arguments"></a>引數

|引數|描述|
| --- | --- |
| --content-path [必要] | 映像存放區中檔案或資料夾與其根的相對路徑。 |
| --timeout -t | 伺服器超時，用於在幾秒鐘內執行操作。 此超時指定用戶端願意等待請求的操作完成的時間持續時間。 此參數的預設值為 60 秒。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|描述|
| --- | --- |
| --debug | 增加記錄詳細資訊，以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細資訊。 使用 --debug 來取得完整偵錯記錄。 |

## <a name="sfctl-store-root-info"></a>sfctl store root-info
取得映像存放區根的內容資訊。

傳回位於映像存放區根目錄中映像存放區內容的相關資訊。

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

## <a name="sfctl-store-stat"></a>sfctl store stat
取得映像存放區內容資訊。

傳回位於所指定 contentPath 映像存放區內容的相關資訊。 contentPath 是相對於映像存放區根目錄的路徑。

### <a name="arguments"></a>引數

|引數|描述|
| --- | --- |
| --content-path [必要] | 映像存放區中檔案或資料夾與其根的相對路徑。 |
| --timeout -t | 伺服器超時，用於在幾秒鐘內執行操作。 此超時指定用戶端願意等待請求的操作完成的時間持續時間。 此參數的預設值為 60 秒。  預設值\: 60。 |

### <a name="global-arguments"></a>全域引數

|引數|描述|
| --- | --- |
| --debug | 增加記錄詳細資訊，以顯示所有偵錯記錄。 |
| --help -h | 顯示此說明訊息並結束。 |
| --output -o | 輸出格式。  允許的值\:json、jsonc、table、tsv。  預設值\:json。 |
| --query | JMESPath 查詢字串。 如需詳細資訊和範例，請參閱 http\://jmespath.org/。 |
| --verbose | 增加記錄詳細資訊。 使用 --debug 來取得完整偵錯記錄。 |


## <a name="next-steps"></a>後續步驟
- [設定](service-fabric-cli.md) Service Fabric CLI。
- 了解如何使用[範例指令碼](/azure/service-fabric/scripts/sfctl-upgrade-application)來使用 Service Fabric CLI。
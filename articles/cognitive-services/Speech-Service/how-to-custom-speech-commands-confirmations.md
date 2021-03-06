---
title: 如何：向自訂命令添加確認（預覽）
titleSuffix: Azure Cognitive Services
description: 在本文中，如何實現自訂命令中命令的確認。
services: cognitive-services
author: encorona-ms
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/05/2019
ms.author: encorona
ms.openlocfilehash: afa197c83b4f66f12863de4185ef7763447f3ed9
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "75462397"
---
# <a name="how-to-add-a-confirmation-to-a-custom-command-preview"></a>如何：向自訂命令添加確認（預覽）

在本文中，您將學習如何向命令添加確認。

## <a name="prerequisites"></a>Prerequisites

您必須已完成以下文章中的步驟：

- [快速入門：創建自訂命令（預覽）](./quickstart-custom-speech-commands-create-new.md)
- [快速入門：使用參數創建自訂命令（預覽）](./quickstart-custom-speech-commands-create-parameters.md)

## <a name="create-a-setalarm-command"></a>創建設置警報命令

為了演示驗證，讓我們創建一個新命令，允許使用者設置警報。

1. 在[語音工作室](https://speech.microsoft.com/)中打開以前創建的自訂命令應用程式
1. 創建新的命令**集警報**
1. 添加名為 DateTime 的參數

   | 設定           | 建議的值                                          | 描述                                                                                      |
   | ----------------- | ---------------------------------------------------------| ------------------------------------------------------------------------------------------------ |
   | 名稱              | Datetime                                                 | 命令參數的描述性名稱                                                    |
   | 必要          | true                                                     | 選中核取方塊，指示在完成命令之前是否需要此參數的值 |
   | 回應範本 | "-什麼時間？                                           | 提示在不知道此參數時請求該參數的值                              |
   | 類型              | Datetime                                                 | 參數的類型，如數位、字串或日期時間                                      |
   | 日期 預設值     | 如果日期今天缺少使用                             |                                                                                                  |
   | 時間預設值     | 如果時間缺少使用，則從一天開始                      |                                                                                                  | 

1. 添加一些示例句子
   
   ```
    set an alarm for {DateTime}
    set alarm {DateTime}
    alarm for {DateTime}
   ```

1. 添加完成規則以確認結果

   | 設定    | 建議的值                                         | 描述                                        |
   | ---------- | ------------------------------------------------------- | -------------------------------------------------- |
   | 規則名稱  | 設置鬧鐘                                               | 描述規則目的的名稱          |
   | 動作    | 語音回應 - "-確定，為 [DateTime]設置警報"       | 規則條件為 true 時要執行的操作 |

## <a name="try-it-out"></a>試做

選擇"測試"面板並嘗試一些交互。

- 輸入：為明天中午設置鬧鐘
- 輸出："好的，2019年6月12日12：00：00的報警設置"

- 輸入：設置報警
- 輸出："什麼時間？
- 輸入：下午 5 點
- 輸出："好的，鬧鐘設置為 12/05/2019 17：00：00"

## <a name="add-the-advanced-rules-for-confirmation"></a>添加確認的高級規則

1. 添加高級規則進行確認。 

    此規則將要求使用者確認警報的日期和時間，並期待下一個回合的確認（是/否）。

   | 設定               | 建議的值                                                                  | 描述                                        |
   | --------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------- |
   | 規則名稱             | 確認日期時間                                                                | 描述規則目的的名稱          |
   | 條件            | 所需參數 - 日期時間                                                    | 確定規則何時可以運行的條件    |   
   | 動作               | 語音回應 - "-是否確實要為 [DateTime]設置鬧鐘？"       | 規則條件為 true 時要執行的操作 |
   | 執行後的狀態 | 等待輸入                                                                   | 轉彎後使用者的狀態                  |
   | 期望          | 確認                                                                     | 對下一輪的預期                      |

1. 添加高級規則以處理成功的確認（使用者說是）

   | 設定               | 建議的值                                                                  | 描述                                        |
   | --------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------- |
   | 規則名稱             | 已接受確認                                                            | 描述規則目的的名稱          |
   | 條件            | 成功確認&所需參數 - 日期時間                           | 確定規則何時可以運行的條件    |   
   | 執行後的狀態 | 準備完成                                                             | 轉彎後使用者的狀態                   |

1. 添加高級規則以處理拒絕確認（使用者說"否"）

   | 設定               | 建議的值                                                                  | 描述                                        |
   | --------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------- |
   | 規則名稱             | 拒絕確認                                                                   | 描述規則目的的名稱          |
   | 條件            | 拒絕確認&必需參數 - 日期時間                               | 確定規則何時可以運行的條件    |   
   | 動作               | 清除參數 - 語音回應&日期時間 - "-沒問題，什麼時候？     | 規則條件為 true 時要執行的操作 |
   | 執行後的狀態 | 等待輸入                                                                   | 轉彎後使用者的狀態                   |
   | 期望          | 引引參數 - 日期時間                                                      | 對下一輪的預期                      |

## <a name="try-it-out"></a>試做

選擇"測試"面板並嘗試一些交互。

- 輸入：為明天中午設置鬧鐘
- 輸出："是否確實要為 2019 年 7 月 12 日 12：00：00 設置鬧鐘？"
- 輸入： 否
- 輸出："沒問題，什麼時候？
- 輸入：下午 5 點
- 輸出："是否確實要為 2019 年 6 月 12 日 17：00：00 設置鬧鐘？"
- 輸入： 是
- 輸出："好的，鬧鐘設置為 12/06/2019 17：00：00"

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [如何：向自訂命令添加一步更正（預覽）](./how-to-custom-speech-commands-one-step-correction.md)
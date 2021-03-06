---
title: 多設備對話（預覽） - 語音服務
titleSuffix: Azure Cognitive Services
description: ''
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: dapine
ms.openlocfilehash: b3802e66b0ba5a68c898e69ec64b01edce1541c1
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79371353"
---
# <a name="what-is-multi-device-conversation-preview"></a>什麼是多設備對話（預覽）？

**多設備對話**便於在多個用戶端之間創建語音或文本對話，並協調它們之間發送的消息。

使用**多設備對話**，您可以：

- 將多個用戶端連接到同一會話中，並管理它們之間的消息的發送和接收。
- 通過可選翻譯，輕鬆轉錄每個用戶端的音訊並將轉錄發送給其他人。
- 通過可選翻譯，在用戶端之間輕鬆發送文本消息。

可以構建可在一系列設備中工作的功能或解決方案。 每個設備可以獨立地將消息（音訊或即時消息的轉錄）發送到所有其他設備。

對話[**轉錄**](conversation-transcription.md)在具有多通道麥克風陣列的單個設備上工作，**而多設備對話**適用于具有多個設備（每個設備具有單個麥克風）的方案。

>[!IMPORTANT]
> 多設備**對話不支援在**用戶端之間發送音訊檔：僅轉錄和/或翻譯。

## <a name="key-features"></a>主要功能

- **即時轉錄**– 每個人都會收到對話的腳本，以便他們可以即時跟蹤文本或將其保存以供以後使用。
- **即時翻譯**– 使用超過 60 種[支援的文本翻譯語言](language-support.md#text-languages)，使用者可以將對話翻譯成他們的首選語言。
- **可讀的抄本**– 轉錄和翻譯易於理解，帶有標點符號和句子中斷。
- **語音或文本輸入**– 每個使用者可以在自己的設備上說話或鍵入內容，具體取決於為參與者所選語言啟用的語言支援功能。 請參閱[語言支援](language-support.md#speech-to-text)。
- **消息中繼**- 多設備會話服務將以他們選擇的語言將一個用戶端發送的消息分發給所有其他用戶端。
- **消息標識**– 使用者在對話中收到的每條消息都將用發送該消息的使用者的昵稱進行標記。

## <a name="use-cases"></a>使用案例

### <a name="lightweight-conversations"></a>羽量級對話

創建和加入對話非常簡單。 一個使用者將充當"主機"並創建一個對話，該對話生成隨機的五個字母對話代碼和 QR 碼。 所有其他使用者可以通過鍵入對話代碼或掃描 QR 碼加入對話。 

由於使用者通過對話代碼加入，並且不需要共用聯繫資訊，因此可以輕鬆創建快速的現場對話。

### <a name="inclusive-meetings"></a>包容性會議

即時轉錄和翻譯可以説明說不同語言和/或失聰或聽力障礙的人訪問對話。 每個人還可以通過說出自己的首選語言或發送即時消息來積極參與對話。

### <a name="presentations"></a>簡報

您還可以為螢幕上和聽眾自己的設備上的演示和講座提供字幕。 在訪問群體加入對話代碼後，他們可以在自己的設備上以首選語言查看成績單。

> [!NOTE]
> 要查看示例，請查看[演示文稿轉換器](https://www.microsoft.com/translator/apps/presentation-translator/)，一個使用多設備會話服務的 PowerPoint 外接程式。 您可以從[這裡](https://www.microsoft.com/download/details.aspx?id=55024)下載。

## <a name="how-it-works"></a>運作方式

所有用戶端都將使用語音 SDK 創建或加入對話。 語音 SDK 與多設備會話服務進行交互，該服務管理會話的存留期，包括參與者清單、每個用戶端選擇的語言和發送的消息。  

每個用戶端都可以發送音訊或即時消息。 該服務將使用語音辨識將音訊轉換為文本，並根據需要發送即時消息。 如果用戶端選擇不同的語言，則服務將將所有消息轉換為每個用戶端的指定語言。

![多設備對話概述圖](media/scenarios/multi-device-conversation.png)

## <a name="overview-of-conversation-host-and-participant"></a>對話、主持人和參與者概述

**會話**是一個使用者為其他參與使用者加入而啟動的會話。 所有用戶端都使用五個字母**的對話代碼**連接到對話。

每個對話創建中繼資料，其中包括：
-    對話開始和結束時間戳記
-    對話中所有參與者的清單，其中包括每個使用者為語音或文本輸入選擇的昵稱和主要語言。


對話中有兩種類型的使用者：**主機**和**參與者**。

**主機**是啟動對話的使用者，充當該對話的管理員。
- 每個對話只能有一個主機
- 在會話期間，主機必須連接到會話。 如果主持人離開對話，所有其他參與者的對話將結束。
- 主機有幾個額外的控制項來管理對話： 
    - 鎖定對話 - 防止其他參與者加入
    - 使所有參與者靜音 - 阻止其他參與者向對話發送任何消息，無論是從語音或即時消息中轉錄的
    - 將單個參與者靜音
    - 取消隱藏所有參與者
    - 取消取消單個參與者的靜音

**參與者**是加入對話的使用者。
- 參與者可以隨時離開並重新加入同一對話，而不會結束其他參與者的對話。
- 參與者無法鎖定對話或靜音/取消靜音他人

> [!NOTE]
> 每次對話最多可以有 100 名參與者，其中 10 個可以在任意給定時間同時發言。

## <a name="language-support"></a>語言支援

創建或加入對話時，每個使用者必須選擇**主要語言**：他們將使用的語言和發送即時消息的語言，以及他們將看到其他使用者的消息的語言。

有兩種語言：**語音到文本**和**純文字**：
- 如果使用者選擇**語音到文本**語言作為主要語言，那麼他們將能夠在對話中使用語音和文本輸入。

- 如果使用者選擇**純文字**語言，則他們只能使用文本輸入並在對話中發送即時消息。 純文字語言是文本翻譯支援的語言，但不支援語音到文本的語言。 您可以在[語言支援](supported-languages.md)頁面上查看可用語言。

除了主要語言之外，每個參與者還可以指定用於翻譯會話的其他語言。

以下是使用者根據所選主要語言在多設備對話中能夠執行的操作的摘要。


| 使用者可以在對話中執行哪些操作 | 語音轉文字 | 純文字 |
|-----------------------------------|----------------|------|
| 使用語音輸入 | ✔️ | ❌ |
| 發送即時消息 | ✔️ | ✔️ |
| 翻譯對話 | ✔️ | ✔️ |

> [!NOTE]
> 有關可用語音到文本和文本翻譯語言的清單，請參閱[支援的語言](supported-languages.md)。



## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [即時翻譯對話](quickstarts/multi-device-conversation.md)

---
title: 意向和實體 - LUIS
titleSuffix: Azure Cognitive Services
description: 單個意圖表示使用者要執行的任務或操作。 它是使用者語句中表達的目的或目標。 請定義一組與使用者想要在您應用程式中執行之動作對應的意圖。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/10/2019
ms.author: diberry
ms.openlocfilehash: 309a2592dbac2918aeb532fbe91e33d296f4e5a5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79221507"
---
# <a name="intents-in-your-luis-app"></a>LUIS 應用中的意圖

意圖代表使用者想要執行的工作或動作。 它是使用者[話語](luis-concept-utterance.md)中表達的目的或目標。

請定義一組與使用者想要在您應用程式中執行之動作對應的意圖。 例如，旅遊應用程式會定義數個意圖：

旅遊應用程式意圖   |   範例語句   | 
------|------|
 BookFlight     |   「幫我預訂下週到里約的班機」 <br/> 「24 日讓我飛到里約」 <br/> 「我需要一張下週日到里約熱內盧的機票」    |
 Greeting     |   「嗨」 <br/>"Hello" <br/>「早安」  |
 CheckWeather | 「波士頓的天氣如何？」 <br/> 「顯示本週的預測」 |
 None         | 「給我餅乾食譜」<br>「湖人隊贏了嗎？」 |

所有應用程式都附帶預定義的意圖，"[無](#none-intent)"，這是回退意圖。 

## <a name="prebuilt-domains-provide-intents"></a>預先建置的定義域會提供意圖
除了定義的意圖之外，還可以使用預[構建的域](luis-how-to-use-prebuilt-domains.md)之一的預構建意圖。 

## <a name="return-all-intents-scores"></a>傳回所有意圖的分數
您會指派語句給單一意圖。 預設情況下，當 LUIS 收到終結點上的話語時，它將返回該陳述的最高意圖。 

如果希望說出所有意圖的分數，則可以在預測 API 的查詢字串上提供標誌。 

|預測 API 版本|旗標|
|--|--|
|V2|`verbose=true`|
|V3|`show-all-intents=true`|

## <a name="intent-compared-to-entity"></a>意圖與實體的比較
意圖表示應用程式應該對使用者執行的操作，並且基於整個陳述。 一個語句只能有一個最高分的意圖，但可以有許多個實體。 

<a name="how-do-intents-relate-to-entities"></a>

 當使用者的「意圖」__ 會觸發用戶端應用程式中的動作 (例如對 checkweather() 函式的呼叫) 時，請建立意圖。 然後創建實體以表示執行操作所需的參數。 

|Intent   | 單位 | 範例語句   | 
|------------------|------------------------------|------------------------------|
| CheckWeather | { "type": "location", "entity": "seattle" }<br>{ "type": "builtin.datetimeV2.date","entity": "tomorrow","resolution":"2018-05-23" } | 天氣`Seattle``tomorrow`怎麼樣？ |
| CheckWeather | { "type": "date_range", "entity": "this weekend" } | Show me the forecast for `this weekend` | 
||||

## <a name="prebuilt-domain-intents"></a>預先建置的定義域意圖

[預構建的域](luis-how-to-use-prebuilt-domains.md)提供具有陳述的意圖。 

## <a name="none-intent"></a>None 意圖

系統會建立 **None** 但刻意保留空白。 **None** 意圖是必要意圖，您無法將其刪除或重新命名。 請以您定義域外的語句填入它。

**"無**"意圖是回退意圖，在每個應用中都很重要，並且應具有總陳述的 10%。 它可用來教導 LUIS 在應用程式定義域 (主題區域) 中不重要的語句。 如果您沒有為 **None** 意圖新增任何語句，LUIS 就會強制讓定義域外的某個語句變成其中一個定義域意圖。 這會教導 LUIS 錯誤的語句意圖而扭曲預測分數。 

當陳述被預測為"無"意圖時，用戶端應用程式可以提出更多問題或提供功能表以引導使用者選擇有效選項。 

## <a name="negative-intentions"></a>負面意圖 
如果您想要判斷負面或正面意圖，例如「我**想要**一輛車」和「我**不**想要一輛車」，您可以建立兩個意圖 (一個正面，一個負面)，然後為每個意圖新增適當的語句。 或者，您也可以建立單一意圖，並標示兩個不同的正面和負面字詞作為實體。  

## <a name="intents-and-patterns"></a>意圖和模式

如果您有示例陳述（可以作為正則運算式在部分或整體中定義），請考慮使用與[模式](luis-concept-patterns.md)配對的[正則運算式實體](luis-concept-entity-types.md#regular-expression-entity)。 

使用正則運算式實體可以保證資料提取，以便匹配模式。 模式匹配保證返回確切的意圖。 

## <a name="intent-balance"></a>意圖平衡
應用程式定義域應該讓各個意圖之間的語句平衡。 請勿讓一個意圖有 10 個語句，而另一個意圖有 500 個語句。 這樣會造成不平衡。 如果您有這種情況，請檢閱有 500 個語句的意圖，看看是否可以將許多意圖都重新組織成[模式](luis-concept-patterns.md)。 

此平衡並不包括 **None** 意圖。 該意圖所包含的語句應該佔應用程式中總語句的 10%。

## <a name="intent-limits"></a>意圖限制
請檢閱[限制](luis-boundaries.md#model-boundaries)，以了解您可以將多少個意圖新增到模型中。 

### <a name="if-you-need-more-than-the-maximum-number-of-intents"></a>如果您所需的意圖超出意圖數目上限 
請先想想您的系統是否使用太多意圖。 

### <a name="can-multiple-intents-be-combined-into-single-intent-with-entities"></a>是否可以將多個意圖結合成含有實體的單一意圖 
太相似的意圖會讓 LUIS 更難以區分它們。 意圖應該要有足夠的差異，才能擷取使用者所要求的主要工作，但它們並不需要擷取您程式碼所採取的每個路徑。 例如，BookFlight 和 FlightCustomerService 可能是旅遊應用程式中的個別意圖，但 BookInternationalFlight 和 BookDomesticFlight 則太相似。 如果您的系統需要區分它們，請使用實體或其他邏輯，而不要使用意圖。 

### <a name="dispatcher-model"></a>發送器模型
深入了解如何使用[分派模型](luis-concept-enterprise.md#when-you-need-to-combine-several-luis-and-qna-maker-apps)將 LUIS 與 QnA Maker 應用程式結合。 

### <a name="request-help-for-apps-with-significant-number-of-intents"></a>針對含有大量意圖的應用程式要求協助
如果減少意圖數目或將意圖分割成多個應用程式對您並不適用，請與支援人員連絡。 如果您的 Azure 訂用帳戶包含支援服務，請與 [Azure 技術支援人員](https://azure.microsoft.com/support/options/)連絡。 

## <a name="next-steps"></a>後續步驟

* 深入了解[實體](luis-concept-entity-types.md)，這些是與意圖相關的重要單字
* 了解如何在 LUIS 應用程式中[新增及管理意圖](luis-how-to-add-intents.md)。
* 檢閱意圖[最佳做法](luis-concept-best-practices.md)

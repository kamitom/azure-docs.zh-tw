---
title: 教學課程：模式 - LUIS
titleSuffix: Azure Cognitive Services
description: 在本教學課程中，您將可以在提供較少語句範例的情況下，使用模式來提高意圖和實體預測準確度。 此模式是以範本語句範例的方式來提供，其中包含用來識別實體及可忽略文字的語法。
services: cognitive-services
author: diberry
ms.custom: seodec18
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 12/17/2019
ms.author: diberry
ms.openlocfilehash: 69894dfc6bcbe9eb56451524c78e82da2745aa52
ms.sourcegitcommit: fe6c9a35e75da8a0ec8cea979f9dec81ce308c0e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/26/2020
ms.locfileid: "75979773"
---
# <a name="tutorial-add-common-pattern-template-utterance-formats-to-improve-predictions"></a>教學課程：新增通用模式範本語句格式以改善預測

在本教學課程中，使用模式來提高意圖和實體預測準確度，其可讓您提供較少的範例語句。 此模式是指派給意圖的範本語句，其中包含可識別實體和可忽略文字的語法。

**在本教學課程中，您將了解如何：**

> [!div class="checklist"]
> * 建立模式
> * 確認模式預測改進情況
> * 將文字標示為可忽略並在模式中建立巢狀結構
> * 使用測試面板來確認模式已成功

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="utterances-in-intent-and-pattern"></a>意圖和模式中的語句

LUIS 應用程式中儲存了兩種類型的語句：

* 意圖中的範例語句
* 模式中的範本語句

以模式新增範本語句，可讓您針對意圖提供較少的範例語句。

模式會以運算式比對與機器學習的組合套用。  範本語句以及範例語句，可讓 LUIS 更容易理解哪些語句符合意圖。

## <a name="import-example-app-and-clone-to-new-version"></a>匯入範例應用程式並複製到新版本

使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/tutorials/custom-domain-batchtest-HumanResources.json?raw=true)。

1. 將 JSON 匯入[預覽 LUIS 入口網站](https://preview.luis.ai)中的新應用程式。

1. 從 [管理]  區段的 [版本]  索引標籤上，複製版本並將它命名為 `patterns`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。

## <a name="create-new-intents-and-their-utterances"></a>建立新意圖及其語句

1. 從導覽列中選取 [建置]  。

1. 在 [意圖]  頁面上，選取 [+ 建立]  以建立新意圖。

1. 在快顯對話方塊方塊中輸入 `OrgChart-Manager`，然後選取 [完成]  。

    ![建立新的訊息快顯視窗](media/luis-tutorial-pattern/hr-create-new-intent-popup.png)

1. 將語句範例新增至意圖。 這些語句並不完全  相同，但有一個可以解取的模式。

    |範例語句|
    |--|
    |`Who is John W. Smith the subordinate of?`|
    |`Who does John W. Smith report to?`|
    |`Who is John W. Smith's manager?`|
    |`Who does Jill Jones directly report to?`|
    |`Who is Jill Jones supervisor?`|

    如果是在意圖的語句 (而不是員工實體的語句) 中標示 keyPhrase 實體，也沒關係。 這兩者都可在端點的 [測試] 窗格中正確預測。

1. 在左側導覽中，選取 [意圖]  。

1. 選取 [+ 建立]  以建立新的意圖。 在快顯對話方塊方塊中輸入 `OrgChart-Reports`，然後選取 [完成]  。

1. 將語句範例新增至意圖。

    |範例語句|
    |--|
    |`Who are John W. Smith's subordinates?`|
    |`Who reports to John W. Smith?`|
    |`Who does John W. Smith manage?`|
    |`Who are Jill Jones direct reports?`|
    |`Who does Jill Jones supervise?`|

### <a name="caution-about-example-utterance-quantity"></a>關於範例語句數量的注意事項

[!INCLUDE [Too few examples](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]

### <a name="train-the-app-before-testing-or-publishing"></a>在測試或發佈之前訓練應用程式

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

### <a name="publish-the-app-to-query-from-the-endpoint"></a>發佈應用程式以從端點進行查詢

[!INCLUDE [LUIS How to Publish steps](includes/howto-publish.md)]

### <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. 移至位址中的 URL 結尾並輸入 `Who is the boss of Jill Jones?`。 最後一個查詢字串參數是語句 `query`。

    ```json
    {
        "query": "Who is the boss of Jill Jones?",
        "prediction": {
            "topIntent": "OrgChart-Manager",
            "intents": {
                "OrgChart-Manager": {
                    "score": 0.313054234
                },
                "OrgChart-Reports": {
                    "score": 0.2462688
                },
                "EmployeeFeedback": {
                    "score": 0.0488328524
                },
                "GetJobInformation": {
                    "score": 0.0156933
                },
                "MoveEmployee": {
                    "score": 0.011265873
                },
                "Utilities.StartOver": {
                    "score": 0.003065792
                },
                "Utilities.Stop": {
                    "score": 0.00300148362
                },
                "Utilities.Cancel": {
                    "score": 0.00271081156
                },
                "None": {
                    "score": 0.00212835032
                },
                "ApplyForJob": {
                    "score": 0.0020669254
                },
                "Utilities.Confirm": {
                    "score": 0.00200891262
                },
                "FindForm": {
                    "score": 0.00194145238
                },
                "Utilities.Help": {
                    "score": 0.00182301877
                }
            },
            "entities": {
                "keyPhrase": [
                    "boss of Jill Jones"
                ],
                "Employee": [
                    [
                        "Employee-45612"
                    ]
                ],
                "$instance": {
                    "keyPhrase": [
                        {
                            "type": "builtin.keyPhrase",
                            "text": "boss of Jill Jones",
                            "startIndex": 11,
                            "length": 18,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "Employee": [
                        {
                            "type": "Employee",
                            "text": "Jill Jones",
                            "startIndex": 19,
                            "length": 10,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

這個查詢成功了嗎？ 對於此定型週期而言並未成功。 兩個熱門意圖的分數已接近，但最高意圖並沒有明顯很高 (超過 60%)，且未遠遠超過下一個意圖的分數。

因為 LUIS 定型每次都不會完全一樣，會有一些變化，所以，這兩個分數可能會在下一個定型週期中反轉。 結果可能會傳回錯誤的意圖。

使用模式來讓正確意圖的分數大幅提高 (以百分比表示)，並拉大與下一個最高分數的差距。

讓這裡的第二個瀏覽器視窗保持開啟。 您稍後會在本教學課程中使用到它。

## <a name="template-utterances"></a>範本語句
由於人力資源定義域的本質，因此有一些常見的方式來詢問組織中的員工關係。 例如：

|表達方式|
|--|
|`Who does Jill Jones report to?`|
|`Who reports to Jill Jones?`|

這些語句太相似，以致若不提供許多語句範例，便無法判斷各個語句中內容相關的唯一性。 藉由為意圖新增模式，無須提供許多語句範例，LUIS 便可學習意圖的常見語句模式。

此意圖的範本語句範例包括：

|範本語句的範例|語法意義|
|--|--|
|`Who does {Employee} report to[?]`|可交換 `{Employee}`<br>忽略 `[?]`|
|`Who reports to {Employee}[?]`|可交換 `{Employee}`<br>忽略 `[?]`|

`{Employee}` 語法不僅會標示是哪一個實體，也會標示實體在範本語句內的位置。 選擇性的語法 `[?]` 會標記選擇性的單字或標點符號。 LUIS 會比對語句，並略過括號內的選擇性文字。

雖然語法看起來像是規則運算式，但它不是規則運算式。 只支援大括號 `{}` 和方括號 `[]` 語法。 它們的巢狀結構最多可以有兩個層級。

為了讓模式與語句進行比對，語句內的實體必須先比對範本語句中的實體。 這表示在具有實體的模式成功之前，實體必須在範例語句中有足夠的範例並具有高程度的預測。 不過，此範本無助於預測實體，只適用於意圖。

**當模式可讓您提供較少的範例語句時，如果偵測不到實體，模式就不會進行比對。**

### <a name="add-the-patterns-for-the-orgchart-manager-intent"></a>新增 OrgChart-Manager 意圖的模式

1. 在上方功能表中，選取 [建置]  。

1. 在左導覽窗格中的 [Improve app performance] \(改善應用程式效能\)  底下，選取 [Patterns] \(模式\)  。

1. 選取 [OrgChart-Manager]  意圖，然後輸入下列範本語句：

    |範本語句|
    |:--|
    |`Who is {Employee} the subordinate of[?]`|
    |`Who does {Employee} report to[?]`|
    |`Who is {Employee}['s] manager[?]`|
    |`Who does {Employee} directly report to[?]`|
    |`Who is {Employee}['s] supervisor[?]`|
    |`Who is the boss of {Employee}[?]`|

1. 同樣在 [模式] 頁面上，選取 [OrgChart-Reports]  意圖，然後輸入下列範本語句：

    |範本語句|
    |:--|
    |`Who are {Employee}['s] subordinates[?]`|
    |`Who reports to {Employee}[?]`|
    |`Who does {Employee} manage[?]`|
    |`Who are {Employee} direct reports[?]`|
    |`Who does {Employee} supervise[?]`|
    |`Who does {Employee} boss[?]`|

### <a name="query-endpoint-when-patterns-are-used"></a>使用模式時查詢端點

模式現在已新增至應用程式，請在預測執行階段端點定型、發佈及查詢應用程式。

1. 選取 [訓練]  。 定型完成之後，請選取 [發佈]  並選取 [生產]  位置，然後選取 [完成]  。

1. 發佈完成後，將瀏覽器索引標籤切換回 [端點 URL] 索引標籤。

1. 移至位址中的 URL 結尾並輸入 `Who is the boss of Jill Jones?` 作為語句。 最後一個查詢字串參數是 `query`。

    ```json
    {
        "query": "Who is the boss of Jill Jones?",
        "prediction": {
            "topIntent": "OrgChart-Manager",
            "intents": {
                "OrgChart-Manager": {
                    "score": 0.999997854
                },
                "OrgChart-Reports": {
                    "score": 6.13748343E-05
                },
                "EmployeeFeedback": {
                    "score": 8.052567E-06
                },
                "GetJobInformation": {
                    "score": 1.18197136E-06
                },
                "MoveEmployee": {
                    "score": 7.65549657E-07
                },
                "None": {
                    "score": 3.975E-09
                },
                "Utilities.StartOver": {
                    "score": 1.53E-09
                },
                "Utilities.Confirm": {
                    "score": 1.38181822E-09
                },
                "Utilities.Help": {
                    "score": 1.38181822E-09
                },
                "Utilities.Stop": {
                    "score": 1.38181822E-09
                },
                "Utilities.Cancel": {
                    "score": 1.25833333E-09
                },
                "FindForm": {
                    "score": 1.15384613E-09
                },
                "ApplyForJob": {
                    "score": 5.26923061E-10
                }
            },
            "entities": {
                "keyPhrase": [
                    "boss of Jill Jones"
                ],
                "Employee": [
                    [
                        "Employee-45612"
                    ]
                ],
                "$instance": {
                    "keyPhrase": [
                        {
                            "type": "builtin.keyPhrase",
                            "text": "boss of Jill Jones",
                            "startIndex": 11,
                            "length": 18,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "Employee": [
                        {
                            "type": "Employee",
                            "text": "Jill Jones",
                            "startIndex": 19,
                            "length": 10,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

意圖預測現在更有信心，而下一個最高意圖的分數明顯降低。 這兩個意圖不會在定型時翻轉。

### <a name="working-with-optional-text-and-prebuilt-entities"></a>使用選用文字和預先建置的實體

本教學課程中的前幾個模式範本語句有一些選用文字範例，例如使用字母 s 的所有格用法 (`'s`)，以及問號用法 (`?`)。 假設您需要允許在語句文字中使用目前和未來日期。

範例語句如下：

|Intent|包含選用文字和預先建置實體的語句範例|
|:--|:--|
|OrgChart-Manager|`Who was Jill Jones manager on March 3?`|
|OrgChart-Manager|`Who is Jill Jones manager now?`|
|OrgChart-Manager|`Who will be Jill Jones manager in a month?`|
|OrgChart-Manager|`Who will be Jill Jones manager on March 3?`|

這裡的每句範例都使用動詞時態 `was`、`is`、`will be`，以及日期 `March 3`、`now` 和 `in a month`，而 LUIS 必須正確地預測這些內容。 請注意，除了 `in` 和 `on` 之外，資料表中的最後兩個範例使用幾乎相同的文字。

允許這項選擇性資訊的範本語句範例：

|Intent|包含選用文字和預先建置實體的語句範例|
|:--|:--|
|OrgChart-Manager|`who was {Employee}['s] manager [[on]{datetimeV2}?`]|
|OrgChart-Manager|`who is {Employee}['s] manager [[on]{datetimeV2}?]`|


使用方括號 `[]`的選用語法可讓此選用文字輕鬆地加入範本語句，並且能以巢狀方式 `[[]]` 增加至第二個層級，然後包含實體或文字。


**問題：為什麼所有 `w` 字母 (每個範本語句的第一個字母) 都是小寫？不能選擇大寫或小寫嗎？** 用戶端應用程式提交至查詢端點的語句會轉換成小寫。 範本語句可以是大寫或小寫，而端點語句也可以是大寫或小寫。 但比對一律會在轉換成小寫之後完成。

**問題：如果 March 3 (3 月 3 日) 會預測為數字 `3` 及日期 `March 3`，為什麼預先建置的號碼不是範例語句的一部分？** 根據上下文，此範本語句使用的是日期，日期可以是常值 (`March 3`) 或抽象表示 (`in a month`)。 日期可以包含數字，但數字不一定會被視為日期。 每次使用實體時，該實體應要最能代表要在預測 JSON 結果中傳回的類型。

**問題：為什麼剖析 `Who will {Employee}['s] manager be on March 3?` 之類的語句時，效果很差。** 文法上，不同動詞時態必須是新的範本語句 (例如此處的 `will` 和 `be` 是分開的)。 現有範本語句並不符合此原則。 雖然語句的意圖沒有變更，但字組在語句中的位置已變更。 此變更會影響 LUIS 中的預測。 您可以使用 [group 和 or ](#use-the-or-operator-and-groups) 處理動詞時態，以結合這些語句。

**請記住：系統會先找出實體，然後比對模式。**

### <a name="edit-the-existing-pattern-template-utterance"></a>編輯現有的模式範本語句

1. 在預覽 LUIS 入口網站中，選取頂端功能表中的 [建置]  ，然後在左側功能表中選取 [模式]  。

1. 搜尋現有範本語句 `Who is {Employee}['s] manager[?]`，選取右側的省略符號 (...)，然後從快顯功能表中選取 [編輯]  。

1. 將範本語句變更為：`who is {Employee}['s] manager [[on]{datetimeV2}?]`

### <a name="add-new-pattern-template-utterances"></a>新增模式範本語句

1. 當您還在 [建置]  的 [模式]  中時，可以新增數個模式範本語句。 從意圖下拉式功能表中選取 [OrgChart Manager]  ，並輸入以下每個範本語句：

    |Intent|包含選用文字和預先建置實體的語句範例|
    |--|--|
    |OrgChart-Manager|`who was {Employee}['s] manager [[on]{datetimeV2}?]`|
    |OrgChart-Manager|`who will be {Employee}['s] manager [[in]{datetimeV2}?]`|
    |OrgChart-Manager|`who will be {Employee}['s] manager [[on]{datetimeV2}?]`|

2. 選取導覽列中的 [定型]  來進行應用程式定型。

3. 定型完成之後，在面板頂端選取 [測試]  ，以開啟測試面板。

4. 輸入數個測試語句，以驗證模式符合，而且意圖分數相當高。

    輸入第一個語句之後，請選取結果下方的 [檢查]  ，然後您就可以看到所有預測結果。 每個語句應該都有 **OrgChart-Manager** 意圖，且應該擷取 Employee 和 datetimeV2 實體的值。

    |語句|
    |--|
    |`Who will be Jill Jones manager`|
    |`who will be jill jones's manager`|
    |`Who will be Jill Jones's manager?`|
    |`who will be Jill jones manager on March 3`|
    |`Who will be Jill Jones manager next Month`|
    |`Who will be Jill Jones manager in a month?`|

所有這些語句中都可找到實體，因此這些語句皆符合相同模式，並且具有很高的預測分數。 您已新增幾個模式，以符合許多語句變化。 您不需要在意圖中新增任何範例語句，即可讓範本語句在模式中運作。

此種模式用法提供了：
* 較高的預測分數
* 意圖中具有相同的範例語句
* 模式中僅有少數結構良好的範本語句

### <a name="use-the-or-operator-and-groups"></a>使用 OR 運算子和群組

先前數個範本語句都非常接近。 使用 **group** `()` 和 **OR** `|` 語法來減少範本語句。

使用 group `()` 和 OR `|` 語法，可以將下列 2 個模式合併成單一模式。

|Intent|包含選用文字和預先建置實體的語句範例|
|--|--|
|OrgChart-Manager|`who will be {Employee}['s] manager [[in]{datetimeV2}?]`|
|OrgChart-Manager|`who will be {Employee}['s] manager [[on]{datetimeV2}?]`|

新的範本語句會是：

第 1 課：建立 Windows Azure 儲存體物件`who ( was | is | will be ) {Employee}['s] manager [([in]|[on]){datetimeV2}?]`。

這會使用 **group** 圍住必要的動詞時態，以及使用選用的 `in` 和 `on` (兩者之間有 **or** 垂直線)。

1. 在 [模式]  頁面上，選取 **OrgChart-Manager** 篩選器。 藉由搜尋 `manager` 來縮小清單。

1. 保留一個範本語句版本 (以在下一個步驟中編輯) 並刪除其他變化。

1. 將範本語句變更為：

    `who ( was | is | will be ) {Employee}['s] manager [([in]|[on]){datetimeV2}?]`

2. 選取導覽列中的 [定型]  來進行應用程式定型。

3. 定型完成之後，在面板頂端選取 [測試]  ，以開啟測試面板。

    使用 [測試] 窗格來測試語句的版本：

    |要在 [測試] 窗格中輸入的語句|
    |--|
    |`Who is Jill Jones manager this month`|
    |`Who is Jill Jones manager on July 5th`|
    |`Who was Jill Jones manager last month`|
    |`Who was Jill Jones manager on July 5th`|
    |`Who will be Jill Jones manager in a month`|
    |`Who will be Jill Jones manager on July 5th`|

使用更多模式語法，您可減少您必須在應用程式中維護的範本語句數目，同時仍有高預測分數。

### <a name="use-the-utterance-beginning-and-ending-anchors"></a>使用語句開頭和結尾錨點

模式語法提供插入號 `^` 的開頭和結尾語句錨點。 開頭和結尾語句錨點可一起用來瞄準以非常明確且可能字面的語句，或分別來瞄準意圖。

## <a name="using-patternany-entity"></a>使用 Pattern.any 實體

Pattern.any 實體可讓您尋找自由格式的資料，其中實體的用字方式使其很難從語句的剩餘部分判斷實體的結尾。

這個人力資源應用程式可協助員工尋找公司表單。

|語句|
|--|
|**HRF-123456** 在哪裡？|
|**HRF-123234** 的作者是誰？|
|**HRF-456098** 是以法文發行的嗎？|

不過，每個表單都具備一個格式化的名稱 (上表所使用的名稱) 及一個易記名稱 (例如 `Request relocation from employee new to the company 2018 version 5`)。

含有易記表單名稱的語句看起來如下：

|語句|
|--|
|**新進員工要求調職 2018 年第 5 版**在哪裡？|
|**「新進員工要求調職 2018 年第 5 版」** 的作者是誰？|
|**新進員工要求調職 2018 年第 5 版**是以法文發行的嗎？|

這個變動長度包含可能造成 LUIS 無法正確辨識實體結束位置的字組。 在模式中使用 Pattern.any 實體可讓您指定表單名稱的開頭和結尾，讓 LUIS 能夠正確擷取表單名稱。

|範本語句的範例|
|--|
|{FormName} 在哪裡[?]|
|{FormName} 的作者是誰[?]|
|{FormName} 是以法文發行的嗎[?]|

### <a name="add-example-utterances-with-patternany"></a>使用 Pattern.any 新增範例語句

1. 從上方瀏覽列中選取 [建置]  ，然後從左方瀏覽列中選取 [意圖]  。

1. 從意圖清單中選取 [FindForm]  。

1. 新增一些範例語句：

    |範例語句|表單名稱|
    |--|--|
    |**實驗室發生火災時該怎麼辦**的表單在哪裡，而且當我閱讀該表單之後需要誰簽名？|實驗室發生火災時該怎麼辦
    |**新進員工要求調職**位於伺服器上的何處？|新進員工要求調職|
    |「**主要校園的健康與福利要求**」的作者是誰且最新版本為何？|主要校園的健康與福利要求|
    |我正在尋找名為「包含實體資產在內的辦公室搬遷要求」  的表單。 |Office 移動要求，包括實體資產|

    在沒有 Pattern.any 實體的情況下，LUIS 很難理解表單標題結束位置，因為表單名稱有許多種變化。

### <a name="create-a-patternany-entity"></a>建立 Pattern.any 實體
Pattern.any 實體可擷取各種不同長度的實體。 其僅適用於模式，因為模式會以語法標示實體的開頭和結尾。

1. 從左側導覽窗格中選取 [Entities] \(實體\)  。

1. 選取 [+ 建立]  、輸入名稱 `FormName`，然後選取 [Pattern.any]  作為類型。 選取 [建立]  。

### <a name="add-a-pattern-that-uses-the-patternany"></a>新增使用 Pattern.any 的模式

1. 從左側導覽窗格中選取 [Patterns] \(模式\)  。

1. 選取 [FindForm]  意圖。

1. 輸入下列使用新實體的範本語句：

    |範本語句|
    |--|
    |`Where is the form ["]{FormName}["] and who needs to sign it after I read it[?]`|
    |`Where is ["]{FormName}["] on the server[?]`|
    |`Who authored ["]{FormName}["] and what is the most current version[?]`|
    |`I'm looking for the form named ["]{FormName}["][.]`|

1. 將應用程式定型。

### <a name="test-the-new-pattern-for-free-form-data-extraction"></a>測試新模式以擷取自由格式資料
1. 從頂端列選取 [Test] \(測試\)  來開啟測試面板。

1. 輸入下列語句：

    `Where is the form Understand your responsibilities as a member of the community and who needs to sign it after I read it?`

1. 選取結果底下的 [Inspect] \(檢查\)  ，以查看實體和意圖的測試結果。

    系統會先尋找實體 `FormName`，然後才尋找可判斷意圖的模式。 如果您的測試結果未偵測到實體，因而未找到模式，您就必須針對意圖新增更多範例語句 (而非模式)。

1. 選取頂端導覽列中的 [Test] \(測試\)  按鈕。

### <a name="using-an-explicit-list"></a>使用明確的清單

如果您發現模式在包含 Pattern.any 時所擷取的實體不正確，請使用[明確清單](reference-pattern-syntax.md#explicit-lists)來更正此問題。

## <a name="what-did-this-tutorial-accomplish"></a>本教學課程有何成果？

本教學課程新增了一些模式，可協助 LUIS 以明顯較高的分數預測意圖，而不需要新增更多範例語句。 標記實體及可忽略的文字可讓 LUIS 對更廣泛的語句套用模式。

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟


> [!div class="nextstepaction"]
> [了解如何將角色與模式搭配使用](luis-tutorial-pattern.md)

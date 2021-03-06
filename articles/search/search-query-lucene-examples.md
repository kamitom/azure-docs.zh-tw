---
title: 使用完整的 Lucene 查詢文法
titleSuffix: Azure Cognitive Search
description: Lucene 查詢語法,用於 Azure 認知搜索服務中的模糊搜索、鄰近搜索、術語提升、正則表達式搜索和通配符搜索。
manager: nitinme
author: HeidiSteen
ms.author: heidist
tags: Lucene query analyzer syntax
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 3c54f864b5bd562fdc0a84b2903198704032b360
ms.sourcegitcommit: 25490467e43cbc3139a0df60125687e2b1c73c09
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/09/2020
ms.locfileid: "80998491"
---
# <a name="use-the-full-lucene-search-syntax-advanced-queries-in-azure-cognitive-search"></a>使用「完整」Lucene 搜尋語法(Azure 認知搜尋中的進階查詢)

為 Azure 認知搜尋建構查詢時,可以將預設[的簡單查詢解析器](query-simple-syntax.md)替換為 Azure 認知搜索中較寬泛的[Lucene 查詢解析器](query-lucene-syntax.md),以制定專門和高級的查詢定義。 

Lucene 解析器支援複雜的查詢建構,例如欄位範圍查詢、模糊和首配符搜索、鄰近搜索、術語提升和正則表達式搜索。 額外的處理需求需要更多能源，因此，您應該預期執行時間會略久。 在本文中，您可以逐步執行範例，以示範使用完整語法時可用的查詢作業。

> [!Note]
> 許多透過完整 Lucene 查詢語法來啟用的特製化查詢建構都不是[文字分析](search-lucene-query-architecture.md#stage-2-lexical-analysis)，因此如果您預期的是詞幹分析或詞形歸併還原，可能會感到意外。 只能對完整詞彙執行語彙分析 (詞彙查詢或片語查詢)。 不完整詞彙的查詢類型 (前置詞查詢、萬用字元查詢、Regex 查詢、模糊查詢) 會直接新增至查詢樹狀結構，並略過分析階段。 只能對不完整的查詢詞彙執行小寫轉換。 
>

## <a name="formulate-requests-in-postman"></a>以 Postman 編寫要求

下列範例會根據 [紐約市 OpenData](https://opendata.cityofnewyork.us/) 計劃所提供的資料集，利用由可用工作組成的 NYC 工作搜尋索引。 這項資料不應視為目前的或已完成。 索引位於 Microsoft 提供的沙箱服務上,這意味著您不需要 Azure 訂閱或 Azure 認知搜尋來嘗試這些查詢。

您的需要是 Postman，或可對 GET 發出 HTTP 要求的對等工具。 如需詳細資訊，請參閱[使用 REST 用戶端瀏覽](search-get-started-postman.md)。

### <a name="set-the-request-header"></a>設定要求標頭

1. 在要求標頭中，將 **Content-Type** 設定為 `application/json`。

2. 新增 **api-key**，並將其設定為此字串：`252044BE3886FE4A8E3BAA4F595114BB`。 這是裝載 NYC 工作索引的沙箱搜尋服務所適用的查詢金鑰。

在指定要求標頭後，您可以將其重複用於本文中的所有查詢，只要替換掉 **search=** 字串即可。 

  ![Postman 要求標頭](media/search-query-lucene-examples/postman-header.png)

### <a name="set-the-request-url"></a>設定要求 URL

請求是一個 GET 命令,它與包含 Azure 認知搜索終結點和搜索字串的 URL 配對。

  ![Postman 要求標頭](media/search-query-lucene-examples/postman-basic-url-request-elements.png)

URL 組合具有下列元素：

+ **`https://azs-playground.search.windows.net/`** 是 Azure 認知搜索開發團隊維護的沙盒搜索服務。 
+ **`indexes/nycjobs/`** 是該服務索引集合中的 NYC 作業索引。 要求上必須同時有服務名稱和索引。
+ **`docs`** 是包含所有可搜尋內容的文檔集合。 要求標頭中提供的查詢 API 金鑰僅適用於以文件集合為目標的讀取作業。
+ **`api-version=2019-05-06`** 設置 api 版本,它是每個請求所需的參數。
+ **`search=*`** 查詢字串,在初始查詢中為空,返回前 50 個結果(預設情況下)。

## <a name="send-your-first-query"></a>傳送第一個查詢

在驗證步驟中，將下列要求貼到 GET 中，然後按一下 [傳送]****。 結果會以詳細 JSON 文件的形式傳回。 傳回整個文件,這允許您查看所有欄位和所有值。

將此 URL 貼上到 REST 用戶端作為驗證步驟,並查看文件結構。

  ```http
  https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&search=*
  ```

查詢字串**`search=*`** 是等效於空搜索或空搜索的未指定的搜索。 這是最簡單的搜索,你可以做。

或者,您可以添加到**`$count=true`** URL 以傳回與搜尋條件匹配的文件計數。 在空的搜尋字串上，這會是索引中的所有文件 (在 NYC 作業的案例中大約有 2800 份)。

## <a name="how-to-invoke-full-lucene-parsing"></a>如何叫用完整 Lucene 剖析

新增 **queryType=full** 以叫用完整查詢語法，覆寫預設的簡單查詢語法。 

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&search=*
```

本文中的所有範例會指定 **queryType=full** 搜尋參數，表示 Lucene 查詢剖析器處理的完整語法。 

## <a name="example-1-query-scoped-to-a-list-of-fields"></a>範例 1:查詢範圍為欄位清單

第一個示例不是特定於 Lucene,但我們率先引入了第一個基本查詢概念:欄位範圍。 此示例僅對幾個特定欄位的範圍為整個查詢和回應。 當您的工具是 Postman 或搜尋總管時，了解如何建構可讀取的 JSON 回應很重要。 

為求簡潔，查詢僅以 *business_title* 欄位為目標，且指定僅傳回公司職稱。 **"搜尋欄位'** 參數將查詢執行限制為僅business_title欄位,並**選擇指定**回應中包括哪些欄位。

### <a name="search-expression"></a>搜尋運算式

```http
&search=*&searchFields=business_title&$select=business_title
```

下面是逗號分隔清單中具有多個字段的相同查詢。

```http
search=*&searchFields=business_title, posting_type&$select=business_title, posting_type
```

逗號之後的空格是可選的。

> [!Tip]
> 使用應用程式代碼中的 REST API 時,不要忘記網址編`$select`碼`searchFields`參數,如與 。

### <a name="full-url"></a>完整 URL

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&search=*&searchFields=business_title&$select=business_title
```

此查詢的回應會如下列螢幕擷取畫面所示。

  ![Postman 範例回應](media/search-query-lucene-examples/postman-sample-results.png)

您可能已注意到在回應中的搜尋分數。 沒有排名時，分數一律為 1，這是因為搜尋不是全文檢索搜尋，或是未套用任何準則。 若是未套用任何準則的 Null 搜尋，資料列會以任意順序傳回。 當您包含實際搜索條件時,您將看到搜索分數演變為有意義的值。

## <a name="example-2-fielded-search"></a>範例 2:欄位搜尋

完全 Lucene 語法支援將單個搜尋運算式範圍範圍範圍到特定欄位。 本示例搜索具有「高級」一詞的業務名稱,而不是初級名稱。

### <a name="search-expression"></a>搜尋運算式

```http
$select=business_title&search=business_title:(senior NOT junior)
```

下面是具有多個字段的相同查詢。

```http
$select=business_title, posting_type&search=business_title:(senior NOT junior) AND posting_type:external
```

### <a name="full-url"></a>完整 URL

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&$select=business_title&search=business_title:(senior NOT junior)
```

  ![Postman 範例回應](media/search-query-lucene-examples/intrafieldfilter.png)

您可以使用**欄位Name:searchExpression**語法定義欄位搜索操作,其中搜尋運算式可以是單一單字或短語,也可以是括弧中的更複雜的運算式,可選使用布林運算元。 部分範例如下：

- `business_title:(senior NOT junior)`
- `state:("New York" OR "New Jersey")`
- `business_title:(senior NOT junior) AND posting_type:external`

如果希望將兩個字串計算為單個實體,請確保在引號中放置多個字串,例如在本例中搜索`state`欄位中的兩個不同的位置。 此外，請確定運算子是大寫，如同您看到的 NOT 和 AND。

**欄位名稱:搜尋表示式**中指定的欄位必須是可搜索欄位。 有關索引屬性在欄位定義中如何使用的詳細資訊[,請參閱創建索引(Azure 認知搜索 REST API)。](https://docs.microsoft.com/rest/api/searchservice/create-index)

> [!NOTE]
> 在上面的範例中,我們不需要使用 參數`searchFields`, 因為查詢的每個部分都有顯式指定的欄位名稱。 但是,如果要運行查詢,`searchFields`其中某些部分限定在特定欄位,其餘部分可以應用於多個字段,則仍可以使用 參數。 例如,`search=business_title:(senior NOT junior) AND external&searchFields=posting_type`查詢將`senior NOT junior`僅`business_title`與欄位匹配,而它將「外部」`posting_type`與欄位匹配。 **欄位名稱:searchExpression**中提供的欄位名稱始終優先`searchFields`於 參數,因此在此示例中,我們`business_title``searchFields`不需要在 參數中包括。

## <a name="example-3-fuzzy-search"></a>範例 3：模糊搜尋

完整 Lucene 語法也支援模糊搜尋，會比對具有類似建構的字詞。 若要執行模糊搜尋，請在單一文字結尾附加波狀符號 `~`，加上選擇性參數 (介於 0 和 2 之間且會指定編輯距離的值)。 比方說，`blue~` 或 `blue~1` 會傳回 blue、blues 和 glue。

### <a name="search-expression"></a>搜尋運算式

```http
searchFields=business_title&$select=business_title&search=business_title:asosiate~
```

直接支援短語,但您可以對短語的元件部分指定模糊匹配。

```http
searchFields=business_title&$select=business_title&search=business_title:asosiate~ AND comm~ 
```


### <a name="full-url"></a>完整 URL

此查詢會搜尋含有 "associate" 一詞 (刻意拼錯) 的工作︰

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:asosiate~
```
  ![模糊搜尋回應](media/search-query-lucene-examples/fuzzysearch.png)


> [!Note]
> 模糊查詢不會進行[分析](search-lucene-query-architecture.md#stage-2-lexical-analysis)。 不完整詞彙的查詢類型 (前置詞查詢、萬用字元查詢、Regex 查詢、模糊查詢) 會直接新增至查詢樹狀結構，並略過分析階段。 只能對不完整的查詢詞彙執行小寫轉換。
>

## <a name="example-4-proximity-search"></a>範例 4：鄰近搜尋
鄰近搜尋可用來尋找文件中彼此相近的詞彙。 在片語的結尾插入波狀符號 "~"，後面加上建立鄰近界限的字數。 例如，"hotel airport"~5 會在文件中每 5 個字內尋找旅館和機場等詞彙。

### <a name="search-expression"></a>搜尋運算式

```http
searchFields=business_title&$select=business_title&search=business_title:%22senior%20analyst%22~1
```

### <a name="full-url"></a>完整 URL

在此查詢中，會搜尋含有 "senior analyst" 一詞，且將其分隔的字數不超過一個字的工作︰

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:%22senior%20analyst%22~1
```
  ![鄰近查詢](media/search-query-lucene-examples/proximity-before.png)

移除 "senior analyst" 一詞之間的單字，然後再試一次。 請注意，相較於上一個查詢傳回 10 份文件，此查詢僅傳回 8 份文件。

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:%22senior%20analyst%22~0
```

## <a name="example-5-term-boosting"></a>範例 5：詞彙提升
提升一詞指的是如果文件包含提升詞彙，則將其評等提高，高於不包含該詞彙的文件。 若要提升詞彙，請使用插入號 "^"，並在搜尋詞彙的結尾加上提升係數 (數字)。 

### <a name="full-urls"></a>完整 URL

在此 "before" 查詢中，搜尋含有 *computer analyst* 的工作時，我們會發現沒有同時包含 *computer* 和 *analyst* 兩個單字的結果，但是 *computer* 工作顯示於結果頂端。

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:computer%20analyst
```
  ![before 的詞彙提升](media/search-query-lucene-examples/termboostingbefore.png)

在 "after" 查詢中再次執行此搜尋，但這次在兩個單字未同時存在的情況下，提升含有 *analyst* 一詞的結果，使其高於 *computer* 一詞。 

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:computer%20analyst%5e2
```
上述查詢較易讀的版本為 `search=business_title:computer analyst^2`。 在可行的查詢中，`^2` 會編碼為 `%5E2`，因此難以檢視。

  ![after 的詞彙提升](media/search-query-lucene-examples/termboostingafter.png)

詞彙提升與評分設定檔的不同之處在於，評分設定檔會提升特定欄位，而不是特定字詞。 下列範例可協助說明差異。

請考慮使用評分設定檔，可提高特定欄位中的相符項目，例如 musicstoreindex 範例中的 **genre** 。 詞彙提升可用來進一步提升某些搜尋詞彙，使其高於其他詞彙。 比方說，"rock^2 electronic" 可提升包含搜尋詞彙的文件﹐使其在 [genre] **** 欄位中高於索引中的其他可搜尋欄位。 此外，包含搜尋詞彙 "rock" 的文件排名會比另一個搜尋詞彙 "electronic" 還高，此為詞彙提升值 (2) 的結果。

在設定係數層級時，提升係數越高，該字詞相對於其他搜尋字詞的關聯性也越高。 根據預設，提升係數為 1。 雖然提升係數必須是正數，但是它可能會小於 1 (例如，0.2)。


## <a name="example-6-regex"></a>範例 6：Regex

規則運算式搜尋會根據正斜線 "/" 之間的內容尋找相符項目，如 [RegExp 類別](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/util/automaton/RegExp.html)中所記錄。

### <a name="search-expression"></a>搜尋運算式

```http
searchFields=business_title&$select=business_title&search=business_title:/(Sen|Jun)ior/
```

### <a name="full-url"></a>完整 URL

在此查詢中,搜索術語"高級"或"初級:"`search=business_title:/(Sen|Jun)ior/`的作業。

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:/(Sen|Jun)ior/
```

  ![RegEx 查詢](media/search-query-lucene-examples/regex.png)

> [!Note]
> RegEx 查詢不會進行[分析](https://docs.microsoft.com/azure/search/search-lucene-query-architecture#stage-2-lexical-analysis)。 只能對不完整的查詢詞彙執行小寫轉換。
>

## <a name="example-7-wildcard-search"></a>範例 7：萬用字元搜尋
您可以使用一般辨識語法進行多個 (\*) 或單一 (?) 字元的萬用字元搜尋。 請注意，Lucene 查詢剖析器支援搭配使用這些符號與單一詞彙，而不是片語。

### <a name="search-expression"></a>搜尋運算式

```http
searchFields=business_title&$select=business_title&search=business_title:prog*
```

### <a name="full-url"></a>完整 URL

在此查詢中，搜尋包含前置詞 'prog' 的工作，其中包括內含 programming 與 programmer 的職稱。 您無法使用 * 或 ? 符號做為搜尋的第一個字元。

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:prog*
```
  ![萬用字元查詢](media/search-query-lucene-examples/wildcard.png)

> [!Note]
> 萬用字元查詢不會進行[分析](https://docs.microsoft.com/azure/search/search-lucene-query-architecture#stage-2-lexical-analysis)。 只能對不完整的查詢詞彙執行小寫轉換。
>

## <a name="next-steps"></a>後續步驟
請在您的程式碼中嘗試指定 Lucene 查詢剖析器。 下列連結說明如何設定 .NET 和 REST API 的搜尋查詢。 這些連結會使用預設的簡單語法，因此您必須套用您從本文了解的內容來指定 **queryType**。

* [使用 .NET SDK 查詢索引](search-query-dotnet.md)
* [使用 REST API 查詢索引](search-create-index-rest-api.md)

您可以在下列連結中找到其他語法參考、查詢架構和範例：

+ [簡單語法查詢範例](search-query-simple-examples.md)
+ [全文檢索搜尋如何在 Azure 認知搜尋中運作](search-lucene-query-architecture.md)
+ [簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [完整的 Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)
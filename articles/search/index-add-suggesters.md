---
title: 建立建議工具
titleSuffix: Azure Cognitive Search
description: 通過創建建議器和制定調用自動完成或自動建議的查詢詞的請求,在 Azure 認知搜尋中啟用提前鍵入的查詢操作。
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/10/2020
ms.openlocfilehash: a6c4051a5b3d557f9ac2927a62492425e7636c0d
ms.sourcegitcommit: fb23286d4769442631079c7ed5da1ed14afdd5fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/10/2020
ms.locfileid: "81113475"
---
# <a name="create-a-suggester-to-enable-autocomplete-and-suggestions-in-azure-cognitive-search"></a>建立建議程式以在 Azure 認知搜尋中開啟自動完成與建議

在 Azure 認知搜尋中,「按類型搜尋」或「鍵入」功能基於添加到[搜索索引](search-what-is-an-index.md)**的建議器**建構。 建議程式支援兩個「即用即用」 搜尋變體:*自動完成*, 它完成要鍵入的字語或短語,以及傳回符合文件的簡短清單*的建議*。  

以下螢幕截圖(來自[C# 示例中的第一個應用](tutorial-csharp-type-ahead-and-suggestions.md))說明了這兩種體驗。 自動完成預測使用者可能鍵入的內容,以"in"作為潛在搜索詞完成"tw"。 建議是實際搜尋結果,每個搜尋結果表示匹配的文檔。 有關建議,可以顯示文檔中最能描述結果的任何部分。 在此示例中,建議由酒店名稱欄位表示。

![自動完成與建議的查詢的視覺化比較](./media/index-add-suggesters/hotel-app-suggestions-autocomplete.png "自動完成與建議的查詢的視覺化比較")

要在 Azure 認知搜索中實現這些行為,有一個索引和查詢元件。 

+ 在索引中,向索引添加建議器。 您可以使用門戶[、REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)或[.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.suggester?view=azure-dotnet)。 本文的其餘部分側重於創建建議器。

+ 在查詢請求中,調用[下面列出的 API](#how-to-use-a-suggester)之一。

基於每個欄位啟用"按類型搜索"支援。 如果您希望體驗類似於螢幕截圖中所示的體驗,則可以在同一搜索解決方案中實現兩種排位前行行為。 這兩個請求都以特定索引*的文檔*集合為目標,並在使用者至少提供三個字元輸入字串後返回回應。

## <a name="what-is-a-suggester"></a>什麼是建議者?

建議程式是一種數據結構,通過存儲用於在部分查詢上匹配的首碼,支援按類型搜索行為。 與標記化術語類似,首碼儲存在倒置索引中,對於建議器欄位集合中指定的每個字段一個。

創建首碼時,建議器有自己的分析鏈,類似於用於全文搜索的分析鏈。 但是,與全文搜索中的分析不同,建議器只能使用預定義的分析器(標準 Lucene、[語言分析器](index-add-language-analyzers.md)或[預定義分析器清單中的其他分析器](index-add-custom-analyzers.md#predefined-analyzers-reference))。 [自定義分析器](index-add-custom-analyzers.md)和配置是特別不允許的,以避免產生不良結果的隨機配置。

> [!NOTE]
> 如果需要解決分析器約束,請使用兩個單獨的欄位來訪問同一內容。 這將允許其中一個字段具有建議程式,而另一個字段可以使用自定義分析器配置進行設置。

## <a name="create-a-suggester"></a>建立建議工具

儘管建議器具有多個屬性,但它主要是啟用 Typeahead 體驗的欄位的集合。 例如，旅遊應用程式應該針對目的地、城市及景點啟用鍵盤預先鍵入緩衝搜尋。 因此,所有三個字段都將進入欄位集合中。

要創建建議程式,請向索引架構添加一個。 索引中可以有一個建議器(具體來說,建議者集合中有一個建議器)。

### <a name="when-to-create-a-suggester"></a>何時建立建議器

建立建議器的最佳時間是創建欄位定義本身。

如果嘗試使用預先存在的欄位創建建議程式,API 將禁止它。 在索引期間創建排型文本,當兩個或多個字符組合中的部分術語與整個術語一起標記時。 鑒於現有欄位已標記化,如果要將它們添加到建議器,則必須重新生成索引。 有關重新編制索引的詳細資訊,請參閱[如何重新生成 Azure 認知搜尋索引](search-howto-reindex.md)。

### <a name="create-using-the-rest-api"></a>使用 REST API 建立

在 REST API 中,透過[建立索引](https://docs.microsoft.com/rest/api/searchservice/create-index)或[更新索引](https://docs.microsoft.com/rest/api/searchservice/update-index)添加建議程式。 

  ```json
  {
    "name": "hotels",
    "fields": [
      . . .
    ],
    "suggesters": [
      {
        "name": "sg",
        "searchMode": "analyzingInfixMatching",
        "sourceFields": ["hotelName", "category"]
      }
    ],
    "scoringProfiles": [
      . . .
    ]
  }
  ```

### <a name="create-using-the-net-sdk"></a>使用 .NET SDK 建立

在 C# 中,定義[建議物件](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.suggester?view=azure-dotnet)。 `Suggesters`是集合,但它只能獲取一個專案。 

```csharp
private static void CreateHotelsIndex(SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = "hotels",
        Fields = FieldBuilder.BuildForType<Hotel>(),
        Suggesters = new List<Suggester>() {new Suggester()
            {
                Name = "sg",
                SourceFields = new string[] { "HotelId", "Category" }
            }}
    };

    serviceClient.Indexes.Create(definition);

}
```

### <a name="property-reference"></a>屬性參考

|屬性      |描述      |
|--------------|-----------------|
|`name`        |建議工具的名稱。|
|`searchMode`  |用來搜尋候選片語的策略。 目前唯一支援的模式是 `analyzingInfixMatching`，其可在句子開頭或中間執行彈性的片語比對。|
|`sourceFields`|建議之內容來源的一或多個欄位清單。 欄位必須為類型`Edm.String``Collection(Edm.String)`與 。 如果在現場指定了分析器,它必須是[此清單中](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.analyzername?view=azure-dotnet)的命名分析器(而不是自定義分析器)。<p/>最佳做法是僅指定那些適合預期和適當回應的欄位,無論是搜索欄中的已完成字串還是下拉清單中的已完成字串。<p/>酒店名稱是一個很好的候選者,因為它具有精度。 描述和註釋等詳細欄位過於密集。 同樣,重複欄位(如類別和標記)的效率較低。 在示例中,我們無論如何都會包含"類別",以證明您可以包含多個字段。 |

<a name="how-to-use-a-suggester"></a>

## <a name="use-a-suggester-in-a-query"></a>在查詢中使用建議器

創建建議程式後,調用查詢邏輯中的相應 API 以調用該功能。 

+ [建議 REST API](https://docs.microsoft.com/rest/api/searchservice/suggestions) 
+ [自動完成 REST API](https://docs.microsoft.com/rest/api/searchservice/autocomplete) 
+ [建議使用HttpMessagesasync方法](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.idocumentsoperations.suggestwithhttpmessagesasync?view=azure-dotnet)
+ [使用 HTTPMessagesasync 方法自動完成](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.idocumentsoperations.autocompletewithhttpmessagesasync?view=azure-dotnet&viewFallbackFrom=azure-dotnet)

API 用法在以下對自動完成 REST API 的調用中進行了說明。 此示例有兩個要點。 首先,與所有查詢一樣,該操作針對索引的文檔集合。 其次,您可以添加查詢參數。 雖然許多查詢參數是兩個 API 共同的,但每個 API 的清單都不同。

```http
GET https://[service name].search.windows.net/indexes/[index name]/docs/autocomplete?[query parameters]  
api-key: [admin or query key]
```

如果在索引中未定義建議程式,則自動完成或建議的調用將失敗。

## <a name="sample-code"></a>範例程式碼

+ [在 C# 範例中建立第一個應用](tutorial-csharp-type-ahead-and-suggestions.md)將展示建議器建構、建議查詢、自動完成和分面導航。 此代碼示例在沙箱 Azure 認知搜尋服務上運行並使用預載入的 Hotels 索引,因此所有需要執行的只是按 F5 來運行應用程式。 無需訂閱或登錄。

+ [DotNetToToAutocomplete](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete)是包含 C# 和 Java 代碼的舊範例。 它還演示了建議器建構、建議查詢、自動完成和分面導航。 此代碼示例使用託管的[NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs)範例資料。 


## <a name="next-steps"></a>後續步驟

我們建議使用以下示例來瞭解如何制定請求。

> [!div class="nextstepaction"]
> [建議與自動完成範例](search-autocomplete-tutorial.md) 

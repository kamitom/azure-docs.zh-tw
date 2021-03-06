---
title: 擷取影像中的文字
titleSuffix: Azure Cognitive Search
description: 處理和提取 Azure 認知搜索管道中的圖像中的文本和其他資訊。
manager: nitinme
author: LuisCabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 98054060210f55803d6e2811e1f494fd3ff00e48
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76838253"
---
# <a name="how-to-process-and-extract-information-from-images-in-ai-enrichment-scenarios"></a>如何在 AI 擴充方案中處理和提取圖像中的資訊

Azure 認知搜索具有多個用於處理圖像和影像檔的功能。 在文件破解期間，可以使用 *imageAction* 參數，從含有英數文字的相片或圖片中擷取文字，例如停止標誌中的「STOP」。 其他案例則包括生成影像的代表文字，例如從蒲公英的相片中生成「蒲公英」，或生成顏色「黃色」。 您也可以擷取影像的中繼資料，例如影像的大小。

本文更詳細地介紹影像處理，並提供有關在 AI 擴充管道中處理圖像的指導。

<a name="get-normalized-images"></a>

## <a name="get-normalized-images"></a>取得標準化影像

在文件破解的過程中，會有一組新的索引子設定參數，用於處理影像檔或內嵌在檔案中的影像。 這些參數會用來標準化影像，以進行更進一步的下游處理。 標準化後的影像會更為一致。 大型影像的大小會調整成最大高度和寬度，使其便於使用。 若影像有提供方向的中繼資料，則會旋轉影像以便垂直載入。 系統會以針對每個影像建立的複雜類型，來擷取中繼資料調整項目。 

影像標準化無法關閉。 反覆執行影像的技術會產生標準化影像。 在索引子上啟用映射正常化要求將技能集附加到該索引子。

| 設定參數 | 描述 |
|--------------------|-------------|
| imageAction   | 如果無需對內嵌影像或影像檔案採取任何動作，則設為「none」。 <br/>若設為「generateNormalizedImages」，則會在文件破解期間產生一系列的標準化影像。<br/>設置為"生成正常化的ImagePerPage"以生成一系列正常化圖像，其中對於資料來源中的 PDF，每個頁面呈現到一個輸出圖像。  其功能與適用於非 PDF 檔案類型的 "generateNormalizedImages" 相同。<br/>針對不是 "none" 的所有選項，這些影像將會公開在 *normalized_images* 欄位中。 <br/>預設值為「none」。 當「dataToExtract」設為「contentAndMetadata」時，此設定僅與 blob 資料來源有直接相關。 <br/>最多從給定文檔中提取 1000 個圖像。 如果文檔中有超過 1000 個圖像，將提取前 1000 個圖像，並生成警告。 |
|  normalizedImageMaxWidth | 所產生的標準化影像的最大寬度 (以像素為單位)。 預設值為 2000。 允許的最大值為 10000。 | 
|  normalizedImageMaxHeight | 所產生標準化影像的最大高度 (以像素為單位)。 預設值為 2000。 允許的最大值為 10000。|

> [!NOTE]
> 如果將*imageAction*屬性設置為"無"以外的任何內容，則無法將*解析模式*屬性設置為"預設"以外的任何內容。  只能將這兩個屬性的其中之一設定為索引子設定中的非預設值。

將 **parsingMode** 參數設為 `json` (將每個 blob 當作單一文件加以索引) ，或設為 `jsonArray` (如果您的 blob 包含 JSON 陣列，且您需要將陣列的每個元素視為個別文件)。

標準化影像的最大寬度與最大高度的預設值為 2000 像素，這是根據 [OCR 技術](cognitive-search-skill-ocr.md)和[影像分析技術](cognitive-search-skill-image-analysis.md)所支援的大小上限。 [OCR 技能](cognitive-search-skill-ocr.md)支援非英語語言的最大寬度和高度為 4200，英語支援 10000。  如果增加最大限制，則根據技能集定義和文檔的語言，處理較大的圖像可能會失敗。 

在[索引子定義](https://docs.microsoft.com/rest/api/searchservice/create-indexer)中指定 imageAction，如下所示：

```json
{
  //...rest of your indexer definition goes here ...
  "parameters":
  {
    "configuration": 
    {
        "dataToExtract": "contentAndMetadata",
        "imageAction": "generateNormalizedImages"
    }
  }
}
```

當 *imageAction* 被設為 "none" 以外的值時，新的 *normalized_images* 欄位將會包含影像的陣列。 每個影像皆為具有下列成員的複雜類型：

| 影像成員       | 描述                             |
|--------------------|-----------------------------------------|
| data               | 以 BASE64 編碼的 JPEG 格式標準化影像字串。   |
| width              | 標準化影像的寬度，以像素為單位。 |
| height             | 標準化影像的高度，以像素為單位。 |
| originalWidth      | 影像標準化之前的原始寬度。 |
| originalHeight      | 影像標準化之前的原始高度。 |
| rotationFromOriginal |  逆時針旋轉為了建立標準化影像而發生的度數。 0 度到 360 度之間的值。 此步驟會從相機或掃描器產生的影像中讀取中繼資料。 通常為 90 度的倍數。 |
| contentOffset | 影像擷取來源的內容欄位中的字元位移。 此欄位僅適用於含內嵌影像的檔案。 |
| 頁號 | 如果從 PDF 提取或渲染圖像，則此欄位包含從 1 開始的 PDF 中提取或呈現的圖像的頁碼。  如果圖像不是來自 PDF，則此欄位將為 0。  |

 *normalized_images* 的範例值：
```json
[
  {
    "data": "BASE64 ENCODED STRING OF A JPEG IMAGE",
    "width": 500,
    "height": 300,
    "originalWidth": 5000,  
    "originalHeight": 3000,
    "rotationFromOriginal": 90,
    "contentOffset": 500,
    "pageNumber": 2
  }
]
```

## <a name="image-related-skills"></a>影像相關技術

以下兩種內建的認知技術會以影像作為輸入源：[OCR](cognitive-search-skill-ocr.md) 和[影像分析](cognitive-search-skill-image-analysis.md)。 

目前這類技術只適用於文件破解步驟所產生的影像。 因此，唯一支援的輸入為 `"/document/normalized_images"`。

### <a name="image-analysis-skill"></a>影像分析技能

[圖像分析技能](cognitive-search-skill-image-analysis.md)根據圖像內容提取一組豐富的視覺要素。 例如，您可以從影像產生標題、產生標記，或辨識名人和地標。

### <a name="ocr-skill"></a>OCR 技術

[OCR 技術](cognitive-search-skill-ocr.md)會從 JPG、PNG、點陣圖這類影像檔案中擷取文字。 其可擷取文字及版面配置資訊。 版面配置資訊會為每個識別出的字串提供週框方塊。

## <a name="embedded-image-scenario"></a>內嵌影像案例

常見的案例之一就是透過下列步驟建立單一字串，字串中包含所有的檔案內容，無論是由文字或影像所生的文字：  

1. [擷取 normalized_images](#get-normalized-images)
1. 執行使用 `"/document/normalized_images"` 作為輸入的 OCR 技術
1. 將這些影像的代表文字與擷取自檔案的原始文字合併。 您可以使用[文字合併](cognitive-search-skill-textmerger.md)技術將這兩個文字區塊合併成一個大型字串。

下列的範例技術集會建立 *merged_text* 欄位，內含文件的文字內容。 也包含有每個內嵌影像的 OCRed 文字。 

#### <a name="request-body-syntax"></a>要求本文語法
```json
{
  "description": "Extract text from images and merge with content text to produce merged_text",
  "skills":
  [
    {
        "description": "Extract text (plain and structured) from image.",
        "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
        "context": "/document/normalized_images/*",
        "defaultLanguageCode": "en",
        "detectOrientation": true,
        "inputs": [
          {
            "name": "image",
            "source": "/document/normalized_images/*"
          }
        ],
        "outputs": [
          {
            "name": "text"
          }
        ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "description": "Create merged_text, which includes all the textual representation of each image inserted at the right location in the content field.",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " ",
      "inputs": [
        {
          "name":"text", "source": "/document/content"
        },
        {
          "name": "itemsToInsert", "source": "/document/normalized_images/*/text"
        },
        {
          "name":"offsets", "source": "/document/normalized_images/*/contentOffset" 
        }
      ],
      "outputs": [
        {
          "name": "mergedText", "targetName" : "merged_text"
        }
      ]
    }
  ]
}
```

現在有了 merged_text 欄位，可以將該欄位對應為索引子定義中的可搜尋欄位。 檔案中的所有內容，包括影像的文字，均可供搜尋。

## <a name="visualize-bounding-boxes-of-extracted-text"></a>將擷取文字的週框方塊視覺化

另一個常見的案例是，對於搜尋結果的版面配置資訊進行視覺化處理。 例如，您想要反白顯示影像中所找到的一段文字，作為搜尋結果的一部分。

由於 OCR 步驟是在標準化影像上執行的，因此版面配置座標是在標準化影像空間中。 當顯示標準化影像時，座標的存在通常不是問題，但是在某些情況下您可能想要顯示原始影像。 在此情況下，請將版面配置中的每個座標點轉換成原始影像座標系統。 

建議您，如果必須將標準化的座標轉換為原始座標空間，可以使用下列演算法：

```csharp
        /// <summary>
        ///  Converts a point in the normalized coordinate space to the original coordinate space.
        ///  This method assumes the rotation angles are multiples of 90 degrees.
        /// </summary>
        public static Point GetOriginalCoordinates(Point normalized,
                                    int originalWidth,
                                    int originalHeight,
                                    int width,
                                    int height,
                                    double rotationFromOriginal)
        {
            Point original = new Point();
            double angle = rotationFromOriginal % 360;

            if (angle == 0 )
            {
                original.X = normalized.X;
                original.Y = normalized.Y;
            } else if (angle == 90)
            {
                original.X = normalized.Y;
                original.Y = (width - normalized.X);
            } else if (angle == 180)
            {
                original.X = (width -  normalized.X);
                original.Y = (height - normalized.Y);
            } else if (angle == 270)
            {
                original.X = height - normalized.Y;
                original.Y = normalized.X;
            }

            double scalingFactor = (angle % 180 == 0) ? originalHeight / height : originalHeight / width;
            original.X = (int) (original.X * scalingFactor);
            original.Y = (int)(original.Y * scalingFactor);

            return original;
        }
```

## <a name="see-also"></a>另請參閱
+ [創建索引子 （REST）](https://docs.microsoft.com/rest/api/searchservice/create-indexer)
+ [影像分析技能](cognitive-search-skill-image-analysis.md)
+ [OCR 技術](cognitive-search-skill-ocr.md) (英文)
+ [文字合併技術](cognitive-search-skill-textmerger.md) (英文)
+ [如何定義技能集](cognitive-search-defining-skillset.md) (英文)
+ [如何對應豐富型欄位](cognitive-search-output-field-mapping.md) (英文)

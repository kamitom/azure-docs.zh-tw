---
title: 防止翻譯內容 - Translator Text API
titleSuffix: Azure Cognitive Services
description: 使用 Translator Text API 防止翻譯內容。 Translator Text API 可讓您標記內容，使其不要翻譯。
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 03/20/2020
ms.author: swmachan
ms.openlocfilehash: c7be4a0ea1a9d24a8b262132632a0bbb63ae1b96
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80052475"
---
# <a name="how-to-prevent-translation-of-content-with-the-translator-text-api"></a>如何使用 Translator Text API 防止翻譯內容

Translator Text API 可讓您標記內容，使其不要翻譯。 例如，您可以標記程式碼、品牌名稱，或當地語系化後沒有意義的字組/片語。

## <a name="methods-for-preventing-translation"></a>防止翻譯的方法

1. 使用 `notranslate` 標記您的內容。 設計上，只有當輸入文本類型設置為 HTML 時，才有效

   範例：

   ```html
   <span class="notranslate">This will not be translated.</span>
   <span>This will be translated. </span>
   ```
   
   ```html
   <div class="notranslate">This will not be translated.</div>
   <div>This will be translated. </div>
   ```

2. 使用[動態字典](dynamic-dictionary.md)來規定特定翻譯。

3. 請勿將字串傳遞至 Translator Text API 進行翻譯。

4. 自訂翻譯人員：使用[自訂翻譯器中的字典](custom-translator/what-is-dictionary.md)來規定概率為 100% 的短語的翻譯。


## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [避免在 Translator API 呼叫中翻譯](reference/v3-0-translate.md)

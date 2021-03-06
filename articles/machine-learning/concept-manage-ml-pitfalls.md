---
title: 使用 AutoML 避免過度擬合&不平衡的資料
titleSuffix: Azure Machine Learning
description: 使用 Azure 機器學習的自動機器學習解決方案識別和管理 ML 模型的常見缺陷。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 04/09/2020
ms.openlocfilehash: 76f920ad6aae68defb567a7a6623d1ffd488af5f
ms.sourcegitcommit: 2d7910337e66bbf4bd8ad47390c625f13551510b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80874852"
---
# <a name="prevent-overfitting-and-imbalanced-data-with-automated-machine-learning"></a>透過自動機器學習防止資料過度擬合和不平衡

在構建機器學習模型時,過度擬合和不平衡的數據是常見的陷阱。 默認情況下,Azure 機器學習的自動機器學習提供圖表和指標,以説明您識別這些風險,並實現最佳實踐來幫助緩解這些風險。 

## <a name="identify-over-fitting"></a>識別過度擬合

當模型過於適合訓練數據,因此無法對看不見的測試數據進行準確預測時,就會發生機器學習中過度擬合的情況。 換句話說,該模型只是記住了訓練數據中的特定模式和雜訊,但不夠靈活,無法對真實數據進行預測。

考慮以下經過培訓的模型及其相應的訓練和測試精度。

| 模型 | 列車精度 | 測試精度 |
|-------|----------------|---------------|
| A | 99.9% | 95% |
| B | 87% | 87% |
| C | 99.9% | 45% |

考慮到模型**A,** 有一個常見的誤解,即如果對看不見的數據的測試精度低於訓練精度,則模型的擬合度過高。 但是,測試準確性應始終低於訓練精度,而過度擬合與適當擬合之間的區別歸結為準確性低*得多*。 

在比較模型**A**和**B**時,模型**A**是一個更好的模型,因為它具有較高的測試精度,雖然測試精度略低,為 95%,但表明存在過度擬合併不顯著差異。 你不會選擇型號**B**只是因為火車和測試精度是更接近在一起。

**型號 C**表示過度擬合的明顯情況;訓練精度很高,但測試精度並不高。 這種區分是主觀的,但來自於對您的問題和數據的瞭解,以及可接受的錯誤程度。

## <a name="prevent-over-fitting"></a>防止過度擬合

在最令人震驚的情況下,過度擬合的模型將假定在訓練期間看到的要素值組合始終會導致目標完全相同的輸出。

防止過度擬合的最佳方法是遵循 ML 最佳實踐,包括:

* 使用更多培訓數據,消除統計偏差
* 防止目標洩漏
* 使用較少的功能
* **正規化和超參數最佳化**
* **模型複雜性限制**
* **交叉驗證**

在自動 ML 的內容中,上述前三個項目是**您實現的最佳做法**。 最後三個粗體專案是**默認情況下用於**防止過度擬合的自動 ML 實現。 在自動化 ML 以外的設置中,所有六種最佳實踐都值得遵循,以避免模型過度擬合。

### <a name="best-practices-you-implement"></a>您設定的最佳做法

使用**更多數據**是防止過度擬合的最簡單和最佳方法,作為額外的好處通常能提高準確性。 當您使用更多數據時,模型將更難記住確切的模式,並且它被迫到達更靈活的解決方案,以適應更多條件。 識別**統計偏差**也很重要,以確保您的訓練數據不包括即時預測數據中不存在的孤立模式。 此方案可能難以解決,因為您的列車和測試組之間可能沒有過度擬合,但與實時測試數據相比,可能存在過度擬合。

目標洩漏是一個類似的問題,您可能看不到列車/測試組之間的過度擬合,而是出現在預測時間。 當模型在訓練期間通過訪問預測時通常不應擁有的數據而"作弊"時,就會發生目標洩漏。 例如,如果您的問題是在週一預測商品價格在星期五是多少,但您的功能之一意外包含了週四的數據,那麼這將是模型在預測時沒有的數據,因為它無法看到未來。 目標洩漏是一個容易錯過的錯誤,但通常的特點是異常高的準確性為您的問題。 如果您試圖預測股票價格,並訓練模型在 95% 的準確性,很可能有目標洩漏在您的功能某處。

刪除功能還可以防止模型有太多的欄位用於記憶特定模式,從而使其更加靈活,從而有助於過度擬合。 定量測量可能很困難,但如果可以移除特徵並保持相同的精度,則可能會使模型更加靈活,並降低了過度擬合的風險。

### <a name="best-practices-automated-ml-implements"></a>最佳實務自動化 ML 機具

正規化是最小化成本函數以懲罰複雜和過度裝配的模型的過程。 有不同類型的正則化函數,但一般來說,它們都懲罰模型係數大小、方差和複雜性。 自動 ML 使用不同的組合使用 L1(拉索)、L2(裡奇)和 ElasticNet(同時 L1 和 L2),並使用不同的模型超參數設置來控制過度擬合。 簡單地說,自動 ML 將改變模型的調節程度,並選擇最佳結果。

自動 ML 還實現了顯式模型複雜性限制,以防止過度擬合。 在大多數情況下,此實現專門用於決策樹或林演演演算法,其中單個樹的最大深度是有限的,並且林或組合技術中使用的樹總數是有限的。

交叉驗證 (CV) 是獲取完整訓練數據的許多子集並訓練每個子集上的模型的過程。 其理念是,模型可以獲得"幸運",並且具有一個子集的極高精度,但通過使用許多子集,模型不會每次都達到如此高的準確性。 執行 CV 時,提供驗證保留數據集、指定 CV 摺疊(子集數)和自動 ML 將訓練模型並調整超參數,以盡量減少驗證集上的錯誤。 一個 CV 摺疊可能過度擬合,但通過使用其中的許多折頁,可以降低最終模型過度擬合的可能性。 權衡是 CV 確實會導致較長的培訓時間,從而增加成本,因為您不是訓練一次模型,而是為每個*n* CV 子集訓練一次。 

> [!NOTE]
> 默認情況下,不會啟用交叉驗證;因此,不會啟用交叉驗證。必須在自動 ML 設定中設定它。 但是,在配置交叉驗證並提供驗證數據集后,該過程將為您自動執行。 請參閱 

<a name="imbalance"></a>

## <a name="identify-models-with-imbalanced-data"></a>使用不平衡資料識別模型

不平衡數據常見於機器學習分類方案的數據中,並指包含每個類中觀測值比例不成比例的數據。 這種不平衡可能導致模型準確性的誤感知正效應,因為輸入數據對一個類有偏差,從而導致經過訓練的模型來類比這種偏差。 

由於分類演演演算法通常按精度進行評估,因此檢查模型的準確率是識別模型是否受不平衡數據影響的好方法。 對於某些類來說,它是否真的具有高精度或非常低的精度?

此外,自動 ML 執行會自動生成以下圖表,這有助於您瞭解模型分類的正確性,並確定可能受不平衡數據影響的模型。

圖表| 描述
---|---
[混淆矩陣](how-to-understand-automated-ml.md#confusion-matrix)| 根據數據的實際標籤評估正確分類的標籤。 
[精密召回](how-to-understand-automated-ml.md#precision-recall-chart)| 根據找到的資料標籤實體的比率評估正確標籤的比率 
[ROC 曲線](how-to-understand-automated-ml.md#roc)| 評估正確標籤與誤報標籤的比率的比率。

## <a name="handle-imbalanced-data"></a>處理不平衡的資料 

作為簡化機器學習工作流目標的一部分,自動化 ML 內置了幫助處理不平衡數據的能力,例如: 

- **權重列**:自動 ML 支援加權列作為輸入,導致數據中的行向上或向下加權,從而使類或多或少變得"重要"。

- 自動 ML 使用的演演演算法可以正確處理高達 20:1 的不平衡,這意味著最常見的類在數據中的行數可以比最不常見的類多 20 倍。

以下技術是處理自動 ML 外部不平衡數據的其他選項。 

- 重新採樣,甚至類不平衡,要麼向上採樣較小的類或向下採樣較大的類。 這些方法需要專業知識來處理和分析。

- 使用更好地處理不平衡數據的性能指標。 例如,F1 分數是精度和召回的加權平均值。 精確測量分類器的精確性-低精度表示大量誤報--,而召回測量分類器完整性時,低召回表示大量假陰性。 

## <a name="next-steps"></a>後續步驟

檢視範例並瞭解如何使用自動機器學習建構模型:

+ 遵循[教學:使用 Azure 機器學習自動訓練回歸模型](tutorial-auto-train-models.md)

+ 設定自動訓練實驗的設定:
  + 在 Azure 機器學習工作室中,[使用這些步驟](how-to-use-automated-ml-for-ml-models.md)。
  + 使用 Python SDK,[請使用這些步驟](how-to-configure-auto-train.md)。



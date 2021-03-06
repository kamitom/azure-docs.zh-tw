---
title: 交叉驗證模型：模組參考
titleSuffix: Azure Machine Learning
description: 瞭解如何在 Azure 機器學習中使用交叉驗證模型模組，通過分區資料來交叉驗證分類或回歸模型的參數估計值。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/11/2020
ms.openlocfilehash: 7550bb7c6bbf7602245f9a9f1ac006ce693b36a8
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79477641"
---
# <a name="cross-validate-model"></a>交叉驗證模型

本文介紹如何在 Azure 機器學習設計器（預覽）中使用交叉驗證模型模組。 *交叉驗證*是機器學習中常用的一種技術，用於評估資料集的可變性以及通過該資料訓練的任何模型的可靠性。  

交叉驗證模型模組將標記的資料集與未經訓練的分類或回歸模型一起作為輸入。 它將資料集劃分為若干個子集（*折疊*），在每個折疊上生成模型，然後為每個折疊返回一組精度統計資訊。 通過比較所有折疊的準確性統計資訊，可以解釋資料集的品質。 然後，您可以瞭解模型是否容易受到資料變化的影響。  

交叉驗證模型還會返回資料集的預測結果和概率，以便您可以評估預測的可靠性。  

### <a name="how-cross-validation-works"></a>交叉驗證的工作原理

1. 交叉驗證隨機將訓練資料分成折疊。 

   如果您先前未分割資料集，演算法的預設值為 10 個摺疊。 要將資料集劃分為不同數量的折疊，可以使用[分區和示例](partition-and-sample.md)模組並指示要使用的折數。  

2.  模組將第 1 頁中的資料放在一邊以進行驗證。 （這有時稱為*保持折疊*。模組使用剩餘的折疊來訓練模型。 

    例如，如果創建五個折疊，模組將在交叉驗證期間生成五個模型。 模組使用五分之四的資料對每個模型進行訓練。 它在剩下的五分之一上測試每個模型。  

3.  在測試每個折疊的模型期間，模組評估多個精度統計資訊。 模組使用的統計資訊取決於要評估的模型類型。 使用不同的統計資訊來評估分類模型與回歸模型。  

4.  當所有折疊的構建和評估過程完成時，交叉驗證模型將生成一組性能指標，並對所有資料評分結果。 查看這些指標，瞭解任何單折是否具有高精度或低精度。 

### <a name="advantages-of-cross-validation"></a>交叉驗證的優點

評估模型的不同和常見方法是使用[拆分資料](split-data.md)將資料劃分為訓練和測試集，然後在訓練資料上驗證模型。 但交叉驗證提供了一些優點：  

-   交叉驗證使用更多的測試資料。

    交叉驗證測量模型在更大的資料空間中具有指定參數的性能。 也就是說，交叉驗證使用整個訓練資料集進行培訓和評估，而不是部分。 相反，如果使用從隨機拆分生成的資料驗證模型，則通常僅根據 30% 或更少的可用資料來評估模型。  

    但是，由於交叉驗證在較大的資料集上多次訓練和驗證模型，因此計算密集型程度要高得多。 這比在隨機拆分上驗證要長得多。  

-   交叉驗證計算資料集和模型。

    交叉驗證不僅僅測量模型的準確性。 它還讓您瞭解資料集的代表性程度以及模型對資料變化的敏感性。  

## <a name="how-to-use-cross-validate-model"></a>如何使用交叉驗證模型

如果資料集較大，則交叉驗證可能需要很長時間才能運行。  因此，您可以在構建和測試模型的初始階段使用交叉驗證模型。 在這個階段，您可以計算模型參數的優劣（假設計算時間是可以容忍的）。 然後，您可以將已建立的參數與[訓練模型](train-model.md)和評估模型模組一起使用來訓練和評估[模型](evaluate-model.md)。

在這種情況下，您同時使用交叉驗證模型訓練和測試模型。

1. 將交叉驗證模型模組添加到管道中。 您可以在 Azure 機器學習設計器中，在 **"模型評分&評估**"類別中找到它。 

2. 連接任何分類或回歸模型的輸出。 

    例如，如果您使用**兩類提升決策樹**進行分類，請使用所需的參數配置模型。 然後，將連接器從分類器的**未訓練模型**埠拖動到交叉驗證模型的匹配埠。 

    > [!TIP] 
    > 您不必訓練模型，因為交叉驗證模型會自動訓練模型作為評估的一部分。  
3.  在交叉驗證模型**的資料集**埠上，連接任何標記的訓練資料集。  

4.  在交叉驗證模型的右側面板中，按一下 **"編輯"列**。 選擇包含類標籤或可預測值的單個列。 

5. 如果要在同一資料上重複跨連續運行的交叉驗證結果，請為**Random 種子**參數設置值。  

6. 提交管道。

7. 有關報表的說明，請參閱["結果](#results)"部分。

## <a name="results"></a>結果

完成所有反覆運算後，交叉驗證模型將創建整個資料集的分數。 它還創建性能指標，可用於評估模型的品質。

### <a name="scored-results"></a>計分的結果

模組的第一個輸出提供每行的來源資料，以及一些預測值和相關概率。 

要查看結果，在管道中，按右鍵交叉驗證模型模組。 選擇**視覺化評分結果**。

| 新增資料行名稱      | 描述                              |
| -------------------- | ---------------------------------------- |
| 評分標籤        | 此列在資料集的末尾添加。 它包含每行的預測值。 |
| 評分概率 | 此列在資料集的末尾添加。 它指示**評分標籤**中值的估計概率。 |
| 折疊號          | 指示在交叉驗證期間分配給每行資料的折疊的從零為基礎的索引。 |

 ### <a name="evaluation-results"></a>Evaluation results

第二個報表按折疊分組。 請記住，在執行期間，交叉驗證模型會隨機將訓練資料拆分為*n*折（預設情況下為 10）。 在資料集上的每次反覆運算中，交叉驗證模型使用一個折疊作為驗證資料集。 它使用剩餘的*n-1*折來訓練模型。 每個*n*模型都根據所有其他折疊中的資料進行測試。

在此報表中，折數按索引值按昇冪列出。  要對任何其他列進行排序，可以將結果另存為資料集。

要查看結果，在管道中，按右鍵交叉驗證模型模組。 選擇**按折疊視覺化評估結果**。


|資料行名稱| 描述|
|----|----|
|折疊號| 每個折疊的識別碼。 如果創建五個倍數，則有五個數據子集，編號為 0 到 4。
|折疊中的示例數|分配給每個折疊的行數。 它們應該大致相等。 |


該模組還包括每個折疊的以下指標，具體取決於要評估的模型類型： 

+ **分類模型**： 精度， 召回， F 得分， AUC， 精度  

+ **回歸模型**：平均絕對誤差、根平均平方誤差、相對絕對誤差、相對平方誤差和確定係數


## <a name="technical-notes"></a>技術說明  

+ 最佳做法是，在使用資料集進行交叉驗證之前，對資料集進行正常化。 

+ 與使用隨機分割資料集驗證模型相比，交叉驗證模型的計算密集程度要高得多，完成時間也更長。 原因是交叉驗證模型多次訓練和驗證模型。

+ 使用交叉驗證測量模型的準確性時，無需將資料集拆分為定型和測試集。 


## <a name="next-steps"></a>後續步驟

請參閱 Azure 機器學習[可用的模組集](module-reference.md)。 


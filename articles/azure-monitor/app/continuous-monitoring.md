---
title: 使用 Azure 管道和 Azure 應用程式見解持續監視 DevOps 發佈管道 |微軟文檔
description: 提供使用應用程式見解快速設置連續監視的說明
ms.topic: conceptual
ms.date: 07/16/2019
ms.openlocfilehash: e565101218b975ef2bd29b8a32a4aa1bf4300b6d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "77655390"
---
# <a name="add-continuous-monitoring-to-your-release-pipeline"></a>將連續監視新增至您的發行管線

Azure 管道與 Azure 應用程式見解集成，允許在整個軟體發展生命週期中持續監視 DevOps 發佈管道。 

通過持續監視，發佈管道可以合併來自應用程式見解和其他 Azure 資源的監視資料。 當發佈管道檢測到應用程式見解警報時，管道可以對部署進行門禁或回滾，直到警報得到解決。 如果所有檢查都通過，部署可以從測試一直自動進行到生產，而無需手動干預。 

## <a name="configure-continuous-monitoring"></a>設定連續監視

1. 在[Azure DevOps](https://dev.azure.com)中，選擇組織和專案。
   
1. 在專案頁面的左側功能表上，選擇 **"** > 管道**版本**"。 
   
1. 下拉 **"新建"** 旁邊的箭頭，然後選擇 **"新建"管道**。 或者，如果您還沒有管道，請選擇顯示的頁面上**的"新建"管道**。
   
1. 在"**選擇範本"** 窗格中，搜索並選擇**具有連續監視的 Azure 應用服務部署**，然後選擇"**應用**"。 

   ![新的 Azure 管道發佈管道](media/continuous-monitoring/001.png)

1. 在 **"階段 1"** 框中，選擇**指向"查看階段任務"的超連結。**

   ![檢視階段工作](media/continuous-monitoring/002.png)

1. 在**階段 1**配置窗格中，完成以下欄位： 

    | 參數        | 值 |
   | ------------- |:-----|
   | **階段名稱**      | 提供舞臺名稱，或將其保留在**階段 1**中。 |
   | **Azure 訂閱** | 下拉並選擇要使用的連結 Azure 訂閱。|
   | **應用類型** | 下拉並選擇你的應用類型。 |
   | **App Service 名稱** | 輸入 Azure 應用服務的名稱。 |
   | **應用程式見解的資源組名稱**    | 下拉並選擇要使用的資源組。 |
   | **Application Insights 資源名稱** | 下拉並選擇所選資源組的應用程式見解資源。

1. 要使用預設警報規則設置保存管道，請在 Azure DevOps 視窗中選擇右上角的 **"保存**"。 輸入描述性注釋，然後選擇 **"確定**"。

## <a name="modify-alert-rules"></a>修改警示規則

開箱即用的**Azure 應用服務部署具有連續監視**範本，有四個警報規則：**可用性**、**失敗請求**、**伺服器回應時間**和**伺服器異常**。 您可以添加更多規則，或更改規則設置以滿足服務等級需求。 

要修改警報規則設置：

1. 在發佈管道頁的左側窗格中，選擇 **"配置應用程式見解警報**"。

1. 在**Azure 監視器警報**窗格中，選擇 **"警報"規則****旁邊的省略**號。
   
1. 在 **"警報規則"** 對話方塊中，選擇警報規則旁邊的下拉符號，如**可用性**。 
   
1. 修改**閾值**和其他設置以滿足您的要求。
   
   ![修改警報](media/continuous-monitoring/003.png)
   
1. 選擇 **"確定**"，然後在 Azure DevOps 視窗中選擇右上角的 **"保存**"。 輸入描述性注釋，然後選擇 **"確定**"。

## <a name="add-deployment-conditions"></a>新增部署條件

將部署門添加到發佈管道時，超出您設置閾值的警報可防止意外發佈升級。 解決警報後，部署可以自動進行。

要添加部署門：

1. 在主管道頁面上，在 **"階段**"下，根據哪個階段需要連續監視門，選擇**部署前條件**或**部署後條件**符號。
   
   ![部署前條件](media/continuous-monitoring/004.png)
   
1. 在 **"部署前條件**配置"窗格中，將 **"蓋茨**"設置為**啟用**。
   
1. 在**部署門**旁邊，選擇 **"添加**"。
   
1. 從下拉式功能表中選擇 **"查詢 Azure 監視器監視警報**"。 此選項允許您同時訪問 Azure 監視器和應用程式見解警報。
   
   ![查詢 Azure 監視器警報](media/continuous-monitoring/005.png)
   
1. 在 **"評估"選項**下，輸入所需的設置值，如"**重新評估門和****門失敗之後的超時**之間的時間"。 

## <a name="view-release-logs"></a>查看發佈日誌

您可以在發佈日誌中看到部署門行為和其他發佈步驟。 要打開日誌：

1. 從管道頁面的左側功能表中選擇 **"釋放**"。 
   
1. 選擇任何版本。 
   
1. 在 **"階段"** 下，選擇任何階段以查看發佈摘要。 
   
1. 要查看日誌，請選擇發佈摘要中的 **"查看日誌**"，選擇任何階段的 **"成功**"或 **"失敗"** 超連結，或將滑鼠懸停在任何階段並選擇 **"日誌**"。 
   
   ![查看發佈日誌](media/continuous-monitoring/006.png)

## <a name="next-steps"></a>後續步驟

有關 Azure 管道的詳細資訊，請參閱[Azure 管道文檔](https://docs.microsoft.com/azure/devops/pipelines)。

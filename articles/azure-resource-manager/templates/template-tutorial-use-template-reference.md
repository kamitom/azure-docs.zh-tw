---
title: 利用範本參考
description: 利用 Azure Resource Manager 範本參考來建立範本。
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: b742982121a20a2b057eba4211584b0386dde411
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "80373174"
---
# <a name="tutorial-utilize-the-arm-template-reference"></a>教學課程：利用 ARM 範本參考

了解如何尋找範本結構描述資訊，以及使用此資訊來建立 Azure Resource Manager (ARM) 範本。

在本教學課程中，您會使用 Azure 快速入門範本中的基底範本。 您可使用範本參考文件來自訂範本。

![Resource Manager 範本參考部署儲存體帳戶](./media/template-tutorial-use-template-reference/resource-manager-template-tutorial-deploy-storage-account.png)

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 開啟快速入門範本
> * 了解範本
> * 尋找範本參考
> * 編輯範本
> * 部署範本

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>Prerequisites

若要完成本文，您需要：

* Visual Studio Code 搭配 Resource Manager Tools 擴充功能。 請參閱[使用 Visual Studio Code 建立 ARM 範本](use-vs-code-to-create-template.md)。

## <a name="open-a-quickstart-template"></a>開啟快速入門範本

[Azure 快速入門範本](https://azure.microsoft.com/resources/templates/)是 ARM 範本的存放庫。 您可以尋找範例範本並加以自訂，而不要從頭建立範本。 本快速入門中使用的範本名為[建立標準儲存體帳戶](https://azure.microsoft.com/resources/templates/101-storage-account-create/)。 此範本會定義 Azure 儲存體帳戶資源。

1. 在 Visual Studio Code 中，選取 [檔案]  >[開啟檔案]  。
1. 在 [檔案名稱]  中，貼上下列 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```

1. 選取 [開啟]  以開啟檔案。
1. 選取 [檔案]  >[另存新檔]  ，在您的本機電腦上將檔案另存為 **azuredeploy.json**。

## <a name="understand-the-schema"></a>了解結構描述

1. 在 VS Code 中，將範本摺疊至根層級。 此時會有最簡單的結構，內含下列元素：

    ![最簡單的 Resource Manager 範本結構](./media/template-tutorial-use-template-reference/resource-manager-template-simplest-structure.png)

    * **$schema**：指定說明範本語言版本的 JSON 結構描述檔案所在的位置。
    * **contentVersion**：為此元素指定任何值，用以記錄範本中的重大變更。
    * **parameters**：指定執行部署以自訂資源部署時所提供的值。
    * **variables**：指定範本中作為 JSON 片段以簡化範本語言運算式的值。
    * **resources**：指定在資源群組中部署或更新的資源類型。
    * **outputs**：指定部署後傳回的值。

1. 擴充**資源**。 已定義 `Microsoft.Storage/storageAccounts` 資源。

    ![Resource Manager 範本儲存體帳戶定義](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resource.png)

## <a name="find-the-template-reference"></a>尋找範本參考

1. 瀏覽至[Azure 範本參考](https://docs.microsoft.com/azure/templates/)。
1. 在 [依標題篩選]  方塊中，輸入**儲存體帳戶**，然後選取 [參考] > [儲存體] ****底下的第一個 [儲存體帳戶]**** 。

    ![Resource Manager 範本參考儲存體帳戶](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts.png)

    資源提供者通常有數個 API 版本：

    ![Resource Manager 範本參考儲存體帳戶版本](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-versions.png)

1. 在左窗格中，選取 [儲存體]  底下的 [所有資源]  。 此頁面會列出儲存體資源提供者的資源類型和版本。 建議您針對範本中定義的資源類型使用最新的 API 版本。

    ![Resource Manager 範本參考儲存體帳戶類型版本](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-types-versions.png)

1. 選取 **storageAccount** 資源類型的最新版本。  撰寫本文時的最新版本是 **2019-06-01**。

1. 此頁面會列出 storageAccount 資源類型的詳細資料。  例如，其會列出 **Sku** 物件的允許值。 SKU 數量比您稍早開啟的快速入門範本中所列的 SKU 還要多。 您可以自訂快速入門範本，以包含所有可用的儲存體類型。

    ![Resource Manager 範本參考儲存體帳戶 SKU](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-skus.png)

## <a name="edit-the-template"></a>編輯範本

在 Visual Studio Code 中，新增其他儲存體帳戶類型，如下列螢幕擷取畫面所示：

![Resource Manager 範本儲存體帳戶資源](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus.png)

## <a name="deploy-the-template"></a>部署範本

請參閱 Visual Studio Code 快速入門中的[部署範本](quickstart-create-templates-use-visual-studio-code.md#deploy-the-template)一節，以取得部署程序。 當您部署範本時，請使用新增的值來指定 **storageAccountType** 參數，例如 **Premium_ZRS**。 如果您使用原始快速入門範本，部署將會失敗，因為 **Premium_ZRS** 不是允許的值。

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可藉由刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]  。
2. 在 [依名稱篩選]  欄位中輸入資源群組名稱。
3. 選取資源群組名稱。  您在資源群組中應該會看到共計六個資源。
4. 從頂端功能表中選取 [刪除資源群組]  。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用範本參考來自訂現有的範本。 若要了解如何建立多個儲存體帳戶執行個體，請參閱：

> [!div class="nextstepaction"]
> [建立多個執行個體](./template-tutorial-create-multiple-instances.md)

---
title: 自訂 SQL 資訊保護 - Azure 安全中心
description: 了解如何在 Azure 資訊安全中心自訂資訊保護原則。
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 2ebf2bc7-232a-45c4-a06a-b3d32aaf2500
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/29/2019
ms.author: memildin
ms.openlocfilehash: 9c776a32b4a35c72fc40a16afb87db9896a763cf
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "75611061"
---
# <a name="customize-the-sql-information-protection-policy-in-azure-security-center-preview"></a>在 Azure 資訊安全中心自訂 SQL 資訊保護原則 (預覽)
 
您可以在 Azure 安全中心為整個 Azure 租戶定義和自訂 SQL 資訊保護原則。

資訊保護是一種高級安全功能，用於在 Azure 資料資源中發現、分類、標記和報告敏感性資料。 發現和分類最敏感的資料（業務、財務、醫療保健、個人資料等）可以在組織資訊保護方面發揮關鍵作用。 它可以作為下列的基礎結構：
- 協助符合資料隱私權標準和法規合規性需求
- 安全方案，如監視（審核）和對敏感性資料異常訪問的警報
- 對包含高度敏感性資料的資料存放區進行存取控制並強化安全性
 
[SQL Information Protection](../sql-database/sql-database-data-discovery-and-classification.md) 會針對 SQL 資料存放區實作此範例，而 Azure SQL Database 目前予以支援。 SQL Information Protection 會自動探索和分類可能的敏感性資料、提供標記機制以使用分類屬性持續標記敏感性資料，並提供詳細儀表板以顯示資料庫的分類狀態。 此外，它還會計算 SQL 查詢的結果集敏感度，以明確地稽核可擷取敏感性資料的查詢，並保護資料。 有關 SQL 資訊保護的詳細資訊，請參閱[Azure SQL 資料庫資料發現和分類](../sql-database/sql-database-data-discovery-and-classification.md)。
 
分類機制根據構成分類法的兩個主要建構：**標籤**和**資訊類型**。
- **標籤**– 用於定義存儲在列中的資料的敏感度等級的主要分類屬性。 
- **資訊類型** – 為儲存在資料行中的資料類型提供額外的細微性。
 
Information Protection 隨附一組預設會使用的內建標籤和資訊類型。 要自訂這些標籤和類型，可以在安全中心自訂資訊保護原則。
 
## <a name="customize-the-information-protection-policy"></a>自訂資訊保護原則
若要自訂 Azure 租用戶的資訊保護原則，您需要具有[租用戶根管理群組的管理權限](security-center-management-groups.md)。 
 
1. 在安全中心主功能表中，在 **"資源安全"** 下轉到**資料存儲&存儲**，然後按一下**SQL 資訊保護**按鈕。

   ![設定資訊保護原則](./media/security-center-info-protection-policy/security-policy.png) 
 
2. 在**SQL 資訊保護**頁中，您可以查看當前標籤集。 這些主要分類屬性用來分類資料的敏感度等級。 在這裡，您可以設定租用戶的 [Information protection labels] \(資訊保護標籤\)**** 和 [資訊類型]****。 
 
### <a name="customizing-labels"></a>自訂標籤
 
1. 您可以編輯或刪除任何現有標籤，或新增標籤。 若要編輯現有標籤，請選取該標籤，然後按一下頂端或右側操作功能表中的 [設定]****。 若要新增標籤，請按一下頂端功能表列或標籤資料表底部的 [建立標籤]****。
2. 在 [設定敏感度標籤]**** 畫面中，您可以建立或變更標籤名稱和描述。 您也可以開啟或關閉 [已啟用]**** 切換，以設定標籤為使用中還是已停用。 最後，您可以新增或移除與標籤建立關聯的資訊類型。 任何探索到且符合該資訊類型的資料都會在分類建議中自動包含相關聯的敏感度標籤。
3. 按一下 [確定]****。
 
   ![設定敏感度標籤](./media/security-center-info-protection-policy/config-sensitivity-label.png)
 
4. 標籤會依遞增敏感度列出。 若要變更標籤之間的排名，請拖曳標籤以重新排列它們在資料表中的順序，或使用 [Move up] \(上移\)**** 和 [Move down] \(下移\)**** 按鈕變更順序。 
 
    ![設定資訊保護原則](./media/security-center-info-protection-policy/move-up.png)
 
5. 請務必在您完成時按一下畫面頂端的 [儲存]****。
 
 
## <a name="adding-and-customizing-information-types"></a>新增和自訂資訊類型
 
1. 您可以按一下 [管理資訊類型]****，以管理和自訂資訊類型。
2. 若要新增 [資訊類型]****，請選取上層功能表中的 [建立資訊類型]****。 您可以設定 [資訊類型]**** 的名稱、描述和搜尋模式字串。 搜尋模式字串可以選擇性地使用具有萬用字元 (使用 '%' 字元) 的關鍵字，而自動化探索引擎會使用它，以根據資料行的中繼資料來識別您資料庫中的敏感性資料。
 
    ![設定資訊保護原則](./media/security-center-info-protection-policy/info-types.png)
 
3. 您也可以新增其他搜尋模式字串、停用一些現有字串，或變更描述，以設定內建 [資訊類型]****。 您無法刪除內建 [資訊類型]**** 或編輯其名稱。 
4. **資訊類型**會以遞增探索排名順序列出，這表示會先嘗試比對清單中較高的類型。 若要變更資訊類型之間的排名，請將類型拖曳至資料表中的正確位置，或使用 [Move up] \(上移\)**** 和 [Move down] \(下移\)**** 按鈕變更順序。 
5. 完成時，請按一下 [確定]****。
6. 在您完成資訊類型的管理之後，請務必按一下特定標籤的 [設定]****，並視需要新增或刪除資訊類型，以建立相關類型與相關標籤的關聯。
7. 請務必按一下主要 [標籤]**** 刀鋒視窗中的 [儲存]****，以套用所有變更。
 
完整定義和儲存資訊保護原則之後，就會套用至租用戶中所有 Azure SQL 資料庫上的資料分類。
 
 
## <a name="next-steps"></a>後續步驟
 
在本文中，您已了解在 Azure 資訊安全中心定義 SQL Information Protection 原則。 若要深入了解如何使用 SQL Information Protection 分類和保護 SQL 資料庫中的敏感性資料，請參閱 [Azure SQL Database 的資料探索與分類](../sql-database/sql-database-data-discovery-and-classification.md)。 

如需 Azure 資訊安全中心內安全性原則和資料安全性的詳細資訊，請參閱下列文章：
 
- [在 Azure 安全中心設置安全性原則](tutorial-security-policy.md)：瞭解如何為 Azure 訂閱和資源組配置安全性原則
- [Azure 安全中心資料安全](security-center-data-security.md)：瞭解安全中心如何管理和保護資料

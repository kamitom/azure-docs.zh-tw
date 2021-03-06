---
title: 在 Azure Active Directory B2C 中管理使用者資料 | Microsoft Docs
description: 了解如何在 Azure AD B2C 中刪除或匯出使用者資料。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/06/2018
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: e245b58449ab773914fc60be056082b82f05035a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "78184480"
---
# <a name="manage-user-data-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中管理使用者資料

 本文討論如何通過使用[Microsoft 圖形 API](https://docs.microsoft.com/graph/use-the-api)提供的操作來管理 Azure 活動目錄 B2C （Azure AD B2C） 中的使用者資料。 管理使用者資料，包括從稽核記錄中刪除或匯出資料。

[!INCLUDE [gdpr-intro-sentence.md](../../includes/gdpr-intro-sentence.md)]

## <a name="delete-user-data"></a>刪除使用者資料

使用者資料會儲存在 Azure AD B2C 目錄和稽核記錄中。 所有使用者審核資料在 Azure AD B2C 中保留 7 天。 如果要在這 7 天內刪除使用者資料，可以使用["刪除使用者](https://docs.microsoft.com/graph/api/user-delete)操作"。 針對資料所在的每個 Azure AD B2C 租用戶，必須個別進行刪除作業。

在 Azure AD B2C 中，會為每個使用者指派一個物件識別碼。 物件識別碼可作為明確的識別碼，供您在 Azure AD B2C 中用來刪除使用者資料。 依架構而定，物件識別碼可能是跨越財務、行銷與客戶關係管理資料庫等其他服務的實用相互關聯識別碼。

在 Azure AD B2C 驗證過程中取得使用者物件識別碼，是最正確的方法。 若使用其他方法接收來自使用者的有效資料要求，則可能需要離線流程 (像是客服專員所做的搜尋) 來找出使用者並記下關聯的物件識別碼。

下列範例示範可能的資料刪除流程：

1. 使用者登入並選取 [刪除我的資料]****。
2. 應用程式在應用程式系統管理區段內，提供刪除資料的選項。
3. 應用程式強制向 Azure AD B2C 驗證。 Azure AD B2C 將包含使用者物件識別碼的權杖提供給應用程式。
4. 權杖由應用程式接收，物件識別碼 用於通過調用 Microsoft 圖形 API 刪除使用者資料。 Microsoft 圖形 API 刪除使用者資料並返回狀態碼 200 OK。
5. 應用程式使用物件識別碼或其他識別碼，依需求協調其他組織系統中的使用者資料刪除作業。
6. 應用程式確認資料刪除，並提供後續步驟給使用者。

## <a name="export-customer-data"></a>匯出客戶資料

從 Azure AD B2C 匯出客戶資料的流程類似於刪除流程。

Azure AD B2C 使用者資料僅限於：

- **儲存在 Azure Active Directory 中的資料**：在 Azure AD B2C 驗證使用者過程中，可以使用物件識別碼，或是電子郵件地址或使用者名稱等登入名稱來擷取資料。
- **使用者特定的稽核事件報告**：您可以使用物件識別碼為資料編製索引。

在下列匯出資料流程的範例中，所述由應用程式執行的步驟，亦可由後端處理序來執行，或由具有目錄中系統管理員角色的使用者來執行：

1. 使用者登入應用程式。 Azure AD B2C 會依需求強制進行 Azure Multi-Factor Authentication 的驗證。
2. 應用程式使用使用者憑據調用 Microsoft 圖形 API 操作來檢索使用者屬性。 Microsoft 圖形 API 以 JSON 格式提供屬性資料。 根據結構描述而定，您可以設定識別碼權杖內容，以包含使用者的所有個人資料。
3. 應用程式會擷取使用者稽核活動。 Microsoft 圖形 API 向應用程式提供事件資料。
4. 應用程式彙總資料，並提供給使用者使用。

## <a name="next-steps"></a>後續步驟

若要了解如何管理使用者存取應用程式的方式，請參閱[管理使用者存取權](manage-user-access.md)。

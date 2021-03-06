---
title: 授權資料作業
titleSuffix: Azure Storage
description: 瞭解授權存取 Azure 儲存的不同方法,包括 Azure 活動目錄、共用密鑰授權或共享訪問簽名 (SAS)。
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 02/24/2020
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 53eca8a0b9e7cc9abb8f89cd56fca5df28f2de0f
ms.sourcegitcommit: b0ff9c9d760a0426fd1226b909ab943e13ade330
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/01/2020
ms.locfileid: "80521934"
---
# <a name="authorizing-access-to-data-in-azure-storage"></a>授權存取 Azure 儲存中的資料

每次存取儲存體帳戶中的資料時，用戶端都會透過 HTTP/HTTPS 對 Azure 儲存體發出要求。 針對受保護資源所發出的每個要求都必須經過授權，服務才能確保用戶端具有所需的資料存取權限。

下表描述了 Azure 儲存為授權存取資源提供的選項:

|  |分享金鑰(儲存帳戶金鑰)  |共用存取簽章 (SAS)  |Azure Active Directory (Azure AD)  |動作目錄(預覽版) |匿名公共讀取存取  |
|---------|---------|---------|---------|---------|---------|
|Azure Blob     |[支援](/rest/api/storageservices/authorize-with-shared-key/)         |[支援](storage-sas-overview.md)         |[支援](storage-auth-aad.md)         |不支援|[支援](../blobs/storage-manage-access-to-resources.md)         |
|Azure 檔案 (SMB)     |[支援](/rest/api/storageservices/authorize-with-shared-key/)         |不支援         |[支援,僅適用於 AAD 網域服務](../files/storage-files-active-directory-overview.md)         |[受支援,認證必須同步到 Azure AD](../files/storage-files-active-directory-overview.md)|不支援         |
|Azure 檔案 (REST)     |[支援](/rest/api/storageservices/authorize-with-shared-key/)         |[支援](storage-sas-overview.md)         |不支援         |不支援 |不支援         |
|Azure 佇列     |[支援](/rest/api/storageservices/authorize-with-shared-key/)         |[支援](storage-sas-overview.md)         |[支援](storage-auth-aad.md)         |不支援 | 不支援         |
|Azure 資料表     |[支援](/rest/api/storageservices/authorize-with-shared-key/)         |[支援](storage-sas-overview.md)         |不支援         |不支援| 不支援         |

下面將簡要描述每個授權選項:

- Blob 與佇列的**Azure 的活動目錄 (Azure AD) 整合**。 Azure AD 提供基於角色的存取控制 (RBAC),用於控制用戶端對存儲帳戶中資源的訪問。 有關 Blob 和佇列的 Azure AD 整合的詳細資訊,請參考[Azure 的動作目錄 授權存取 Azure blob 與佇列](storage-auth-aad.md)。

- **Azure 活動目錄域服務 (Azure AD DS)** 對 Azure 檔案的身份驗證。 Azure 檔案支援透過 Azure AD DS 透過伺服器訊息區塊 (SMB) 進行基於識別的授權。 可以使用 RBAC 對用戶端對儲存帳戶中的 Azure 檔案資源的訪問許可權進行細粒度控制。 有關使用網域服務的 Azure 檔身分驗證的詳細資訊,請參閱我們的[概述](../files/storage-files-active-directory-overview.md)。

- **Azure 檔案的活動目錄 (AD) 身份驗證(預覽)。** Azure 檔案透過 AD 支援透過 SMB 進行基於身份的授權。 AD 網域服務可以託管在本地電腦或 Azure VM 中。 SMB 存取檔受支援,使用來自域聯接的電腦的 AD 認證,無論是本地還是在 Azure 中。 您可以將 RBAC 用於共用等級的存取控制,將 NTFS DACL 用於目錄/檔案級許可權實施。 有關使用網域服務的 Azure 檔身分驗證的詳細資訊,請參閱我們的[概述](../files/storage-files-active-directory-overview.md)。

- **共用金鑰授權**，適用於 Blob、檔案、佇列和資料表。 使用共用金鑰的用戶端會在每個要求中，傳遞使用儲存體帳戶存取金鑰所簽署的標頭。 如需詳細資訊，請參閱[使用共用金鑰進行授權](/rest/api/storageservices/authorize-with-shared-key/)。
- **共用存取簽章**，適用於 Blob、檔案、佇列和資料表。 共用存取簽章 (SAS) 可提供儲存體帳戶資源的有限委派存取權。 對簽章的有效時間間隔或對其授與的權限新增條件約束，可讓您彈性管理存取權。 如需詳細資訊，請參閱[使用共用存取簽章 (SAS)](storage-sas-overview.md)。
- **匿名公開讀取存取**，適用於容器和 Blob。 不需要授權。 如需詳細資訊，請參閱 [管理對容器與 Blob 的匿名讀取權限](../blobs/storage-manage-access-to-resources.md)。  

根據預設，Azure 儲存體中的所有資源都會受到保護，並只供帳戶擁有者使用。 雖然您可以使用上述任何授權策略來對用戶端授與儲存體帳戶資源的存取權，但 Microsoft 建議您盡可能使用 Azure AD，以獲得最大安全性並方便您使用。

## <a name="next-steps"></a>後續步驟

- [使用 Azure 活動目錄授權存取 Azure Blob 和佇列](storage-auth-aad.md)
- [使用共用金鑰進行授權](/rest/api/storageservices/authorize-with-shared-key/)
- [使用分享存取簽章 (SAS) 給 Azure 資源的有限存取權限](storage-sas-overview.md)

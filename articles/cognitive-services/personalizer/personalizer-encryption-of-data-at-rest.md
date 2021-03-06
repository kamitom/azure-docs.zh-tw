---
title: 靜態資料的個人化服務加密
titleSuffix: Azure Cognitive Services
description: 靜態資料的個人化服務加密。
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: egeaney
ms.openlocfilehash: 63526454bb366b214c27bddce24ed9ebc09556b3
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80071223"
---
# <a name="personalizer-service-encryption-of-data-at-rest"></a>靜態資料的個人化服務加密

當資料保存到雲中時，個人化服務會自動對資料進行加密。 個人化服務加密可保護您的資料，並説明您履行組織安全和合規性承諾。

[!INCLUDE [cognitive-services-about-encryption](../../../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> 客戶管理的金鑰僅在 E0 定價層上可用。 要請求使用客戶管理的金鑰，請填寫並提交[個人化服務客戶管理金鑰請求表](https://aka.ms/cogsvc-cmk)。 大約需要 3-5 個工作日才能回復您的請求狀態。 根據需求，您可能會被放置在佇列中，並在可用空間時獲得批准。 一旦批准將 CMK 與個人化服務一起使用，您將需要創建新的個人化程式資源，並選擇 E0 作為定價層。 創建具有 E0 定價層的個人化資源後，可以使用 Azure 金鑰保存庫來設置託管標識。

[!INCLUDE [cognitive-services-cmk](../../../includes/cognitive-services-cmk-regions.md)]

[!INCLUDE [cognitive-services-cmk](../../../includes/cognitive-services-cmk.md)]

## <a name="next-steps"></a>後續步驟

* [個人化服務客戶管理的關鍵請求表](https://aka.ms/cogsvc-cmk)
* [瞭解有關 Azure 金鑰保存庫的更多](https://docs.microsoft.com/azure/key-vault/key-vault-overview)

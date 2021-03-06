---
title: 使用 Azure 事件中心中的共用訪問簽名授權訪問
description: 本文提供有關使用共用訪問簽名 （SAS） 授權訪問 Azure 事件中心資源的資訊。
services: event-hubs
ms.service: event-hubs
documentationcenter: ''
author: spelluru
ms.topic: conceptual
ms.date: 08/22/2019
ms.author: spelluru
ms.openlocfilehash: bdb1896f8a40c6de21ae76b536bfccec316341cd
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "69992792"
---
# <a name="authorizing-access-to-event-hubs-resources-using-shared-access-signatures"></a>使用共用訪問簽名授權訪問事件中心資源
共用訪問簽名 （SAS） 為您提供了授予對事件中心命名空間中的資源的有限存取權限的方法。 SAS 根據授權規則保護對事件中心資源的訪問。 這些規則在命名空間或實體（事件中心或主題）上配置。 本文概述了 SAS 模型，並回顧了 SAS 最佳實踐。

## <a name="what-are-shared-access-signatures"></a>什麼是共用訪問簽名？
共用訪問簽名 （SAS） 根據授權規則提供對事件中心資源的委派訪問。 授權規則具有名稱、與特定權限相關聯，並且承載了密碼編譯金鑰組。 您可以通過事件中心用戶端或您自己的代碼使用規則的名稱和金鑰來生成 SAS 權杖。 然後，用戶端可以將權杖傳遞給事件中心，以證明請求的操作的授權。

SAS 是一種基於聲明的授權機制，使用簡單的權杖。 使用 SAS 時，金鑰一律不會透過網路傳遞。 金鑰可用來以密碼編譯方式簽署稍後可由服務驗證的資訊。 SAS 的作用可類似於用戶端直接擁有授權規則名稱和相符金鑰的使用者名稱和密碼。 SAS 可以類似于聯合安全模型，其中用戶端從安全權杖服務接收有時間限制且已簽名的訪問權杖，而無需獲得簽名金鑰。

> [!NOTE]
> Azure 事件中心支援使用 Azure 活動目錄 （Azure AD） 授權到事件中心資源。 使用 Azure AD 返回的 OAuth 2.0 權杖授權使用者或應用程式比共用訪問簽名 （SAS） 更安全且便於使用。 使用 Azure AD，無需將權杖存儲在代碼中，並存在潛在的安全性漏洞風險。
>
> Microsoft 建議盡可能將 Azure AD 與 Azure 事件中心應用程式一起使用。 有關詳細資訊，請參閱使用[Azure 活動目錄 授權訪問 Azure 事件中心資源](authorize-access-azure-active-directory.md)。

> [!IMPORTANT]
> SAS（共用訪問簽名）權杖對於保護您的資源至關重要。 在提供細微性的同時，SAS 授予用戶端對事件中心資源的存取權限。 它們不應公開共用。 共用時，如果出於故障排除原因需要，請考慮使用任何日誌檔的還原版本或從日誌檔中刪除 SAS 權杖（如果存在），並確保螢幕截圖也不包含 SAS 資訊。

## <a name="shared-access-authorization-policies"></a>共用訪問授權策略
每個事件中心命名空間和每個事件中心實體（事件中心實例或 Kafka 主題）都有一個由規則組成的共用訪問授權策略。 命名空間層級的原則會套用至命名空間內的所有實體，無論其個別的原則組態為何。
對於每個授權原則規則，您會決定三項資訊：名稱、範圍和權限。 該名稱是該作用域中的唯一名稱。 範圍是相關資源的 URI。 對於事件中心命名空間，作用域是完全限定的功能變數名稱 （FQDN），如`https://<yournamespace>.servicebus.windows.net/`。

策略規則提供的許可權可以是：
- **發送**+ 授予向實體發送消息的權利
- **傾聽**– 授予收聽或接收實體的權利
- **管理**– 授予管理命名空間拓撲的權利，包括創建和刪除實體

> [!NOTE]
> **管理**許可權包括 **"發送**和**偵聽"** 許可權。

命名空間或實體策略最多可以容納 12 個共用訪問授權規則，為三組規則提供空間，每個規則都涵蓋基本權利，以及"發送"和"偵聽"的組合。 此限制強調 SAS 策略存儲不是使用者或服務帳戶存儲。 如果應用程式需要根據使用者或服務標識授予對事件中心資源的訪問，則應實現在身份驗證和訪問檢查後發出 SAS 權杖的安全權杖服務。

為授權規則分配了**主鍵**和**輔助鍵**。 這些金鑰是加密強金鑰。 不要丟失或洩漏它們。 它們始終在 Azure 門戶中可用。 您可以使用其中一個產生的金鑰，以及您可以隨時重新產生它們。 如果您重新產生或變更原則中的金鑰，所有先前根據該金鑰發行的權杖都將立即無效。 不過，根據這類權杖建立的作用中連線會繼續運作，直到權杖到期。

創建事件中心命名空間時，將自動為命名空間創建名為**RootManageSharedAccessKey**的策略規則。 此策略具有**整個**命名空間的管理許可權。 建議您將此規則視為管理根帳號，不要在應用程式中使用它。 您可以通過 PowerShell 或 Azure CLI 在門戶中的命名空間的 **"配置**"選項卡中創建其他策略規則。

## <a name="best-practices-when-using-sas"></a>使用 SAS 時的最佳做法
當您在應用程式中使用共用存取簽章時，您必須留意兩個潛在風險：

- 如果 SAS 洩露，則獲取 SAS 的任何人都可以使用它，這可能會危及事件中心資源。
- 如果提供給用戶端應用程式的 SAS 過期，並且應用程式無法從服務中檢索新的 SAS，則應用程式的功能可能會受阻。

下列關於使用共用存取簽章的建議，將可協助您平衡這些風險：

- **如有必要，讓用戶端自動續訂 SAS：** 用戶端應在過期之前續訂 SAS，以便在提供 SAS 的服務無法接通留出重試時間。 如果您的 SAS 用於少量預計在過期期限內完成的即時、短期操作，則可能沒有必要，因為 SAS 預計不會續訂。 不過，如果您有定期透過 SAS 做出要求的用戶端，則到期的可能性便有可能發生。 關鍵考慮是平衡 SAS 短期需求（如前所述）與確保客戶儘早請求續訂的需要（以避免因 SAS 在成功續訂之前到期而中斷）。
- **小心 SAS 開始時間**：如果將 SAS 的開始時間設置為**現在**，則由於時鐘偏斜（根據不同的機器在目前時間的差異），故障可能會間歇性地在最初幾分鐘內觀察到。 一般而言，請將開始時間設為至少 15 分鐘之前的時間。 或者，根本不設置它，這將使它在所有情況下立即有效。 這通常也適用于到期時間。 請記住，您可以在任何請求上觀察任意方向的時鐘偏差長達 15 分鐘。 
- **具體到要訪問的資源**：安全最佳實踐是為使用者提供所需的最低許可權。 如果使用者只需要單一實體的讀取存取權，則授與他們該單一實體的讀取存取權，而非授與他們所有實體的讀取/寫入/刪除存取權。 如果 SAS 受到攻擊，它還有助於減少損壞，因為 SAS 在攻擊者手中的功率較低。
- **不要總是使用 SAS：** 有時與針對事件中心的特定操作相關的風險大於 SAS 的好處。 對於此類操作，請創建一個中介層服務，在商務規則驗證、身份驗證和審核後寫入事件中心。
- **始終使用 HTTP**：始終使用 Https 創建或分發 SAS。 如果 SAS 通過 HTTP 傳遞並截獲，則執行中間人附加的攻擊者能夠讀取 SAS，然後像預期使用者一樣使用它，從而可能洩露敏感性資料或允許惡意使用者損壞資料。

## <a name="conclusion"></a>結論
共用訪問簽名可用於向用戶端提供事件中心資源的有限許可權。 對於使用 Azure 事件中心的任何應用程式，它們是安全模型的重要組成部分。 如果您遵循本文中列出的最佳實踐，則可以使用 SAS 提供更大的靈活性訪問資源，而不會損害應用程式的安全性。

## <a name="next-steps"></a>後續步驟
請參閱以下相關文章： 

- [使用 Azure 活動目錄對應用程式對 Azure 事件中心的請求進行身份驗證](authenticate-application.md)
- [使用 Azure 活動目錄對託管標識進行身份驗證以訪問事件中心資源](authenticate-managed-identity.md)
- [使用共用訪問簽名對 Azure 事件中心的請求進行身份驗證](authenticate-shared-access-signature.md)
- [使用 Azure 活動目錄授權訪問事件中心資源](authorize-access-azure-active-directory.md)



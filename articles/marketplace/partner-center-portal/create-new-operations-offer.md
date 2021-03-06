---
title: 為商業市場的運營產品創建新的動態 365
description: 如何使用 Microsoft 合作夥伴中心的商業應用商店門戶在 Azure 應用商店、AppSource 或通過雲解決方案供應商 （CSP） 計畫，為運營設置創建新的 Dynamics 365 產品/服務進行上市或銷售。
author: dsindona
ms.author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: f7c4b25dad50b8fe620d358aa7fb3e9decabc2f9
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80294344"
---
# <a name="create-a-new-dynamics-365-for-operations-offer"></a>為運營產品創建新的 Dynamics 365

本主題介紹如何為運營創建新的 Dynamics 365 產品。 [Microsoft 財務和運營動態 365](https://dynamics.microsoft.com/finance-and-operations)是一項企業資源規劃 （ERP） 服務，支援高級財務、運營、製造和供應鏈管理。 Dynamics 365 運營的所有優惠都必須經過我們的認證流程。

要開始為運營產品創建 Dynamics 365，請確保首先[創建合作夥伴中心帳戶](./create-account.md)並打開[商業市場儀表板](https://partner.microsoft.com/dashboard/commercial-marketplace/offers)，並選擇 **"概述"** 頁。

![合作夥伴中心上的商業市場儀表板](./media/new-offer-overview.png)

>[!Note]
> 產品/服務發佈後，只有在重新發佈後，才會在系統和商店前面更新對合作夥伴中心中提供的優惠進行編輯。 請確保在進行更改後提交要發佈的產品/服務。


## <a name="create-a-new-offer"></a>建立新的供應項目

選擇 **"+ 新產品/服務**"按鈕，然後為"操作"功能表項目選擇**Dynamics 365。** 將顯示"**新建產品/服務**"對話方塊。

### <a name="offer-id-and-alias"></a>優惠 ID 和別名

- **優惠 ID**：帳戶中每個產品/服務的唯一識別碼。 此 ID 將在市場產品/服務的 URL 位址和 Azure 資源管理器範本（如果適用）中向客戶可見。 優惠 ID 必須是小寫字母數位字元（包括連字號和底線，但沒有空格）。 此 ID 限制為 50 個字元，在選擇"**創建**"後無法更改。  例如，如果您在此處輸入*測試報價 1，* 則產品/服務 URL 將為`https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`。

- **優惠別名**：用於指合作夥伴中心內產品/服務的名稱。 此名稱不會在市場中使用，並且與向客戶顯示的要約名稱和其他值不同。 選擇 "**創建**"後，無法更改此值。

輸入**優惠 ID**和**優惠別名**後，選擇 **"創建**"。 然後，您將能夠處理產品/服務的所有不同部分。

## <a name="offer-setup"></a>優惠設置

**"產品/服務設置"** 頁要求提供以下資訊。 請務必在完成這些欄位後選擇 **"保存**"。

### <a name="how-do-you-want-potential-customers-to-interact-with-this-listing-offer"></a>您希望潛在客戶如何與此上市產品進行交互？

選擇要用於此優惠的選項。

#### <a name="get-it-now-free"></a>立即獲取（免費）

通過提供有效的 URL（從*HTTP*或*HTTPs*開頭），客戶可以訪問你的應用，從而免費向客戶列出您的優惠。  例如： `https://contoso.com/my-app`

#### <a name="free-trial-listing"></a>免費試用（清單）

通過提供有效的 URL（從*HTTP*或*HTTPs*開頭），以便客戶獲得試用版，從而向客戶列出您的優惠。"  例如：`https://contoso.com/trial/my-app`。 免費試用版產品清單的優惠由您的服務創建、管理和配置，並且沒有由 Microsoft 管理的訂閱。

> [!NOTE]
> 應用程式將通過試用連結收到的權杖只能用於通過 Azure 活動目錄 （Azure AD） 獲取使用者資訊，以自動在應用中創建帳戶。 使用此權杖進行身份驗證不支援 Microsoft 帳戶。

#### <a name="contact-me"></a>與我連絡

通過連接客戶關係管理 （CRM） 系統收集客戶聯繫資訊。 將請求客戶共用其資訊的許可權。 這些客戶詳細資訊以及找到產品/服務的產品/服務的產品/地區的產品/服務名稱、ID 和市場源將發送到您配置的 CRM 系統。 有關配置 CRM 的詳細資訊，請參閱[連接潛在客戶管理](#connect-lead-management)。 

### <a name="test-drive"></a>試用產品

試駕是向潛在客戶展示您的報價的絕佳方式，為他們提供"購買前試用"選項，從而提高轉化率並生成高素質潛在客戶。 [瞭解有關測試磁碟機的更多詳細資訊。](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/what-is-test-drive)

要啟用試駕，請選中 **"啟用試駕"** 框。 然後，您需要在[測試磁碟機技術配置](#test-drive-technical-configuration)配置中配置演示環境，以便客戶在固定時間段內試用您的產品/服務。

#### <a name="type-of-test-drive"></a>試駕類型

然後選取下列選項：

- **[Azure 資源管理器](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/azure-resource-manager-test-drive)**：包含構成解決方案的所有 Azure 資源的部署範本。 適合此方案的產品僅使用 Azure 資源。
- **[面向業務中心的動態 365：Microsoft](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cpp-business-central-offer)** 託管和維護業務中央企業資源規劃系統（財務、運營、供應鏈、CRM 等）的試駕服務（包括配置和部署）。  
- **[面向客戶參與的動態 365：Microsoft](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/dyn365ce/cpp-customer-engagement-offer)** 託管和維護客戶參與系統的試駕服務（包括配置和部署）（銷售、服務、專案服務、現場服務等）。  
- **[操作動態 365：](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cpp-dynamics-365-operations-offer)** Microsoft 託管和維護財務和運營企業資源規劃系統（財務、運營、製造、供應鏈等）的試駕服務（包括配置和部署）。 
- **[邏輯應用](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/logic-app-test-drive)**：包含所有複雜解決方案體系結構的部署範本。 任何自訂產品都應使用這種類型的試駕。
- **[Power BI](https://docs.microsoft.com/power-bi/service-template-apps-overview)**：指向自訂儀表板的嵌入式連結。 想要演示互動式 Power BI 視覺物件的產品應使用這種類型的測試磁碟機。 您只需要上傳內嵌的 Power BI URL。

#### <a name="additional-test-drive-resources"></a>其他試駕資源

- [試駕技術最佳實踐](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [試駕行銷最佳實踐](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/marketing-and-best-practices)
- [試駕概述一個呼叫器](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf)

## <a name="connect-lead-management"></a>連接潛在客戶管理

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

有關詳細資訊，請參閱[潛在客戶管理概述](./commercial-marketplace-get-customer-leads.md)。

在繼續下一節之前，請記住**保存**。

## <a name="properties"></a>屬性

屬性**頁面**允許您定義用於在市場對產品/服務進行分組的類別和行業、應用版本和支援產品/服務的法律合同。 完成此頁面後選擇 **"保存**"。

### <a name="category"></a>類別

選擇至少一個，最多選擇三個類別。 這些類別將用於將您的產品/服務放入相應的市場搜索區域。 請務必在產品/服務說明中標注您的產品/服務如何支援這些類別。

### <a name="industry"></a>業界

[!INCLUDE [Industry Taxonomy](./includes/industry-taxonomy.md)]

### <a name="app-version"></a>應用程式版本

輸入產品/服務的版本號。 客戶將在產品/服務的詳細資訊頁面上看到此版本。

### <a name="terms-and-conditions"></a>條款和條件

在條款及條件欄位中提供您自己的法律**條款及條件**。 您還可以提供可在其中找到您的條款及條件的 URL。 客戶在試用您的優惠之前，必須接受這些條款。

## <a name="offer-listing"></a>優惠清單

"產品/服務"清單頁顯示您的優惠將在其中列出的語言。 目前，**英語（美國）** 是唯一可用的選項。

您需要為每個語言/市場定義市場詳細資訊（報價名稱、說明、圖像等）。 選擇語言/市場名稱以提供此資訊。

> [!NOTE]
> 報價清單內容（如說明、文檔、螢幕截圖、使用條款等）不需要以英文表示，只要要約說明以短語開頭，"此應用程式僅以 [非英語語言]提供。 提供*有用的連結 URL*以提供產品/服務清單內容中所用語言以外的語言提供內容也是可接受的。

### <a name="name"></a>名稱

您在此處輸入的名稱將作為產品/服務清單的標題顯示給客戶。 此欄位預填充了創建產品/服務時為 **"產品/服務"別名**輸入的文本，但您可以更改此值。 此名稱可以注冊商標（您可以包括商標或版權符號）。 名稱不能超過 50 個字元，不能包含任何表情符號。

### <a name="short-description"></a>簡短描述

提供優惠的簡短說明（最多 100 個字元）。 此說明可用於市場搜尋結果。

### <a name="description"></a>描述

提供更久的報價說明（最多 3，000 個字元）。 此描述將顯示在市場清單概述中的客戶。 包括您的產品/服務的價值主張、主要優勢、類別和/或行業協會、應用內購買機會以及任何必需的披露。

編寫說明的一些提示：  

- 在您的描述中的前幾個句子清楚描述您供應項目的價值主張。 在價值主張中包括以下資訊：
  - 產品的描述
  - 從產品中受益的使用者類型
  - 產品解決的客戶需求或痛苦
- 請記住，前幾個句子可能會顯示在搜尋引擎結果中。  
- 請勿依賴特色與功能來銷售您的產品。 反之，要鎖定在您所提供的價值。  
- 請盡可能使用業界特定詞彙或凸顯優點的字眼。
- 請考慮使用 HTML 標籤來格式化您的描述，使其更具吸引力。

為了使您的報價描述更具吸引力，請使用豐富的文字編輯器來設置您的說明格式。

![使用富文字編輯器](./media/text-editor2.png)

使用以下說明使用富文字編輯器：

- 要更改內容的格式，請突出顯示要設置格式的文本並選擇文本樣式，如下所示：

     ![使用富文字編輯器更改文本格式](./media/text-editor3.png)

- 要向文本添加專案符號或編號清單，請使用以下選項：

     ![使用富文字編輯器添加清單](./media/text-editor4.png)

- 要向文本添加或刪除縮進，請使用以下選項：

     ![使用富文字編輯器縮進](./media/text-editor5.png)

### <a name="search-keywords"></a>搜尋關鍵字

您可以選擇輸入最多三個搜索關鍵字，説明客戶在市場上找到您的產品/服務。 為獲得最佳效果，請嘗試在描述中使用這些關鍵字。

### <a name="products-your-app-works-with"></a>與您的應用程式一起運作的產品

如果您想讓客戶知道你的應用適用于特定產品，請在此處輸入最多三個產品名稱。

### <a name="support-urls"></a>支援 URL

本部分允許您提供連結，説明客戶更好地瞭解您的產品/服務。

#### <a name="help-link"></a>説明連結

輸入客戶可以詳細瞭解您的優惠的 URL。

#### <a name="privacy-policy-url"></a>隱私權原則 URL

輸入組織隱私政策的 URL。 您有責任確保您的應用符合隱私法律和法規，並提供有效的隱私政策。

### <a name="contacts"></a>連絡人

在本節中，提供**支援連絡人**和**工程連絡人**的名稱、電子郵件和電話號碼。 此資訊不會向客戶顯示，但可供 Microsoft 使用，並且可能會提供給 CSP 合作夥伴。

在 **"支援"連絡人**部分中，提供**支援 URL，** 其中 CSP 合作夥伴可以找到對您的產品/服務的支援。

### <a name="supporting-documents"></a>證明檔

在此處至少提供一份（最多三份）相關行銷文檔，如白皮書、摺頁冊、檢查表或演示文稿。 這些檔必須採用 .pdf 格式。

### <a name="marketplace-images"></a>市場映射

在本節中，您可以提供向客戶展示產品/服務時將使用的徽標和圖像。 所有圖像必須採用 .png 格式。

#### <a name="store-logos"></a>Microsoft Store標誌。

提供兩種尺寸的產品徽標：**小 （48 x 48）** **和大 （216 x 216）。**

#### <a name="hero"></a>主圖

英雄形象是可選的。 如果提供一個，它必須測量 815 x 290 圖元。

#### <a name="screenshots"></a>螢幕擷取畫面

添加顯示優惠工作原理的螢幕截圖。 至少需要一個螢幕截圖，您最多隻能添加五個螢幕截圖。 所有螢幕截圖必須為 1280 x 720 圖元。

#### <a name="videos"></a>影片

您可以選擇添加最多四個演示您的優惠的視頻。 這些視頻應託管在 YouTube 和/或 Vimeo 上。 對於每個視頻，輸入視頻的名稱、URL 和視頻的縮略圖圖像（1280 x 720 圖元）

#### <a name="additional-marketplace-listing-resources"></a>其他市場清單資源

- [市場產品/服務清單的最佳實踐](https://docs.microsoft.com/azure/marketplace/gtm-offer-listing-best-practices)

## <a name="availability"></a>可用性

"**可用性"** 頁面為您提供了有關產品/服務在何處以及如何提供的選項。

### <a name="markets"></a>市場

本部分允許您指定報價可用的市場。 為此，請選擇 **"編輯市場"，該市場**將顯示 **"市場選擇**"快顯視窗。

預設情況下，不選擇任何市場。 選擇至少一個市場來發佈您的報價。 按一下 **"選擇全部"** 以使您的產品/服務在每個可能的市場都可用，或選擇要添加的特定市場。 完成後，選擇"**保存**"。

此處的選擇僅適用于新的收購;如果有人已經擁有你的應用程式在某一市場，你後來刪除該市場，誰已經在這個市場的報價的人可以繼續使用它，但在這個市場沒有新的客戶將能夠得到您的報價。

> [!IMPORTANT]
> 您有責任滿足任何當地法律要求，即使這些要求未在此處或合作夥伴中心列出。

請記住，即使您選擇所有市場，當地法律和限制或其他因素也可能阻止某些優惠在某些國家和地區上市。

### <a name="preview-audience"></a>預覽受眾

在將優惠即時發佈到更廣泛的市場產品/服務之前，您首先需要將其提供給有限的**預覽版受眾**。 在此處輸入**Hide 鍵**（僅使用小寫字母和/或數位的任何字串）。 預覽受眾的成員可以使用此隱藏鍵作為權杖，在市場中查看優惠的預覽。

然後，當您準備好使產品/服務可用並刪除預覽限制時，您需要刪除**隱藏鍵**並再次發佈。

## <a name="technical-configuration"></a>技術配置

"**技術配置**"頁定義用於連接到產品/服務的技術詳細資訊。 此連接使我們能夠為最終客戶提供您的報價，如果他們選擇購買。

### <a name="solution-identifier"></a>解決方案識別碼

為您的解決方案提供解決方案識別碼 （GUID）。

要查找解決方案識別碼，請：
1. 在 Microsoft 動態生命週期服務 （LCS） 中，選擇**解決方案管理**。
2. 選擇解決方案，然後在**包概述**中查找**解決方案識別碼**。 如果識別碼為空，請選擇 **"編輯"** 並重新發佈包，然後重試。

### <a name="release-version"></a>發行版本

選擇此解決方案使用的財務和運營的 Dynamics 365 版本。

## <a name="test-drive-technical-configuration"></a>試駕技術配置

如果在["產品/服務設置"](#offer-setup)頁中選擇 **"啟用試駕**"，則需要在此處提供詳細資訊，以便客戶體驗您的產品/服務中的試駕。

通過 **"測試磁碟機**"頁面，您可以設置演示（或"試駕"），使客戶能夠在承諾購買之前試用您的產品/服務。 在文章"[什麼是試駕"中瞭解更多資訊。](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/what-is-test-drive) 如果您不想再為產品/服務提供試駕，請返回**["產品/服務設置](#offer-setup)**"頁面並取消選中**啟用試駕**。

提供以下類型的測試磁碟機，每種磁碟機都有自己的技術配置要求。

- [Azure 資源管理器](#technical-configuration-for-azure-resource-manager-test-drive)
- [Dynamics 365](#technical-configuration-for-dynamics-365-test-drive)
- [邏輯應用程式](#technical-configuration-for-logic-app-test-drive)
- [電源 BI（](#technical-configuration-not-required-for-power-bi-test-drives)不需要技術配置）

### <a name="technical-configuration-for-azure-resource-manager-test-drive"></a>Azure 資源管理器測試磁碟機的技術配置

包含構成解決方案的所有 Azure 資源的部署範本。 適合此方案的產品僅使用 Azure 資源。 詳細瞭解如何設置 Azure[資源管理器測試磁碟機](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/azure-resource-manager-test-drive)。

- **區域**（必需）：目前有 26 個 Azure 支援的區域，可以在其中提供您的試駕。 通常，您需要在預期客戶數量最多的區域提供試駕，以便他們選擇最接近的區域以獲得最佳性能。 您需要確保允許訂閱部署所選擇的每個區域所需的所有資源。

- **實例**：選擇類型（熱或冷）和可用實例數，這些實例將乘以產品/服務可用的區域數。

**熱**：這種類型的實例已部署，並等待每個選定區域的訪問。 客戶可以立即訪問測試磁碟機*的熱*實例，而無需等待部署。 缺點是，這些執行個體一直在您的 Azure 訂用帳戶中執行，因此將會產生可觀的運作成本。 強烈建議至少使用一個*熱*實例，因為大多數客戶不希望等待完全部署，如果沒有*熱*實例，則會導致客戶使用量下降。

**冷**：這種類型的實例表示每個區域可能部署的實例總數。 冷實例要求在整個測試磁碟機資源管理器範本在客戶請求試駕時進行部署，因此*冷*實例的載入速度比*熱*實例慢得多。 權衡是，您只需要在試駕期間付費，它*並不總是*像*使用熱*實例那樣在 Azure 訂閱上運行。

- **試用 Azure 資源管理器範本**：上載包含 Azure 資源管理器範本的 .zip。  在快速入門文章"[使用 Azure 門戶創建和部署 Azure 資源管理器範本"](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal)中瞭解有關創建 Azure 資源管理器範本詳細資訊。

- **試駕持續時間**（必需）：輸入試駕將保持活動狀態的時間長度（以小時數計）。 在此持續時間過後，試用產品將會自動終止。 此持續時間只能由整小時數設置（例如，"2"小時;"1.5"無效）。

### <a name="technical-configuration-for-dynamics-365-test-drive"></a>Dynamics 365 試駕的技術配置

Microsoft 可以通過託管和維護使用此類測試磁碟機的服務預配和部署來消除設置測試磁碟機的複雜性。 無論試駕是面向業務中心、客戶參與還是運營受眾，此類託管試駕的配置都是相同的。

- **最大併發測試磁碟機**（必需）：設置一次可以使用試駕的最大客戶數。 每個併發使用者將在測試磁碟機處於活動狀態時使用 Dynamics 365 許可證，因此您需要確保有足夠的許可證來支援最大限制集。 建議值為 3-5 個。

- **試駕持續時間**（必需）：通過定義小時數輸入試駕將保持活動狀態的時間長度。 在此多小時後，會話將結束，不再使用您的許可證之一。 根據產品/服務的複雜性，我們建議值為 2-24 小時。 此持續時間只能由整小時數設置（例如，"2"小時;"1.5"無效）。  如果使用者時間已用完，並且希望再次訪問試駕，使用者可以請求新的會話。

- **實例 URL（** 必需）：客戶將開始其試駕的 URL。 通常，使用安裝示例資料（例如，）`https://testdrive.crm.dynamics.com`運行應用的 Dynamics 365 實例的 URL。

- **實例 Web API URL（** 必需）：通過登錄到 Microsoft 365 帳戶並導航到 **"設置**\&"gt，檢索 Dynamics 365 實例的 Web API URL;**自訂**\&gt;**開發人員資源**\&gt;**實例 Web API（服務根 URL），** 複製此處找到的 URL（`https://testdrive.crm.dynamics.com/api/data/v9.0`例如。

- **角色名稱**（必需）：提供您在自訂 Dynamics 365 試駕中定義的安全形色名稱。 這將在使用者試駕期間（例如，試駕角色）分配給使用者。

### <a name="technical-configuration-for-logic-app-test-drive"></a>邏輯應用試駕的技術配置

任何自訂產品都應使用這種類型的試駕部署範本，該範本包含各種複雜的解決方案體系結構。 有關設置邏輯應用測試磁碟機的詳細資訊，請訪問 GitHub 上的[操作](https://github.com/Microsoft/AppSource/blob/master/Setup-your-Azure-subscription-for-Dynamics365-Operations-Test-Drives.md)[和客戶參與度](https://github.com/Microsoft/AppSource/wiki/Setting-up-Test-Drives-for-Dynamics-365-app)。

- **區域**（必需，單選下拉清單）：目前有 26 個 Azure 支援的區域，可以在其中提供您的試駕。 邏輯應用的資源將部署在您選擇的區域中。 如果邏輯應用在特定區域中存儲了任何自訂資源，請確保在此處選擇該區域。 確保區域的自訂資源可用的最佳方式是在門戶中的 Azure 訂閱上本地完全部署邏輯應用，並在進行此選擇之前驗證其是否正常工作。

- **最大併發測試磁碟機**（必需）：設置一次可以使用試駕的最大客戶數。 這些測試磁碟機已部署，使客戶能夠立即訪問它們，而無需等待部署。

- **試駕持續時間**（必需）：輸入試駕將保持活動狀態的時間長度（以小時數計）。 此時間段結束後，試駕將自動終止。

- **Azure 資源組名稱**（必需）：輸入邏輯應用測試磁碟機保存的[Azure 資源組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups)名稱。

- **Azure 邏輯應用名稱**（必需）：輸入將測試磁碟機分配給使用者的邏輯應用的名稱。 此邏輯應用必須保存在上面的 Azure 資源組中。

- **取消預配邏輯應用名稱**（必需）：輸入邏輯應用的名稱，該應用在客戶完成後取消配置試駕。 此邏輯應用必須保存在上面的 Azure 資源組中。

### <a name="technical-configuration-not-required-for-power-bi-test-drives"></a>Power BI 測試磁碟機不需要技術配置

想要演示互動式 Power BI 視覺物件的產品可以使用嵌入式連結共用自訂儀表板作為試駕，無需進一步的技術配置。 詳細瞭解如何設置[Power BI](https://docs.microsoft.com/power-bi/service-template-apps-overview)範本應用。

### <a name="deployment-subscription-details"></a>部署訂閱詳細資訊

為了代表您部署測試磁碟機，請創建並提供單獨的唯一 Azure 訂閱。 （電源 BI 測試磁碟機不需要）。

- **Azure 訂閱 ID（Azure**資源管理器和邏輯應用需要）：輸入訂閱 ID 以授予對 Azure 帳戶服務的存取權限，以便進行資源使用方式報告和計費。 如果尚未創建，我們建議您考慮[創建單獨的 Azure 訂閱](https://docs.microsoft.com/azure/billing/billing-create-subscription)以用於測試磁碟機。 您可以通過登錄到[Azure 門戶](https://portal.azure.com/)並導航到左側功能表的 **"訂閱"** 選項卡來查找 Azure 訂閱 ID。 選擇該選項卡將顯示您的訂閱 ID（例如，"a83645ac-1234-5ab6-6789-1h234g764ghty"）。

- **Azure AD 租戶 ID（** 必需）：輸入 Azure 活動目錄 （AD）[租戶 ID](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)。 要查找此 ID，請登錄到[Azure 門戶](https://portal.azure.com/)，選擇左側功能表中的"活動目錄"選項卡，選擇 **"屬性**"，然後查找列出的**目錄 ID**號（例如，50c464d3-4930-494c-963c-1e951d15360e）。 您還可以使用功能變數名稱 URL 在 ：[https://www.whatismytenantid.com](https://www.whatismytenantid.com)查找組織的租戶 ID。

- **Azure AD 租戶名稱**（動態 365 需要）：輸入 Azure 活動目錄 （AD） 名稱。 要查找此名稱，請登錄到[Azure 門戶](https://portal.azure.com/)，在右上角，租戶名稱將列在您的帳戶名稱下。

- **Azure AD 應用 ID（** 必需）：輸入 Azure 活動目錄 （AD）[應用程式 ID](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)。 要查找此 ID，請登錄到[Azure 門戶](https://portal.azure.com/)，選擇左側功能表中的"活動目錄"選項卡，選擇**應用註冊**，然後查找列出的應用程式**ID**號（例如，50c464d3-4930-494c-963c-1e951d15360e）。

- **Azure AD 應用用戶端機密**（必需）：輸入 Azure AD 應用程式[用戶端金鑰](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#certificates-and-secrets)。 要查找此值，請登錄到 Azure[門戶](https://portal.azure.com/)。 選擇左側功能表中的**Azure 活動目錄**選項卡，選擇**應用註冊**，然後選擇試駕應用。 接下來，選擇**證書和機密**，選擇 **"新建用戶端機密**"，輸入說明，選擇 **"從不在過期**"下，然後選擇"**添加**"。 **Never** 請確保向下複製該值。 （在記下該值之前，不要離開頁面，否則您將無法訪問該值。

在繼續下一節之前，請記住**保存**！

### <a name="test-drive-marketplace-listings"></a>試駕市場清單

**"測試磁碟機**"選項卡下的 **"應用商店清單**"選項顯示您的試駕可用的語言。 目前，**英語（美國）** 是唯一可用的地點。 選擇語言名稱以輸入描述試駕體驗的資訊。

- **說明**（必需）：描述您的試駕、演示的內容、供使用者試驗的目標、要流覽的功能以及任何相關資訊，以説明使用者確定是否獲取您的產品/服務。 此欄位中最多可輸入 3，000 個字元的文本。

- **訪問資訊**（Azure 資源管理器和邏輯測試磁碟機所需的資訊）：解釋客戶需要瞭解哪些內容才能訪問和使用此測試磁碟機。 演練使用產品/服務的方案，以及客戶在整個試駕中訪問功能的準確情況。 此欄位中最多可輸入 10，000 個字元的文本。

- **使用者手冊**（必需）：深入瞭解您的試駕體驗。 《使用者手冊》應準確介紹您希望客戶從體驗試駕中獲得什麼，並作為他們可能存在的任何問題的參考。 檔必須採用 PDF 格式，並在上載後命名（最多 255 個字元）。

- **視頻：添加視頻**（可選）：視頻可以上傳到 YouTube 或 Vimeo，並在此處引用連結和縮略圖圖像 （533 x 324 圖元），以便客戶可以查看資訊的演練，以説明他們更好地瞭解試駕，包括如何成功使用您的產品/服務的功能，並瞭解突出其優勢的方案。
  - **名稱**（必需）
  - **網址（僅限 YouTube 或 Vimeo）（** 必填）
  - **縮略圖 （533 x 324px）**： 影像檔必須採用 PNG 格式。

## <a name="supplemental-content"></a>補充內容

此頁面允許您提供有關您的優惠的其他資訊，以説明我們驗證您的產品/服務。 此資訊不會向客戶顯示或發佈到市場。

### <a name="validation-assets"></a>驗證資產

在此部分上載[自訂分析報告 （CAR）。](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/dev-tools/customization-analysis-report) 此報告是根據一組預定義的最佳實踐規則分析自訂和擴展模型生成的。

此檔必須採用 .xls 或 .xlsx 格式。 如果您有多個報表，則可以上載包含所有報表的 .ZIP 檔案。

### <a name="does-solution-include-localizations"></a>解決方案是否包括當地語系化？

如果解決方案允許使用本地標準和策略（例如，如果它適應不同國家/地區所需的不同工資規則），請選擇 **"是**"。 否則，請選取 [否]****。

### <a name="does-solution-enable-translations"></a>解決方案是否支援翻譯？

如果解決方案中的文本可以翻譯成其他語言，請回答 **"是**"。 否則，請選取 [否]****。

## <a name="publish"></a>發佈

### <a name="submit-offer-to-preview"></a>提交優惠以預覽

完成產品/服務的所有必需部分後，選擇門戶右上角的**發佈**。 您將被重定向到 **"審閱"和"發佈"** 頁。

如果您是首次發佈此產品/服務，您可以：

- 請參閱產品/服務的每個部分的完成狀態。
    - *未啟動*- 表示該部分尚未觸摸，應完成。
    - *不完整*- 表示該節有需要修復的錯誤或需要提供更多資訊。 返回部分並更新它。
    - *完成*- 表示部分已完成，提供了所有必需的資料，並且沒有錯誤。 報價的所有部分都必須處於完全狀態，然後才能提交報價。
- 在 **"認證備註"** 部分中，除了任何有助於瞭解應用的補充說明外，請向認證團隊提供測試說明，以確保正確測試應用。
- 通過選擇 **"提交**"提交要發佈的產品/ 我們將向您發送一封電子郵件，告知您何時可以審核和批准優惠的預覽版本。 返回合作夥伴中心，選擇 **"Go-live"** 以優惠形式向公眾發佈您的優惠（如果是私人優惠，則向私人受眾發佈）。

## <a name="next-steps"></a>後續步驟

- [更新商業市集中的現有供應項目](./update-existing-offer.md)

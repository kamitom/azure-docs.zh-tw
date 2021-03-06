---
title: 架構概觀 - Azure Active Directory | Microsoft Docs
description: 了解什麼是 Azure Active Directory 租用戶，以及如何使用 Azure Active Directory 管理 Azure。
services: active-directory
author: msaburnley
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 05/23/2019
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 854fb4649f8c1113f20abe5807dd0ce473ba6ee3
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "77368059"
---
# <a name="what-is-the-azure-active-directory-architecture"></a>什麼是 Azure Active Directory 架構？

Azure Active Directory (Azure AD) 可讓您安全地管理您使用者的 Azure 服務和資源存取權。 Azure AD 隨附一套完整的身分識別管理功能。 如需 Azure AD 功能的詳細資訊，請參閱[什麼是 Azure Active Directory？](active-directory-whatis.md)

您可以使用 Azure AD，建立及管理使用者和群組，並啟用權限以允許和拒絕企業資源存取。 如需身分識別管理的相關資訊，請參閱 [Azure 身分識別管理的基本概念](active-directory-whatis.md)。

## <a name="azure-ad-architecture"></a>Azure AD 架構

Azure AD 分佈各地的架構結合廣泛監視、自動化重設路徑、容錯移轉和復原功能，可為客戶提供全公司的可用性和效能。

本文涵蓋下列架構元素︰

*   服務架構設計
*   延展性
*   持續可用性
*   資料中心

### <a name="service-architecture-design"></a>服務架構設計

若要建置可存取且可使用、資料豐富的系統，最常見的方式就是透過獨立建置組塊或縮放單位。 對於 Azure AD 資料層，縮放單位稱之為「分割區」**。

資料層有數個可提供讀寫功能的前端服務。 下圖顯示了單目錄磁碟分割的元件如何在整個地理分佈的資料中心中傳遞。

  ![單目錄區域圖](./media/active-directory-architecture/active-directory-architecture.png)

Azure AD 架構的元件包括主要複本和次要複本。

#### <a name="primary-replica"></a>主要複本

「主要複本」** 會接收其所屬資料分割的所有「寫入」**。 所有寫入作業會先立即複寫至不同資料中心內的次要複本，然後將成功傳回給呼叫端，因而確保寫入的地理備援持久性。

#### <a name="secondary-replicas"></a>您應該

所有目錄*讀取*都從*輔助副本*提供服務，這些副本位於物理位於不同地理位置的資料中心。 次要複本有許多個，因為資料會以非同步方式複寫。 目錄讀取（如身份驗證請求）是從靠近客戶的資料中心提供服務的。 次要複本負責提供讀取延展性。

### <a name="scalability"></a>延展性

延展性是擴充服務以符合效能需求的能力。 寫入延展性是由分割資料來達成。 讀取延展性則是由將資料從一個資料分割複寫到分散世界各地的多個次要複本來達成。

來自目錄應用程式之要求會路由傳送至它們實際最接近的資料中心。 寫入會透明地重新導向到主要複本，以提供讀寫一致性。 次要複本會大幅擴充資料分割的規模，因為目錄通常會為讀取提供服務。

目錄應用程式會連接到最接近的資料中心。 此連線可改善效能，因此有可能相應放大。 由於目錄分割可以有許多次要複本，所以次要複本可放置於較接近目錄用戶端的地方。 只有密集寫入的內部目錄服務元件會直接以使用中主要複本為目標。

### <a name="continuous-availability"></a>持續可用性

可用性 (或運作時間) 定義系統執行不中斷的能力。 Azure AD 的高可用性的關鍵是，服務可以快速跨多個地理分佈的資料中心轉移流量。 每個資料中心都是獨立的，這支援去相關故障模式。 通過這種高可用性設計，Azure AD 不需要停機進行維護活動。

相較於企業 AD 設計，Azure AD 的分割區設計已經過簡化，使用單一主機設計，其中包含仔細協調且具決定性的主要複本容錯移轉程序。

#### <a name="fault-tolerance"></a>容錯

如果可容忍硬體、網路和軟體失敗，則系統的可用性更高。 若為目錄上的每個資料分割，則存在高可用性的主複本︰即主要複本。 只有對磁碟分割的寫入會在此複本執行。 此複本正受到持續且嚴密的監視，而如果偵測到失敗，寫入即可立即轉換至另一個複本 (會成為新的主要複本）。 在容錯移轉期間，通常可能喪失寫入可用性 1-2 分鐘。 在這段期間，讀取可用性不受影響。

讀取作業 (其數目大幅超出寫入的數量) 只會移至次要複本。 由於次要複本具等冪性，將讀取導向另一個複本 (通常位於相同資料中心) 可輕鬆補償在指定之資料分割中任何一個複本的遺失。

#### <a name="data-durability"></a>資料耐久性

在確認寫入之前，寫入將持久地承諾到至少兩個資料中心。 首先在主資料庫上提交寫入，然後立即將寫入複製到至少一個其他資料中心，從而發生這種情況。 此寫入操作可確保託管主資料庫的資料中心的潛在災難性丟失不會導致資料丟失。

Azure AD 會將[復原時間目標 (RTO)](https://en.wikipedia.org/wiki/Recovery_time_objective) 維持為零，因此在容錯移轉時不會遺失資料。 這包括：

* 權杖發行和目錄讀取
* 目錄寫入只允許約 5 分鐘的 RTO

### <a name="datacenters"></a>資料中心

Azure AD 的複本會儲存在位於世界各地的資料中心。 有關詳細資訊，請參閱[Azure 全域基礎結構](https://azure.microsoft.com/global-infrastructure/)。

Azure AD 跨資料中心運行，具有以下特徵：

* 驗證、圖表和其他 AD 服務位於閘道服務後方。 閘道會管理這些服務的負載平衡。 如果使用交易健康狀態探查偵測到任何狀況不良的伺服器，它就會自動進行容錯移轉。 基於這些運行狀況探測，閘道動態地將流量路由到健康的資料中心。
* 對於*讀取*，目錄具有輔助副本和相應的前端服務，在多個資料中心中運行的主動-主動配置。 如果整個資料中心發生故障，流量將自動路由到其他資料中心。
 *對於*寫入*，目錄將通過計畫（新主資料庫同步到舊主主）或緊急容錯移轉過程跨資料中心容錯移轉主（主主）副本。 資料持久性是通過將任何提交複製到至少兩個資料中心來實現的。

#### <a name="data-consistency"></a>資料一致性

目錄模型是其中一項最終一致性。 以非同步方式複寫的分散式系統有一個典型問題，就是從「特定」複本傳回的資料可能不是最新狀態。 

Azure AD 會對以次要複本為目標的應用程式提供讀寫一致性，其做法是將其寫入路由傳送至主要複本，並以同步方式將寫入提取回到次要複本。

使用 Azure AD 的 Microsoft 圖形 API 進行應用程式寫入從保持關聯到目錄副本，以便進行讀寫一致性。 Microsoft Graph API 服務維護一個邏輯會話，該會話與用於讀取的輔助副本具有相關性;相關性在服務使用輔助副本資料中心中的分散式緩存的"副本權杖"中捕獲。 此權杖則會接著用於相同邏輯工作階段中的後續作業。 要繼續使用相同的邏輯會話，後續請求必須路由到同一 Azure AD 資料中心。 如果目錄用戶端請求被路由到多個 Azure AD 資料中心，則無法繼續邏輯會話;如果發生這種情況，則用戶端具有多個具有獨立讀寫一致性的邏輯會話。

 >[!NOTE]
 >寫入會立即複寫到邏輯工作階段的讀取所發行至的次要複本。

#### <a name="backup-protection"></a>備份保護

目錄會對使用者和租用戶實作虛刪除 (而不是實刪除)，以便在遭客戶意外刪除的情況下輕鬆復原。 如果您的租用戶系統管理員不小心刪除使用者，他們可以輕鬆地復原並還原已刪除的使用者。

Azure AD 會實作所有資料的每日備份，因此可以在任何邏輯刪除或損毀的情況下可靠地還原資料。 資料層會運用錯誤修正碼，以便檢查是否有錯誤並自動更正特定類型的磁碟錯誤。

#### <a name="metrics-and-monitors"></a>計量和監視

執行高可用性服務需要世界級的計量和監視功能。 Azure AD 會持續分析及報告其每項服務的重要服務健康狀態計量和成功準則。 也會在每項 Azure AD 服務及所有服務內持續開發並微調計量，以及針對每個案例進行監視與警示。

如果任何 Azure AD 服務無法如預期般運作，則會立即採取行動以盡快還原功能。 Azure AD 追蹤的最重要計量是如何快速偵測及緩和客戶的即時網站問題。 我們大舉投入監視和警示，以將偵測時間降至最低 (TTD 目標︰<5 分鐘)，並投入作業整備性以將緩和時間降至最低 (TTM 目標︰<30 分鐘)。

#### <a name="secure-operations"></a>安全作業

使用任何作業的作業控制項 (例如 Multi-Factor Authentication (MFA))，以及稽核所有的作業。 此外，使用即時提高權限系統，隨時對任何操作工作授與必要的暫時存取權。 如需詳細資訊，請參閱[受信任的雲端](https://azure.microsoft.com/support/trust-center)。

## <a name="next-steps"></a>後續步驟

[Azure 活動目錄開發人員指南](https://docs.microsoft.com/azure/active-directory/develop)
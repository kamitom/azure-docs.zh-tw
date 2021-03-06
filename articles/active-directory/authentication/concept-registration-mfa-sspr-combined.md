---
title: SSPR 與 MFA 的合併註冊 - Azure Active Directory
description: Azure AD 多重要素驗證和自助服務密碼重設註冊（預覽版）
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/06/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: sahenry
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4cb5aca128679b21072a2a3daa503dc43a8e2885
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "78942883"
---
# <a name="combined-security-information-registration-preview"></a>組合安全資訊註冊（預覽）

在合併註冊之前，使用者分別註冊了 Azure 多重要素驗證和自助服務密碼重設 （SSPR） 的身份驗證方法。 人們感到困惑的是，類似的方法被用於多重要素驗證和SSPR，但他們必須註冊這兩個功能。 現在，通過合併註冊，使用者可以註冊一次，並獲得多重要素驗證和 SSPR 的優勢。

![顯示使用者已註冊安全資訊的我的個人資料](media/concept-registration-mfa-sspr-combined/combined-security-info-defualts-registered.png)

在啟用新體驗之前，請查看此以管理員為中心的文檔和以使用者為中心的文檔，以確保您瞭解此功能的功能和效果。 基於[使用者文檔](../user-help/user-help-security-info-overview.md)進行培訓，為使用者準備新體驗，並説明確保成功推出。

Azure AD 組合安全資訊註冊目前不適用於 Azure 美國政府、Azure 德國或 Azure 中國 21Vianet 等國家雲。

|     |
| --- |
| 多重要素驗證和 Azure 活動目錄 （Azure AD） 自助服務密碼重設的組合安全資訊註冊是 Azure AD 的公共預覽功能。 如需有關預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。|
|     |

> [!IMPORTANT]
> 啟用原始預覽和增強組合註冊體驗的使用者將看到新行為。 啟用兩種體驗的使用者將僅看到新的"我的設定檔"體驗。 新的"我的個人資料"與合併註冊的外觀保持一致，為使用者提供無縫體驗。 使用者可以通過訪問[https://myprofile.microsoft.com](https://myprofile.microsoft.com)查看我的個人資料。

> [!NOTE] 
> 在嘗試訪問安全資訊選項時，可能會遇到錯誤訊息。 例如，"對不起，我們無法登錄"。 在這種情況下，請確認您沒有任何阻止 Web 瀏覽器上協力廠商 Cookie 的配置或群組原則物件。 

我的"設定檔"頁面根據訪問該頁面的電腦的語言設置進行當地語系化。 Microsoft 存儲瀏覽器緩存中使用的最新語言，因此後續訪問頁面的嘗試將繼續以使用的最後一種語言呈現。 如果清除緩存，頁面將重新呈現。 如果要強制特定語言，可以添加到`?lng=<language>`URL 的末尾，其中要呈現的語言的代碼。 `<language>`

![設置 SSPR 或其他安全驗證方法](media/howto-registration-mfa-sspr-combined/combined-security-info-my-profile.png)

## <a name="methods-available-in-combined-registration"></a>組合註冊中可用的方法

組合註冊支援以下身份驗證方法和操作：

|   | 註冊 | 變更 | 刪除 |
| --- | --- | --- | --- |
| Microsoft 驗證器 | 是（最多 5 個） | 否 | 是 |
| 其他驗證器應用 | 是（最多 5 個） | 否 | 是 |
| 硬體權杖 | 否 | 否 | 是 |
| 電話 | 是 | 是 | 是 |
| 備用手機 | 是 | 是 | 是 |
| 辦公室電話 | 否 | 否 | 否 |
| 電子郵件 | 是 | 是 | 是 |
| 安全性問題 | 是 | 否 | 是 |
| 應用程式密碼 | 是 | 否 | 是 |
| FIDO2 安全金鑰<br />*僅從[安全資訊](https://mysignins.microsoft.com/security-info)頁面進行託管模式*| 是 | 是 | 是 |

> [!NOTE]
> 應用密碼僅適用于已強制實施多重要素驗證的使用者。 應用密碼對於通過條件訪問策略啟用多重要素驗證的使用者不可用。

使用者可以將以下選項之一設置為預設的多重要素驗證方法：

- 微軟身份驗證器 – 通知。
- 身份驗證器應用或硬體權杖 + 代碼。
- 電話。
- 簡訊。

隨著我們繼續向 Azure AD 添加更多身份驗證方法，這些方法將在合併註冊中可用。

## <a name="combined-registration-modes"></a>組合註冊模式

合併註冊有兩種模式：中斷和管理。

- **中斷模式**是類似于嚮導的體驗，在使用者註冊或刷新登錄時呈現給使用者。

- **管理模式**是使用者設定檔的一部分，允許使用者管理其安全資訊。

對於這兩種模式，以前註冊了可用於多重要素驗證的方法的使用者在訪問其安全資訊之前需要執行多重要素驗證。

### <a name="interrupt-mode"></a>中斷模式

如果為租戶啟用了兩種策略，則合併註冊同時尊重多重要素驗證和 SSPR 策略。 這些策略控制使用者在登錄期間是否中斷註冊，以及哪些方法可用於註冊。

以下是可能提示使用者註冊或刷新其安全資訊的幾個方案：

- 通過身份保護強制註冊的多重要素驗證註冊：在登錄期間要求使用者註冊。 它們註冊多重要素驗證方法和 SSPR 方法（如果使用者啟用了 SSPR）。
- 通過每個使用者多重要素驗證強制進行多重要素驗證註冊：在登錄期間要求使用者註冊。 它們註冊多重要素驗證方法和 SSPR 方法（如果使用者啟用了 SSPR）。
- 通過條件訪問或其他策略強制進行多重要素驗證註冊：當使用者使用需要多重要素驗證的資源時，系統將被要求進行註冊。 它們註冊多重要素驗證方法和 SSPR 方法（如果使用者啟用了 SSPR）。
- 實施 SSPR 註冊：要求使用者在登錄期間註冊。 它們僅註冊 SSPR 方法。
- 實施 SSPR 刷新：使用者需要按管理員設置的時間間隔查看其安全資訊。使用者會顯示其資訊，並可以確認當前資訊或根據需要進行更改。

強制執行註冊時，將顯示使用者從大多數到最低安全點，以符合多重要素驗證和 SSPR 策略所需的最少方法數。

例如：

- 為 SSPR 啟用了使用者。 SSPR 策略需要兩種方法來重置，並啟用了移動應用代碼、電子郵件和電話。
   - 此使用者需要註冊兩種方法。
      - 預設情況下，使用者將顯示身份驗證器應用和手機。
      - 使用者可以選擇註冊電子郵件，而不是驗證器應用或電話。

此流程圖描述在登錄期間中斷註冊時向使用者顯示哪些方法：

![組合安全資訊流程圖](media/concept-registration-mfa-sspr-combined/combined-security-info-flow-chart.png)

如果同時啟用了多重要素驗證和 SSPR，我們建議您強制實施多重要素驗證註冊。

如果 SSPR 策略要求使用者定期查看其安全資訊，則使用者在登錄過程中會中斷並顯示其所有註冊方法。 如果當前資訊是最新的，他們可以確認當前資訊，也可以在需要時進行更改。 使用者在訪問此頁面時必須執行多重要素驗證。

### <a name="manage-mode"></a>管理模式

使用者可以通過訪問[https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)或從"我的個人資料"中選擇**安全資訊**來訪問管理模式。 從那裡，使用者可以添加方法、刪除或更改現有方法、更改預設方法等。

## <a name="key-usage-scenarios"></a>關鍵使用方案

### <a name="set-up-security-info-during-sign-in"></a>在登入期間設定安全性資訊

管理員已強制執行註冊。

使用者尚未設置所有必需的安全資訊並轉到 Azure 門戶。 輸入使用者名和密碼後，系統會提示使用者設置安全資訊。 然後，使用者按照嚮導中顯示的步驟設置所需的安全資訊。 如果您的設置允許，使用者可以選擇設置預設顯示的方法以外的方法。 完成嚮導後，使用者將查看他們設置的方法及其用於多重要素驗證的預設方法。 要完成安裝過程，使用者將確認資訊並繼續訪問 Azure 門戶。

### <a name="set-up-security-info-from-my-profile"></a>從"我的個人資料"設置安全資訊

管理員尚未強制執行註冊。

尚未設置所有必需的安全資訊的使用者將轉到[https://myprofile.microsoft.com](https://myprofile.microsoft.com)。 使用者選擇左側窗格中**的安全資訊**。 從那裡，使用者選擇添加方法，選擇任何可用的方法，並按照步驟設置該方法。 完成後，使用者將看到剛剛在"安全資訊"頁上設置的方法。

### <a name="delete-security-info-from-my-profile"></a>從"我的個人資料"中刪除安全資訊

以前至少設置一個方法的使用者將導航到[https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)。 使用者選擇刪除以前註冊的方法之一。 完成後，使用者不再在"安全資訊"頁上看到該方法。

### <a name="change-the-default-method-from-my-profile"></a>從"我的設定檔"更改預設方法

以前至少設置一種可用於多重要素驗證的方法的使用者將導航到[https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)。 使用者將當前預設方法更改為其他預設方法。 完成後，使用者在"安全資訊"頁上看到新的預設方法。

## <a name="next-steps"></a>後續步驟

[強制使用者重新註冊身份驗證方法](howto-mfa-userdevicesettings.md#manage-authentication-methods)

[在租戶中啟用合併註冊](howto-registration-mfa-sspr-combined.md)

[SSPR 和 MFA 使用和見解報告](howto-authentication-methods-usage-insights.md)

[多因素身份驗證和 SSPR 的可用方法](concept-authentication-methods.md)

[配置自助服務密碼重設](howto-sspr-deployment.md)

[設定 Azure Multi-Factor Authentication](howto-mfa-getstarted.md)

---
title: 條件存取 -需要所有使用者 MFA - Azure 的目錄
description: 建立自訂條件存取原則,要求所有使用者執行多重身份驗證
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 04/02/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: c3e7b1a656c92e37a709b57dae463f6644003e42
ms.sourcegitcommit: 441db70765ff9042db87c60f4aa3c51df2afae2d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2020
ms.locfileid: "80755199"
---
# <a name="conditional-access-require-mfa-for-all-users"></a>條件存取:要求所有使用者使用 MFA

正如微軟身份安全目錄亞歷克斯·韋納特在他的博客文章《[你的Pa$$word中提到的那樣,$word並不重要](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984):

> 您的密碼並不重要,但 MFA 確實如此! 根據我們的研究,如果您使用 MFA,您的帳戶被入侵的可能性要降低 99.9% 以上。

本文中的指南將説明您的組織為您的環境創建平衡的 MFA 策略。

## <a name="user-exclusions"></a>使用者排除

條件存取策略是功能強大的工具,我們建議將以下帳戶排除在策略之外:

* **緊急訪問**或**破玻璃**帳戶,以防止租戶範圍的帳戶鎖定。 在不太可能的情況下,所有管理員都鎖定在租戶之外,您的緊急訪問管理帳戶可用於登錄到租戶,採取措施恢復訪問許可權。
   * 有關詳細資訊,請參閱[" 在 Azure AD 中管理緊急訪問帳戶](../users-groups-roles/directory-emergency-access.md)「一文。
* **服務帳戶****和服務主體**,如 Azure AD 連接同步帳戶。 服務帳戶是非與任何特定使用者關聯的非互動式帳戶。 它們通常由允許以程式設計方式訪問應用程式的後端服務使用,但也用於登錄系統以進行管理。 應排除類似這樣的服務帳戶,因為無法以程式設計方式完成 MFA。 條件訪問不會阻止服務主體進行的呼叫。
   * 如果您的組織在指令碼或程式碼中使用這些帳戶，請考慮將它們取代為[受管理的身分識別](../managed-identities-azure-resources/overview.md)。 作為臨時解決方法,可以從基線策略中排除這些特定帳戶。

## <a name="application-exclusions"></a>應用程式排除

組織可能有許多雲應用程式在使用中。 並非所有應用程式都可能需要同等的安全性。 例如,工資和出勤申請可能需要 MFA,但自助餐廳可能不需要。 管理員可以選擇從策略中排除特定應用程式。

## <a name="create-a-conditional-access-policy"></a>建立條件式存取原則

以下步驟將幫助創建條件訪問策略,以要求分配的管理角色執行多重身份驗證。

1. 以全域管理員、安全管理員或條件存取管理員的身份登入**Azure 門戶**。
1. 瀏覽到**Azure 的目錄** > **安全** > **條件存取**。
1. 選取 [新增原則]****。
1. 為您的策略指定一個名稱。 我們建議組織為其策略的名稱創建有意義的標準。
1. 在 **"配置'** 下,選擇 **"使用者"和"組**"
   1. 在 **'包括**'下,選擇**所有使用者**
   1. 在 **「排除」** 下,選擇 **「使用者」和「組」,** 然後選擇組織的緊急訪問或碎玻璃帳戶。 
   1. 選取 [完成]  。
1. 在 **"雲應用"或"包括** > **"下**,選擇 **"所有雲應用**"。
   1. 在 **「排除 」** 下,選擇不需要多重身份驗證的任何應用程式。
1. 在**條件** > **客戶端應用(預覽)** 下,將 **「設定配置為** **」,** 然後選擇 **「完成**」。
1. 在 **「存取控制** > **授予」下**,選擇 **「授予存取**」,「**需要多重身份驗證**」,然後選擇 **「選擇」。**
1. 確認設置並將 **「啟用策略」** 設定為 **「打開**」。
1. 選擇 **「建立**」以啟用策略。

### <a name="named-locations"></a>具名位置

組織可以選擇將稱為 **「命名位置**」的已知網路位置合併到其條件存取策略中。 這些命名位置可能包括受信任的 IPv4 網路,如主辦公地點的網路。 有關設定命名位置的詳細資訊,請參閱文章[「Azure 活動目錄條件存取」中的位置條件是什麼?](location-condition.md)

在上面的示例策略中,如果從公司網路訪問雲應用,組織可以選擇不需要多重身份驗證。 在這種情況下,他們可以向策略添加以下配置:

1. 在 **"分配"** 下,選擇 **"條件** > **位置**"。
   1. 設定**檔 。**
   1. 包括**任何位置**。
   1. 排除**所有信任的位置**。
   1. 選取 [完成]  。
1. 選取 [完成]  。
1. **保存**策略更改。

## <a name="next-steps"></a>後續步驟

[條件存取一般原則](concept-conditional-access-policy-common.md)

[使用僅條件存取報告模式確定影響](howto-conditional-access-report-only.md)

[使用條件存取「如果」工具模擬登入行為](troubleshoot-conditional-access-what-if.md)

---
title: Windows 認證與 Azure MFA 伺服器 - Azure 活動目錄
description: 部署 Windows 驗證與 Azure Multi-Factor Authentication Server。
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4abfb970ca322724adb0f8919b7509bc8a641378
ms.sourcegitcommit: 62c5557ff3b2247dafc8bb482256fef58ab41c17
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2020
ms.locfileid: "80652806"
---
# <a name="windows-authentication-and-azure-multi-factor-authentication-server"></a>Windows 驗證與 Azure Multi-Factor Authentication Server

使用 Azure Multi-Factor Authentication Server 的 [Windows 驗證] 區段來啟用及設定應用程式的 Windows 驗證。 在設定 Windows 驗證之前，請注意下列清單︰

* 設定之後，重新啟動 Azure Multi-Factor Authentication，「終端機服務」才會生效。
* 如果選取"需要 Azure 多重身份驗證使用者匹配",而您不在使用者清單中,則重新啟動後將無法登錄到電腦。
* 信任的 IP 取決於應用程式是否可以提供用於驗證的用戶端 IP。 目前僅支援終端機服務。  

> [!IMPORTANT]
> 自 2019 年 7 月 1 日起,Microsoft 將不再為新部署提供 MFA 伺服器。 希望用戶進行多重身份驗證的新客戶應使用基於雲的 Azure 多重身份驗證。 在 7 月 1 日之前啟動 MFA 伺服器的現有客戶將能夠像往常一樣下載最新版本、將來的更新並生成啟動認證。

> [!NOTE]
> 在 Windows Server 2012 R2 上，不支援這項功能來保護終端機服務。

## <a name="to-secure-an-application-with-windows-authentication-use-the-following-procedure"></a>要使用 Windows 認證保護應用程式,請使用以下步驟

1. 在 Azure Multi-Factor Authentication Server 中，按一下 [Windows 驗證] 圖示。
   ![MFA 伺服器中的 Windows 認證](./media/howto-mfaserver-windows/windowsauth.png)
2. 核取 [啟用 Windows 驗證]**** 核取方塊。 預設不核取此方塊。
3. [應用程式] 索引標籤可讓系統管理員設定一或多個應用程式要經過 Windows 驗證。
4. 選取伺服器或應用程式 – 指定是否啟用伺服器/應用程式。 按一下 [確定]  。
5. 單擊"**添加..."**
6. [信任的 IP] 索引標籤可讓您針對來自特定 IP 的 Windows 工作階段，略過 Azure Multi-Factor Authentication。 例如，如果員工從辦公室和家裡使用應用程式，您可能決定當他們在辦公室時不要響起 Azure Multi-Factor Authentication 的電話。 為此，您可以將辦公室子網路指定為信任的 IP 項目。
7. 單擊"**添加..."**
8. 如果要跳過單個 IP 位址,請選擇 **「單個 IP」。。**
9. 如果要跳過整個 IP 範圍,請選擇**IP 範圍**。 範例：10.63.193.1-10.63.193.100。
10. 如果要使用子網路表示法指定 IP 範圍,請選擇**子網路**。 輸入子網路的起始 IP，並從下拉式清單中挑選適當的網路遮罩。
11. 按一下 [確定]  。

## <a name="next-steps"></a>後續步驟

- [設定 Azure MFA Server 的協力廠商 VPN 應用裝置](howto-mfaserver-nps-vpn.md)

- [利用 Azure MFA 的 NPS 擴充功能強化現有的驗證基礎結構驗證](howto-mfa-nps-extension.md)

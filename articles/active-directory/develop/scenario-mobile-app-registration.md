---
title: 註冊調用 Web API 的移動應用 |蔚藍
titleSuffix: Microsoft identity platform
description: 瞭解如何建構 Web API 的行動應用程式(應用程式的代碼配置)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: a5b65bbdc790a27a06298a77aac2721e071adec8
ms.sourcegitcommit: d187fe0143d7dbaf8d775150453bd3c188087411
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80882704"
---
# <a name="register-mobile-apps-that-call-web-apis"></a>註冊呼叫 Web API 的行動應用程式

本文包含説明您註冊要創建的移動應用程式的說明。

## <a name="supported-account-types"></a>支援的帳戶類型

移動應用程式支援的帳戶類型取決於要啟用的體驗和要使用的流。

### <a name="audience-for-interactive-token-acquisition"></a>互動式權杖抓取的受眾

大多數移動應用程式使用互動式身份驗證。 如果你的應用使用此形式的身份驗證,則可以從任何[帳戶類型](quickstart-register-app.md#register-a-new-application-using-the-azure-portal)登錄使用者。

### <a name="audience-for-integrated-windows-authentication-username-password-and-b2c"></a>整合 Windows 認證、使用者名稱和密碼和 B2C 的受眾

如果您有通用 Windows 平臺 (UWP) 應用,則可以使用整合 Windows 身份驗證登錄使用者。 要使用整合的 Windows 身份驗證或使用者名稱和密碼身份驗證,應用程式需要在您自己的業務線 (LOB) 開發人員租戶中登錄使用者。 在獨立的軟體供應商 (ISV) 方案中,應用程式可以在 Azure 活動目錄組織中登錄使用者。 Microsoft 個人帳戶不支援這些身份驗證流。

您還可以使用傳遞 B2C 許可權和策略的社交標識登錄使用者。 要使用此方法,只能使用互動式身份驗證和使用者名和密碼身份驗證。 目前僅支援 Xamarin.iOS、Xamarin.Android 和 UWP 上的使用者名和密碼身份驗證。

有關詳細資訊,請參閱[方案和支援的身份驗證串流](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows)和[方案以及受支援的平台和語言](authentication-flows-app-scenarios.md#scenarios-and-supported-platforms-and-languages)。

## <a name="platform-configuration-and-redirect-uris"></a>平台配置與重定向 URI  

### <a name="interactive-authentication"></a>互動式識別驗證

當您構建使用互動式身份驗證的移動應用時,最關鍵的註冊步驟是重定向 URI。 您可以透過[**認證**邊列選項卡上的平台設定](https://aka.ms/MobileAppReg)設定互動式身份驗證。

此體驗將使你的應用能夠通過 Microsoft 身份驗證器 (和 Android 上的 Intune 公司門戶) 獲得單一登錄 (SSO)。 它還將支援設備管理原則。

應用註冊門戶提供預覽體驗,以説明您計算 iOS 和 Android 應用程式的中轉回覆 URI:

1. 在應用程式註冊門戶中,選擇 **「身份驗證** > **試用」 新體驗**。

   ![認證邊列選項卡, 您可以在其中選擇新體驗](https://user-images.githubusercontent.com/13203188/60799285-2d031b00-a173-11e9-9d28-ac07a7ae894a.png)

2. 選擇 **「添加平臺**」。

   ![新增平臺](https://user-images.githubusercontent.com/13203188/60799366-4c01ad00-a173-11e9-934f-f02e26c9429e.png)

3. 當支援平台清單時,請選擇**iOS**。

   ![選擇移動應用程式](https://user-images.githubusercontent.com/13203188/60799411-60de4080-a173-11e9-9dcc-d39a45826d42.png)

4. 輸入捆綁包 ID,然後選擇 **「註冊**」。

   ![輸入捆綁包識別碼](https://user-images.githubusercontent.com/13203188/60799477-7eaba580-a173-11e9-9f8b-431f5b09344e.png)

完成這些步驟後,將為您計算重定向URI,如下圖所示。

![產生的重定向 URI](https://user-images.githubusercontent.com/13203188/60799538-9e42ce00-a173-11e9-860a-015a1840fd19.png)

如果您希望手動配置重定向URI,可以通過應用程式清單執行此操作。 以下是清單的建議格式:

- **iOS**:`msauth.<BUNDLE_ID>://auth` 
  - 例如，輸入 `msauth.com.yourcompany.appName://auth`
- **安卓**系統 :`msauth://<PACKAGE_NAME>/<SIGNATURE_HASH>`
  - 您可以通過 KeyTool 命令使用釋放金鑰或調試密鑰生成 Android 簽名哈希。

### <a name="username-password-authentication"></a>使用者名稱密碼身份驗證

如果應用僅使用使用者名和密碼身份驗證,則無需為應用程式註冊重定向 URI。 此流對 Microsoft 標識平臺版本 2.0 終結點進行往返訪問。 您的應用程式不會在任何特定 URI 上被呼叫。 

但是,您需要將應用程式標識為公共用戶端應用程式。 為此,請從應用程式的**身份驗證**部分開始。 在 **「進階設定**」子節中,在 **「預設型態類型」** 段落中,對於問題 **,將應用程式視為公共用戶端**,請選擇「**是**」。 。

## <a name="api-permissions"></a>API 權限

移動應用程式代表登錄使用者調用 API。 你的應用需要請求委派的許可權。 這些許可權也稱為作用域。 根據所需的體驗,可以通過 Azure 門戶靜態請求委派的許可權。 或者,您可以在運行時動態請求它們。 

通過靜態註冊許可權,允許管理員輕鬆批准你的應用。 建議進行靜態註冊。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [程式碼設定](scenario-mobile-app-configuration.md)

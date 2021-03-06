---
title: 用於自訂開發應用程式的 Azure 入口註冊欄位
description: 使用 Azure AD 註冊自訂開發的應用程式的指南
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 06/28/2019
ms.author: ryanwi
ms.openlocfilehash: c44575ca43063388d5c65855542cf15700d2cb5a
ms.sourcegitcommit: d187fe0143d7dbaf8d775150453bd3c188087411
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80883163"
---
# <a name="azure-portal-registration-fields-for-custom-developed-apps"></a>用於自訂開發應用程式的 Azure 入口註冊欄位

本文提供 [Azure 入口網站](https://portal.azure.com)上應用程式註冊表單中所有可用欄位的簡短說明。

## <a name="register-a-new-application"></a>註冊新的應用程式

-   若要註冊新的應用程式，請瀏覽至 [Azure 入口網站](https://portal.azure.com)。

-   在左側導航窗格中,按一下**Azure 活動目錄。**

-   選擇 [應用程式註冊]****，然後按一下 [新增]****。

-   這樣會開啟應用程式註冊表單。

## <a name="fields-in-the-application-registration-form"></a>應用程式註冊表單中的欄位

| 欄位            | 描述                                                                              |
|------------------|------------------------------------------------------------------------------------------|
| 名稱             | 應用程式的名稱。 至少應該有四個字元。                |
| 支援的帳戶類型| 選擇您希望應用程式支援的帳戶:僅此組織目錄中的帳戶、任何組織目錄中的帳戶或任何組織目錄中的帳戶以及個人 Microsoft 帳戶中的帳戶。  |
| 重定向 URI(選擇性的) | 選擇要建構的應用類型 **、Web**或**公共用戶端(移動&桌面),** 然後輸入應用程式的重定向 URI(或回覆 URL)。 若為 Web 應用程式，請提供應用程式的基底 URL。 例如，http://localhost:31544 可能是在您的本機電腦上執行之 Web 應用程式的 URL。 使用者會使用此 URL 來登入 Web 用戶端應用程式。 若為公用用戶端應用程式，請提供 Azure AD 用來傳回權杖回應的 URI。 輸入特定於應用程式的值,如myapp://auth。要檢視 Web 應用程式或本機應用程式的具體範例,請查看我們的[快速入門](https://docs.microsoft.com/azure/active-directory/develop)。|

填充上述欄位後,應用程式將註冊在 Azure 門戶中,並重定向到應用程式概述頁。 **"管理'** 下左方窗格中的設定頁具有更多欄位可供您自訂應用程式。 下表描述了所有欄位。 您只能看到這些欄位的子集,具體取決於您是創建了 Web 應用程式還是公共用戶端應用程式。

### <a name="overview"></a>概觀

| 欄位           | 描述        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 應用程式識別碼  | 當您註冊應用程式時，Azure AD 會將應用程式識別碼指派給您的應用程式。 應用程式識別碼可在對 Azure AD 的驗證要求中用來唯一識別您的應用程式，還可用來存取資源 (例如圖形 API)。                                                          |
| 應用程式識別碼 URI      | 這應該是一個唯一的 URI,通常**&lt;HTTPs://租&gt;/&lt;\_戶 名稱應用程式\_名稱&gt;** 的窗體。 這在授權授予流期間使用,作為指定令牌應為其頒發的資源的唯一標識符。 它在核發的存取權杖中也會變成 'aud' 宣告。 |

### <a name="branding"></a>商標

| 欄位           | 描述        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 上傳新標誌 | 您可以使用此欄位上傳應用程式的標誌。 徽標必須採用 .bmp、.jpg 或 .png 格式,並且檔大小應小於 100 KB。 影像尺寸應該是 215 x 215 像素，而中央影像尺寸為 94 x 94 像素。|
| 首頁 URL   | 這是在應用程式註冊期間指定的登入 URL。|

### <a name="authentication"></a>驗證

| 欄位           | 描述        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 登出 URL      | 這是單個註銷註銷 URL。 當使用者透過 Azure AD 使用任何其他已註冊的應用程式來清除他們的工作階段時，Azure AD 會傳送登出要求給這個 URL。|
| 支援的帳戶類型  | 這個參數指定應用程式是否可供多重租用戶使用。 一般而言，這表示外部組織可以在其租用戶中註冊應用程式，並授權存取其組織的資料，以使用您的應用程式。|
| 重新導向 URL      | 重定向或答覆 URL 是 Azure AD 返回應用程式請求的任何權杖的終結點。 針對原生應用程式，會在成功授權之後將使用者傳送到此 URI。 Azure AD 檢查應用程式在 OAuth 2.0 請求中的重定向 URI 是否與門戶中的已註冊值之一匹配。|

### <a name="certificates-and-secrets"></a>憑證和機密

| 欄位           | 描述        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 用戶端密碼            | 您可以建立用戶端機密或密鑰,以程式設計方式存取 Azure AD 保護的 Web API,而無需任何使用者互動。 在 **"新建用戶端"金鑰**頁中,輸入金鑰說明和到期日期,並保存以生成密鑰。 務必將它儲存在安全的位置，因為您稍後將無法存取它。             |

## <a name="next-steps"></a>後續步驟

[使用 Azure Active Directory 來管理應用程式](../manage-apps/what-is-application-management.md)

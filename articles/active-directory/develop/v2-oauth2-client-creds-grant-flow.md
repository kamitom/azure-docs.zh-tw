---
title: OAuth 2.0 客戶端認證串流在 Microsoft 標識平臺上 |蔚藍
description: 使用 OAuth 2.0 身份驗證協定的 Microsoft 識別平台實現建置 Web 應用程式。
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: a03f8cb412b6d6ae95165331ae836bdfde5d670d
ms.sourcegitcommit: d187fe0143d7dbaf8d775150453bd3c188087411
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80886291"
---
# <a name="microsoft-identity-platform-and-the-oauth-20-client-credentials-flow"></a>微軟識別平臺和 OAuth 2.0 客戶端認證串流

您可以使用 RFC 6749 中指定的 [OAuth 2.0 用戶端認證授與](https://tools.ietf.org/html/rfc6749#section-4.4) (有時稱為「雙方 OAuth」**，透過使用應用程式識別碼來存取 Web 主控資源。 這類型的授與通常用於必須在背景中執行 (不需與使用者直接互動) 的伺服器對伺服器互動。 這些類型的應用程式通常稱為*精靈*或*服務帳戶*。

本文介紹如何直接針對應用程式中的協議進行程式設計。 如果可能,我們建議您使用受支援的 Microsoft 身份驗證庫 (MSAL) 來[獲取權杖並調用安全的 Web API。](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows)  也看看[使用MSAL的範例應用程式](sample-v2-code.md)。

OAuth 2.0 用戶端認證授與流程可允許 Web 服務 (機密用戶端) 在呼叫另一個 Web 服務時，使用它自己的認證來進行驗證，而不是模擬使用者。 在此案例中，用戶端通常是中介層 Web 服務、精靈服務或網站。 對於較高層級的保證，Microsoft 身分識別平台也可讓呼叫服務使用憑證 (而非共用密碼) 做為認證。

> [!NOTE]
> Microsoft 識別平台終結點不支援所有 Azure AD 方案和功能。 要確定是否應使用 Microsoft 識別平台終結點,請閱讀有關[Microsoft 識別平臺限制](active-directory-v2-limitations.md)。

在較為典型的「三腳 OAuth」** 中，用戶端應用程式會獲得代表特定使用者存取資源的權限。 此權限是由使用者 委派 給應用程式 (通常在 [同意](v2-permissions-and-consent.md) 過程中)。 不過，在用戶端認證 (*二方 OAuth*) 流程中，權限會直接授與應用程式本身。 當應用程式向資源出示權杖時，資源會強制要求應用程式本身具備執行動作的授權，而不是使用者。

## <a name="protocol-diagram"></a>通訊協定圖表

整個用戶端認證流程看起來類似下圖。 我們會在本文稍後說明每個步驟。

![顯示客戶端認證串流的圖表](./media/v2-oauth2-client-creds-grant-flow/convergence-scenarios-client-creds.svg)

## <a name="get-direct-authorization"></a>取得直接授權

應用程式通常會收到直接授權，而可透過下列兩種方式之一存取資源：

* [在資源中透過存取控制清單 (ACL)](#access-control-lists)
* [在 Azure AD 中透過應用程式權限指派](#application-permissions)

這兩種方法是 Azure AD 中最常見的方法，並建議用於執行用戶端認證流程的用戶端和資源。 資源也可以選擇以其他方法授權其用戶端。 每個資源伺服器都可以選擇最適合其應用程式的方法。

### <a name="access-control-lists"></a>存取控制清單

資源提供者可能會根據它所知的應用程式 (用戶端) 識別碼清單，強制執行授權檢查並授與特定層級的存取權。 當資源從 Microsoft 標識平台終結點接收權杖時,它可以解碼權杖`appid`並`iss`從和聲明中提取用戶端的應用程式 ID。 然後，它會將應用程式與它所保有的存取控制清單 (ACL) 進行比較。 各資源之間的 ACL 細微性及方法可能有極大的差異。

常見的使用案例是使用 ACL 來針對 Web 應用程式或 Web API 執行測試。 Web API 可能只會將完整權限中的一部分授與特定用戶端。 要在 API 上執行端到端測試,請建立一個測試用戶端,從 Microsoft 標識平台終結點獲取權杖,然後將它們發送到 API。 接著，API 會檢查 ACL 是否有測試用戶端的應用程式識別碼，以便提供 API 完整功能的完整存取權。 如果您使用這類 ACL，請務必不要只驗證呼叫者的 `appid` 值，還要驗證權杖的 `iss` 值是否可信任。

對於需要存取具有個人 Microsoft 帳戶之取用者使用者所擁有資料的精靈和服務帳戶來說，這種授權相當常見。 對於組織所擁有的資料，建議您透過應用程式權限取得必要的授權。

### <a name="application-permissions"></a>應用程式權限

您可以使用 API 來公開一組**應用程式許可權**,而不是使用 ACL。 應用程式權限是由組織的系統管理員授與應用程式，並且只能用來存取該組織及其員工所擁有的資料。 例如，Microsoft Graph 會公開數個可執行下列操作的應用程式權限︰

* 讀取所有信箱中的郵件
* 讀取和寫入所有信箱中的郵件
* 以任何使用者身分傳送郵件
* 讀取目錄資料

如需有關應用程式權限的詳細資訊，請參閱 [Microsoft Graph](https://developer.microsoft.com/graph)。

若要在您的應用程式中使用應用程式權限，請遵循後續小節中討論的步驟。


> [!NOTE]
> 當作為應用程式進行身份驗證時,而不是使用使用者時,不能使用"委派許可權"(由使用者授予的範圍)。  您必須使用"應用程式許可權",也稱為"角色",這些許可權由管理員授予應用程式(或通過 Web API 的預授權)。    


#### <a name="request-the-permissions-in-the-app-registration-portal"></a>在應用程式註冊入口網站中要求權限

1. 通過新的[應用程式註冊(預覽)體驗](quickstart-register-app.md)註冊和創建應用。
2. 在應用註冊(預覽)體驗中轉到您的應用程式。 導航到**證書&機密**部分,並添加新**的用戶端機密**,因為至少需要一個用戶端密鑰來請求令牌。
3. 找出 [API 權限]**** 區段，然後新增您應用程式所需的**應用程式權限**。
4. [儲存]**** 應用程式註冊。

#### <a name="recommended-sign-the-user-into-your-app"></a>推薦:將使用者登錄到你的應用

通常，當您建置使用應用程式權限的應用程式時，應用程式會需要一個可供系統管理員核准應用程式權限的頁面或檢視。 此頁面可以是應用程式登入流程的一部分、應用程式設定的一部分，或是專用的「連接」流程。 在許多情況下，應用程式只在使用者利用工作或學校 Microsoft 帳戶登入之後顯示此「連接」檢視是很合理的。

如果將使用者登錄到應用,則可以在要求使用者批准應用程式許可權之前標識使用者所屬的組織。 雖然這並非絕對必要，但這麼做可協助您為使用者建立更直覺式的體驗。 要登錄使用者,請按照我們的[Microsoft 身份平台協定教程](active-directory-v2-protocols.md)進行操作。

#### <a name="request-the-permissions-from-a-directory-admin"></a>向目錄管理員要求權限

當您準備好向組織的管理員請求許可權時,可以將使用者重定向到 Microsoft 標識平臺*管理員同意終結點*。

> [!TIP]
> 嘗試在 Postman 中執行這項要求！ (使用您自己的應用 ID 獲得最佳結果 - 教程應用程式不會請求有用的許可權。[試著在郵遞員中執行此![要求](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro tip: Try pasting the following request in a browser.
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| 參數 | 條件 | 描述 |
| --- | --- | --- |
| `tenant` | 必要 | 您想要要求權限的目錄租用戶。 這可以採用 GUID 或易記名稱格式。 如果您不知道使用者屬於哪個租用戶，而想要讓他們以任何租用戶登入，請使用 `common`。 |
| `client_id` | 必要 | [Azure 門戶和應用註冊](https://go.microsoft.com/fwlink/?linkid=2083908)體驗**的應用程式(用戶端)ID**分配給應用。 |
| `redirect_uri` | 必要 | 您想要傳送回應以供應用程式處理的重新導向 URI。 它必須與您在入口網站中註冊的其中一個重新導向 URI 完全相符，只是它必須是採用 URL 編碼，並且可以有額外的路徑區段。 |
| `state` | 建議 | 要求中包含的值，也會隨權杖回應傳回。 它可以是您所想要內容中的字串。 此狀態用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |

此時,Azure AD 強制只有租戶管理員才能登錄到完成請求。 系統會請系統管理員核准您在應用程式註冊入口網站中，為您應用程式要求的所有直接應用程式權限。

##### <a name="successful-response"></a>成功回應

如果系統管理員為您的應用程式核准權限，則成功的回應看起來會像這樣︰

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| 參數 | 描述 |
| --- | --- |
| `tenant` | 將應用程式所要求的權限授與應用程式的目錄租用戶 (採用 GUID 格式)。 |
| `state` | 一個包含在要求中而將一併在權杖回應中傳回的值。 它可以是您所想要內容中的字串。 此狀態用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |
| `admin_consent` | 設定為 [True]****。 |

##### <a name="error-response"></a>錯誤回應

如果系統管理員沒有為您的應用程式核准權限，則失敗的回應看起來會像這樣︰

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| 參數 | 描述 |
| --- | --- |
| `error` | 您可用來分類錯誤類型並對錯誤做出反應的錯誤碼字串。 |
| `error_description` | 可協助您識別錯誤根本原因的特定錯誤訊息。 |

從應用程式佈建端點收到成功回應後，您的應用程式便已取得它所要求的直接應用程式權限。 現在，您可以針對您想要的資源要求權杖。

## <a name="get-a-token"></a>取得權杖

取得應用程式的必要授權後，請繼續取得 API 的存取權杖。 要使用客戶端認證取得權杖,請向`/token`Microsoft 識別平台終結點傳送 POST 請求:

> [!TIP]
> 嘗試在 Postman 中執行這項要求！ (使用您自己的應用 ID 獲得最佳結果 - 教程應用程式不會請求有用的許可權。[試著在郵遞員中執行此![要求](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

### <a name="first-case-access-token-request-with-a-shared-secret"></a>第一種情況︰使用共用密碼的存取權杖要求

```
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

```
// Replace {tenant} with your tenant! 
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token'
```

| 參數 | 條件 | 描述 |
| --- | --- | --- |
| `tenant` | 必要 | 應用程式預期要對其執行作業的目錄租用戶 (以 GUID 或網域名稱格式)。 |
| `client_id` | 必要 | 指派給應用程式的應用程式識別碼。 您可以在用來註冊應用程式的入口網站中找到這項資訊。 |
| `scope` | 必要 | 在這個要求中針對 `scope` 參數傳遞的值應該是您所要資源的資源識別碼 (應用程式識別碼 URI)，並附加 `.default` 尾碼。 就 Microsoft Graph 範例而言，值為 `https://graph.microsoft.com/.default`。 <br/>此值告訴 Microsoft 識別平台終結點,在已為應用配置的所有直接應用程式許可權中,終結點應為與要使用的資源關聯的許可權頒發權杖。 若要深入了解有關 `/.default` 範圍，請參閱[同意文件](v2-permissions-and-consent.md#the-default-scope)。 |
| `client_secret` | 必要 | 在應用註冊門戶中為應用生成的客戶端密鑰。 用戶端密碼必須在傳送之前先進行 URL 編碼。 |
| `grant_type` | 必要 | 必須設為 `client_credentials`。 |

### <a name="second-case-access-token-request-with-a-certificate"></a>第二種情況︰使用憑證的存取權杖要求

```
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

| 參數 | 條件 | 描述 |
| --- | --- | --- |
| `tenant` | 必要 | 應用程式預期要對其執行作業的目錄租用戶 (以 GUID 或網域名稱格式)。 |
| `client_id` | 必要 |指派給應用程式的應用程式 (用戶端) 識別碼。 |
| `scope` | 必要 | 在這個要求中針對 `scope` 參數傳遞的值應該是您所要資源的資源識別碼 (應用程式識別碼 URI)，並附加 `.default` 尾碼。 就 Microsoft Graph 範例而言，值為 `https://graph.microsoft.com/.default`。 <br/>此值通知 Microsoft 識別平台終結點,即您為應用配置的所有直接應用程式許可權中,它應為與要使用的資源關聯的許可權頒發權杖。 若要深入了解有關 `/.default` 範圍，請參閱[同意文件](v2-permissions-and-consent.md#the-default-scope)。 |
| `client_assertion_type` | 必要 | 此值必須設定為 `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`。 |
| `client_assertion` | 必要 | 您必須建立判斷提示 (JSON Web 權杖)，並使用註冊的憑證來簽署，以作為應用程式的認證。 請參閱[憑證認證](active-directory-certificate-credentials.md)，以了解如何註冊您的憑證與判斷提示的格式。|
| `grant_type` | 必要 | 必須設為 `client_credentials`。 |

請注意，在透過共用密碼要求的情況中，參數幾乎相同，不同之處在於使用下列兩個參數來取代 client_secret 參數：client_assertion_type 和 client_assertion。

### <a name="successful-response"></a>成功回應

成功的回應看起來會像這樣：

```
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
}
```

| 參數 | 描述 |
| --- | --- |
| `access_token` | 所要求的存取權杖。 應用程式可以使用這個權杖向受保護的資源 (例如 Web API) 進行驗證。 |
| `token_type` | 表示權杖類型值。 Microsoft 識別平台支援的唯一類型`bearer`是 。 |
| `expires_in` | 存取權杖的有效時間長度 (以秒為單位)。 |

### <a name="error-response"></a>錯誤回應

錯誤回應看起來像這樣：

```
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/.default is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| 參數 | 描述 |
| --- | --- |
| `error` | 您可用來分類發生的錯誤類型並對錯誤做出反應的錯誤碼字串。 |
| `error_description` | 可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |
| `error_codes` | 可協助進行診斷的 STS 特定錯誤碼清單。 |
| `timestamp` | 錯誤發生時間。 |
| `trace_id` | 可協助進行診斷的要求唯一識別碼。 |
| `correlation_id` | 可協助跨元件進行診斷的要求唯一識別碼。 |

## <a name="use-a-token"></a>使用權杖

既然您已取得權杖，請使用該權杖來對資源提出要求。 當權杖到期時，只要重複執行對 `/token` 端點的要求，即可取得全新的存取權杖。

```
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

```
// Pro tip: Try the following command! (Replace the token with your own.)
```

```
curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbG...." 'https://graph.microsoft.com/v1.0/me/messages'
```

## <a name="code-samples-and-other-documentation"></a>程式碼範例和其他文件

讀取 Microsoft 驗證程式庫中的[用戶端認證概觀文件](https://aka.ms/msal-net-client-credentials)

| 範例 | 平台 |描述 |
|--------|----------|------------|
|[active-directory-dotnetcore-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) | .NET core 2.1 主控台 | 一個簡單的.NET Core 應用程式，它會顯示使用應用程式的身分識別查詢 Microsoft Graph 之租用戶的使用者，而不是代表使用者。 該範例也會說明使用憑證進行驗證的變化。 |
|[active-directory-dotnet-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)|ASP.NET MVC | 一個 Web 應用程式，它使用應用程式的身分識別同步處理 Microsoft Graph 中的資料，而不是代表使用者。 |

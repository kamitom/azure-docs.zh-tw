---
title: Microsoft Graph API
description: Microsoft 圖形 API 是一種 RESTful Web API,使您能夠存取 Microsoft 雲服務資源。
author: davidmu1
services: active-directory
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/13/2020
ms.author: davidmu
ms.custom: aaddev
ms.openlocfilehash: 67dbf696903e7a930d75762deb00ad58ed1a4f69
ms.sourcegitcommit: d187fe0143d7dbaf8d775150453bd3c188087411
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80886461"
---
# <a name="microsoft-graph-api"></a>Microsoft Graph API

Microsoft 圖形 API 是一種 RESTful Web API,使您能夠存取 Microsoft 雲服務資源。 註冊應用並獲取使用者或服務的身份驗證權杖後,可以向Microsoft圖形API發出請求。 有關詳細資訊,請參閱[微軟圖形概述](https://docs.microsoft.com/graph/overview)。

Microsoft Graph 公開 REST API 和用戶端庫以造訪以下 Microsoft 365 服務的資料:
- Office 365 服務:Delve、Excel、微軟預訂、微軟團隊、OneDrive、OneNote、Outlook/Exchange、規劃器和 SharePoint
- 企業移動和安全服務:高級威脅分析、高級威脅防護、Azure 活動目錄、標識管理器和 Intune
- Windows 10 服務:活動、裝置、通知
- Dynamics 365 Business Central

## <a name="versions"></a>版本

Microsoft Graph 目前支援兩個版本:v1.0 和 beta。 v1.0 版本包括一般可用的API。 對所有生產應用使用 v1.0 版本。 測試版包括當前處於預覽狀態的 API。 由於我們可能會對 Beta API 引入重大更改,因此建議您僅使用測試版來測試正在開發中的應用;請勿在生產應用中使用 Beta API。 有關詳細資訊,請參閱[Microsoft 圖形的版本控制、支援和中斷更改策略](https://docs.microsoft.com/graph/versioning-and-support)。

要開始使用測試版 API,請參閱[Microsoft 圖形測試版終結點參考](https://docs.microsoft.com/graph/api/overview?view=graph-rest-beta)

要開始使用 v1.0 API,請參閱[Microsoft 圖形 REST API v1.0 引用](https://docs.microsoft.com/graph/api/overview?view=graph-rest-1.0)

## <a name="get-started"></a>開始使用

要讀取或寫入資源(如使用者或電子郵件),請建構如下所示的請求:

`{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}`

有關建構請求元素的詳細資訊,請參閱[使用 Microsoft 圖形 API](https://docs.microsoft.com/graph/use-the-api)

快速入門示例可用於向您展示如何存取 Microsoft 圖形 API 的強大功能。 可用的範例存取兩個服務,其中一個身份驗證:Microsoft 帳戶和 Outlook。 每個快速入門都會從 Microsoft 帳戶使用者的配置檔中訪問資訊,並顯示其日曆中的事件。
快速入門包括四個步驟:
- 選擇您的平台
- 取得應用程式 ID(用戶端 ID)
- 建置範例
- 登入並檢視行事曆上的事件

完成快速入門後,您有一個可以運行的應用。 有關詳細資訊,請參閱 Microsoft[圖形快速入門常見問題解答](https://docs.microsoft.com/graph/quick-start-faq)。 要開始使用範例,請參閱[Microsoft 圖形快速入門](https://developer.microsoft.com/graph/quick-start)。

## <a name="tools"></a>工具

Microsoft 圖形資源管理員是一個基於 Web 的工具,您可以使用它使用 Microsoft 圖形 API 生成和測試請求。 您可以在以下處存取 Microsoft`https://developer.microsoft.com/graph/graph-explorer`圖形 資源管理員。

Postman 是一種工具,您還可以使用 Microsoft 圖形 API 生成和測試請求。 您可以在以下位置下載郵遞員`https://www.getpostman.com/`。 要在 Postman 中與 Microsoft 圖形進行互動,請使用 Postman 中的 Microsoft 圖形集合。 有關詳細資訊,請參閱使用[Postman 與 Microsoft 圖形 API](/graph/use-postman?context=graph%2Fapi%2Fbeta&view=graph-rest-beta)。

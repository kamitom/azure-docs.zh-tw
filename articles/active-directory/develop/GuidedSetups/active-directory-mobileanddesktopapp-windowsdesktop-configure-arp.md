---
title: Azure AD v2 Windows 桌面入門 ─設定
description: Windows Desktop .NET (XAML) 應用程式可取得存取權杖，以及呼叫受 Azure Active Directory v2 端點保護之 API 的方法。
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/29/2020
ms.author: ryanwi
ms.custom: aaddev
ms.openlocfilehash: d82f9beecb1b558fca094c31f8c6718c990debd1
ms.sourcegitcommit: d187fe0143d7dbaf8d775150453bd3c188087411
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80885764"
---
# <a name="add-the-applications-registration-information-to-your-app"></a>將應用程式的註冊資訊新增到您的應用程式
在這個步驟中，您需要將應用程式識別碼新增到您的專案。

1.  開啟 `App.xaml.cs` 並將包含 `ClientId` 的那一行取代為：

```csharp
private static string ClientId = "[Enter the application Id here]";
```

### <a name="what-is-next"></a>下一步

[!INCLUDE [Test and Validate](../../../../includes/active-directory-develop-guidedsetup-windesktop-test.md)]

---
title: 使用 C 為 Azure 資料資源管理器添加群集主體#
description: 在本文中，您將瞭解如何使用 C# 為 Azure 資料資源管理器添加群集主體。
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: e6c3970890dfe2c669dee1acf631e9dd45ab1085
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76965056"
---
# <a name="add-cluster-principals-for-azure-data-explorer-by-using-c"></a>使用 C 為 Azure 資料資源管理器添加群集主體#

> [!div class="op_single_selector"]
> * [C#](cluster-principal-csharp.md)
> * [Python](cluster-principal-python.md)
> * [Azure 資源管理器範本](cluster-principal-resource-manager.md)

Azure 資料總管是一項快速又可高度調整的資料探索服務，可用於處理記錄和遙測資料。 在本文中，可以使用 C# 為 Azure 資料資源管理器添加群集主體。

## <a name="prerequisites"></a>Prerequisites

* 如果尚未安裝 Visual Studio 2019，您可以下載並使用**免費的** [Visual Studio 2019 Community 版本](https://www.visualstudio.com/downloads/)。 請確保在視覺化工作室設置期間啟用**Azure 開發**。
* 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費 Azure 帳戶](https://azure.microsoft.com/free/)。
* [創建群集](create-cluster-database-csharp.md)。

## <a name="install-c-nuget"></a>安裝 C# NuGet

* 安裝[Microsoft.Azure.管理.kusto](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)。
* 安裝[Microsoft.Rest.用戶端運行時.Azure.](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication)身份驗證。

[!INCLUDE [data-explorer-authentication](../../includes/data-explorer-authentication.md)]

## <a name="add-a-cluster-principal"></a>添加群集主體

下面的示例演示如何以程式設計方式添加群集主體。

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";

var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, clientSecret);
var kustoManagementClient = new KustoManagementClient(serviceCreds)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster that is created as part of the Prerequisites
var clusterName = "mykustocluster";
string principalAssignmentName = "clusterPrincipalAssignment1";
string principalId = "xxxxxxxx";//User email, application ID, or security group name
string role = "AllDatabasesAdmin";//AllDatabasesAdmin or AllDatabasesViewer
string tenantIdForPrincipal = tenantId;
string principalType = "App";//User, App, or Group

var clusterPrincipalAssignment = new ClusterPrincipalAssignment(principalId, role, principalType, tenantId: tenantIdForPrincipal);
await kustoManagementClient.ClusterPrincipalAssignments.CreateOrUpdateAsync(resourceGroupName, clusterName, principalAssignmentName, clusterPrincipalAssignment);
```

|**設定** | **建議的值** | **欄位描述**|
|---|---|---|
| tenantId | *xxxxxx-xxxxxx-xxx-xxx-xxxxxxxxx* | 您的租用戶識別碼。 也稱為目錄 ID。|
| subscriptionId | *xxxxxx-xxxxxx-xxx-xxx-xxxxxxxxx* | 用於資源創建的訂閱 ID。|
| clientId | *xxxxxx-xxxxxx-xxx-xxx-xxxxxxxxx* | 可以訪問租戶中資源的應用程式的用戶端 ID。|
| clientSecret | *xxxxxxxxxxxxxxx* | 可以訪問租戶中資源的應用程式的用戶端機密。 |
| resourceGroupName | *testrg* | 包含群集的資源組的名稱。|
| clusterName | *mykustocluster* | 群集的名稱。|
| 主體分配名稱 | *群集主體分配1* | 群集主體資源的名稱。|
| principalId | *xxxxxx-xxxxxx-xxx-xxx-xxxxxxxxx* | 主體 ID，可以是使用者電子郵件、應用程式 ID 或安全性群組名稱。|
| 角色 (role) | *所有資料庫管理* | 群集主體的角色，可以是"所有資料庫管理"或"所有資料庫檢視器"。|
| 租戶 Idfor 主要 | *xxxxxx-xxxxxx-xxx-xxx-xxxxxxxxx* | 主體的租戶 ID。|
| principalType | *應用程式* | 主體的類型，可以是"使用者"、"應用"或"組"|

## <a name="next-steps"></a>後續步驟

* [添加資料庫主體](database-principal-csharp.md)

---
title: 此 PowerShell 指令碼用以更新 Azure Cosmos 帳戶的區域
description: Azure PowerShell 指令碼範例 - 更新 Azure Cosmos 帳戶的區域
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 03/21/2020
ms.author: mjbrown
ms.openlocfilehash: 4a8e24b4445e2bf29b35e87600de260c523d4376
ms.sourcegitcommit: 07d62796de0d1f9c0fa14bfcc425f852fdb08fb1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "80367171"
---
# <a name="update-an-azure-cosmos-accounts-regions-using-powershell"></a>使用 PowerShell 更新 Azure Cosmos 帳戶的區域

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>範例指令碼

> [!NOTE]
> 您不能在相同的作業中修改區域和變更其他 Cosmos 帳戶屬性。 這些必須以兩個不同的作業來完成。
> [!NOTE]
> 此範例會示範如何使用 SQL (核心) API 帳戶。 若要針對其他 API 使用此範例，請複製相關的屬性，並套用至您的 API 專屬指令碼。

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/common/ps-account-update-region.ps1 "Update Azure Cosmos account regions")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
|**Azure 資源**| |
| [Set-AzResource](https://docs.microsoft.com/powershell/module/az.resources/set-azresource) | 更新資源。 |
|**Azure 資源群組**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |
|||

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure Cosmos DB PowerShell 指令碼](../../../powershell-samples.md)中找到其他 Azure Cosmos DB PowerShell 指令碼範例。

---
title: Azure PowerShell 範例 - 單一區域擴展集
description: 此指令碼會建立在單一可用性區域中執行 Windows Server 2016 的虛擬機器擴展集。
author: cynthn
tags: azure-resource-manager
ms.service: virtual-machine-scale-sets
ms.topic: sample
ms.date: 04/05/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: a9b3547844d0b2dcca58a95d04c9cb81d32fa52c
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/24/2020
ms.locfileid: "76276422"
---
# <a name="create-a-single-zone-virtual-machine-scale-set-with-powershell"></a>使用 PowerShell 建立單一區域虛擬機器擴展集
此指令碼會建立在單一可用性區域中執行 Windows Server 2016 的虛擬機器擴展集。 執行指令碼之後，您可以透過 RDP 存取虛擬機器。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/create-single-availability-zone/create-single-availability-zone.ps1 "Create single-zone scale set")]

## <a name="clean-up-deployment"></a>清除部署
執行下列命令來移除資源群組、擴展集和所有相關資源。

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
| [New-AzVmss](/powershell/module/az.compute/new-azvmss) | 建立虛擬機器擴展集和所有支援的資源，包括虛擬網路、負載平衡器和 NAT 規則。 |
| [Get-AzVmss](/powershell/module/az.compute/get-azvmss) | 取得虛擬機器擴展集的相關資訊。 |
| [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) | 新增自訂指令碼的 VM 擴充來安裝基本 Web 應用程式。 |
| [Update-AzVmss](/powershell/module/az.compute/update-azvmss) | 更新虛擬機器擴展集模型以套用 VM 擴充。 |
| [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) | 取得指派給負載平衡器使用的公用 IP 位址資訊。 |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | 移除資源群組及其內含的所有資源。 |

## <a name="next-steps"></a>後續步驟
如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure 虛擬機器擴展集文件](../powershell-samples.md)中找到其他的虛擬機器擴展集 PowerShell 指令碼範例。

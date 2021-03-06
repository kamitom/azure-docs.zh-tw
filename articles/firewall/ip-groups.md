---
title: Azure 防火牆中的 IP 群組
description: IP 組允許您對 Azure 防火牆規則的 IP 位址進行分組和管理。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: victorh
ms.openlocfilehash: e0638cbccd5e3bc282dbdd7d3b5918e29081a12b
ms.sourcegitcommit: 441db70765ff9042db87c60f4aa3c51df2afae2d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2020
ms.locfileid: "80757156"
---
# <a name="ip-groups-preview-in-azure-firewall"></a>Azure 防火牆中的 IP 群組(預覽)

> [!IMPORTANT]
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

IP 群組允許您透過以下方式對 Azure 防火牆規則的 IP 位址進行群組和管理:

- 為 DNAT 規則中的來源位址
- 為網路規則中的來源地址或目標位址
- 應用程式規則中的來源位址


IP 組可以具有單個 IP 位址、多個 IP 位址或一個或多個 IP 位址範圍。

IP 組可以在 Azure 防火牆 DNAT、網路和應用程式規則中重複使用,用於跨 Azure 區域和訂閱的多個防火牆。 組名稱必須是唯一的。 您可以在 Azure 門戶、Azure CLI 或 REST API 中設定 IP 組。 提供了一個範例範本,以説明您入門。

## <a name="sample-format"></a>樣本格式

以下 IPv4 位址格式範例在 IP 群組中有效:

- 單位址: 10.0.0.0
- CIDR 表示法: 10.1.0.0/32
- 位址範圍: 10.2.0.0-10.2.0.31

## <a name="create-an-ip-group"></a>建立 IP 群組

可以使用 Azure 門戶、Azure CLI 或 REST API 創建 IP 組。 有關詳細資訊,請參閱創建[IP 組(預覽)。](create-ip-group.md)

## <a name="browse-ip-groups"></a>瀏覽 IP 群組
1. 在 Azure 門戶搜索欄中,鍵入**IP 組**並選擇它。 您可以看到 IP 群組的清單,也可以選擇 **「新增」** 以建立新的 IP 組。
2. 選擇 IP 組以打開概覽頁面。 您可以編輯、添加或刪除 IP 位址或 IP 群組。

   ![IP 組概述](media/ip-groups/overview.png)

## <a name="manage-an-ip-group"></a>管理 IP 群組

您可以看到 IP 群組中的所有 IP 位址及其關聯的規則或資源。 要刪除 IP 組,必須首先將 IP 群組與使用它的資源分離。

1. 要檢視或編輯 IP 位址,請在左邊窗格的**設定「** 下選擇**IP 位址**。
2. 要新增單個或多個 IP 位址,請選擇 **「添加 IP 位址**」 。 這將打開「**拖動」 或「瀏覽」** 頁面進行上傳,或者您可以手動輸入位址。
3.    選擇編輯或移除 IP 位址右邊的省略號 (**...** 要編輯或刪除多個 IP 位址,請選擇框,然後選擇頂部的 **「編輯**」或 **「刪除**」 。
4. 最後,可以匯出 CSV 檔案格式的檔案。

> [!NOTE]
> 如果在 IP 群組中仍在使用規則時刪除 IP 群組中的所有 IP 位址,則跳過該規則。


## <a name="use-an-ip-group"></a>使用 IP 群組

現在,在建立 Azure 防火牆 DNAT、應用程式或網路規則時,可以選擇**IP 群組**作為 IP 位址的**來源類型**或**目標類型**。

> [!NOTE]
> 防火牆策略中不支援 IP 組。 它目前僅受傳統防火牆規則的支援。

![防火牆中的 IP 群組](media/ip-groups/fw-ipgroup.png)

## <a name="region-availability"></a>區域可用性

IP 組在所有公共雲區域都可用。

## <a name="ip-address-limits"></a>IP 位址限制

對於 50 個或更少的 IP 組,每個防火牆實例最多可以有 5000 個單獨的 IP 位址。 對於 51 到 100 個 IP 組,每個防火牆實例可以有 500 個單獨的 IP 位址。

### <a name="examples"></a>範例

#### <a name="example-1-supported"></a>範例 1:受支援

|IP 群組  |• IP 位址  |表示法  |規則  |
|---------|---------|---------|---------|
|IPGroup1 |4096     |10.0.0.0/20  |Rule1|
|IPGroup2     |3|196.0.0.0 - 196.0.0.2|Rule1|
|IPGroup3     |1|1.2.3.4|Rule1|
|     |**總計 4100**|         |         |
|     |         |         |         |

#### <a name="example-2-supported"></a>範例 2:受支援

|IP 群組  |• IP 位址  |表示法  |規則  |
|---------|---------|---------|---------|
|IPGroup1 |4096     |10.0.0.0/20  |Rule1|
|IPGroup2     |4096|11.0.0.0/20|Rule1|
|     |**總計 8192**|         |         |

#### <a name="example-3-not-supported"></a>範例 3:不支援

|IP 群組  |• IP 位址  |表示法  |規則  |
|---------|---------|---------|---------|
|IPGroup1 |8192     |10.0.0.0/20, 11.0.0.0/20  |Rule1|
|     |**總計 8192**|||

#### <a name="example-4-supported"></a>範例 4:受支援

|IP 群組  |• IP 位址  |表示法  |規則  |
|---------|---------|---------|---------|
|IPGroup1 |4096     |10.0.0.0/20  |Rule1|
|IPGroup2     |4096|11.0.0.0/20|Rule2|
|     |**總計 8192**|         |         |


## <a name="related-azure-powershell-cmdlets"></a>相關 Azure 電源外殼 cmdlet

以下 Azure PowerShell cmdlet 可建立與管理 IP 群組:

- [New-AzIpGroup](https://docs.microsoft.com/powershell/module/az.network/new-azipgroup?view=azps-3.4.0)
- [移除-AzIP 群組](https://docs.microsoft.com/powershell/module/az.network/remove-azipgroup?view=azps-3.4.0)
- [Get-AzIpGroup](https://docs.microsoft.com/powershell/module/az.network/get-azipgroup?view=azps-3.4.0)
- [Set-AzIpGroup](https://docs.microsoft.com/powershell/module/az.network/set-azipgroup?view=azps-3.4.0)
- [新-Az防火牆網路規則](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallnetworkrule?view=azps-3.4.0)
- [New-AzFirewallApplicationRule](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallapplicationrule?view=azps-3.4.0)
- [新-Aa防火牆](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallnatrule?view=azps-3.4.0)

## <a name="next-steps"></a>後續步驟

- 深入了解如何[部署和設定 Azure 防火牆](tutorial-firewall-deploy-portal.md)。
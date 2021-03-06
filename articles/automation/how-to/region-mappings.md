---
title: Azure 自動化和日誌分析工作區映射
description: 本文介紹了自動化帳戶和日誌分析工作區之間允許的映射，以支援解決方案
services: automation
ms.service: automation
ms.subservice: process-automation
author: mgoedtel
ms.author: magoedte
ms.date: 05/20/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 965d6b206bb64e90fe59798ce0c37ccf029117f5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "74849508"
---
# <a name="workspace-mappings"></a>工作區對應

啟用更新管理、更改跟蹤和庫存等解決方案或非工作時間解決方案期間的啟動/停止 VM 時，僅支援某些區域連結日誌分析工作區和自動化帳戶。 此映射僅適用于自動化帳戶和日誌分析工作區。 報告到自動化帳戶或日誌分析工作區的資源可以駐留在其他區域。

## <a name="supported-mappings"></a>支援的映射

下表顯示支援的對應：

|**Log Analytics 工作區區域**|**Azure 自動化區域**|
|---|---|
|**我們**||
|東US<sup>1</sup>|EastUS2|
|WestUS2|WestUS2|
|西中院<sup>2</sup>|西中院<sup>2</sup>|
|**Canada**||
|CanadaCentral|CanadaCentral|
|**亞太**||
|AustraliaSoutheast|AustraliaSoutheast|
|SoutheastAsia|SoutheastAsia|
|CentralIndia|CentralIndia|
|JapanEast|JapanEast|
|**歐洲**||
|UKSouth|UKSouth|
|WestEurope|WestEurope|
|**US Gov**||
|USGovVirginia|USGovVirginia|

<sup>1</sup>將日誌分析工作區映射到自動化帳戶的 EastUS 映射不是區域映射的確切區域，而是正確的映射。

<sup>2</sup>由於容量限制，在創建新資源時，該區域不可用。 這包括自動化帳戶和日誌分析工作區。 然而，該區域已有的相互聯繫資源應繼續發揮作用。

## <a name="unlink-workspace"></a>取消連結工作區

如果您決定不再希望將自動化帳戶與日誌分析工作區集成，則可以直接從 Azure 門戶取消連結帳戶。 在繼續操作之前，如果正在使用更新管理、更改跟蹤和庫存，或在非工作時間使用 VM，則首先需要刪除這些 VM。 如果不刪除它們，將阻止此過程繼續。 檢閱已匯入特定解決方案的相關文章，以了解移除解決方案所需的步驟。

移除這些解決方案之後，您可以執行下列步驟以將您的自動化帳戶取消連結。

> [!NOTE]
> 某些包含舊版 Azure SQL 監視解決方案的解決方案可能已建立自動化資產，在取消連結工作區之前，可能也需要先加以移除。

1. 從 Azure 入口網站開啟您的自動化帳戶，然後在 [自動化帳戶] 頁面上，在左側的 [相關資源]**** 區段下選取 [已取消連結的工作區]****。

2. 在 [取消連結工作區] 頁面上，按一下 [取消連結工作區]****。 您將收到一個提示，驗證您希望繼續。

3. 當 Azure 自動化嘗試將您的帳戶從 Log Analytics 工作區取消連結時，您可以從功能表在 [通知]**** 下追蹤進度。

若使用「更新管理」解決方案，您可以在移除解決方案之後選擇移除已不再需要的下列項目。

* 更新排程 - 每個都會有符合您建立的更新部署名稱

* 為解決方案創建的混合工作組 - 每個組將命名為類似`machine1.contoso.com_9ceb8108-26c9-4051-b6b3-227600d715c8`。

若使用「於下班時間啟動/停止 VM」解決方案，您可以在移除解決方案之後選擇移除已不再需要的下列項目。

* 啟動及停止 VM Runbook 排程
* 啟動及停止 VM Runbook
* 變數

或者，您也可以從日誌分析工作區取消將工作區從自動化帳戶中取消連結。 在工作區上，選擇 **"相關資源**"下的 **"自動化帳戶**"。 在"自動化帳戶"頁上，選擇 **"取消連結帳戶**"。

## <a name="next-steps"></a>後續步驟

瞭解如何加入以下解決方案：

更新管理和更改跟蹤和庫存：

* 從[虛擬機器](../automation-onboard-solutions-from-vm.md)
* 從[您的自動化帳戶](../automation-onboard-solutions-from-automation-account.md)
* [流覽多台電腦](../automation-onboard-solutions-from-browse.md)時
* 從[運行簿](../automation-onboard-solutions.md)

於下班時間開始/停止 VM

* [在非工作時間部署啟動/停止 VM](../automation-solution-vm-management.md)

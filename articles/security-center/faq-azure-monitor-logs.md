---
title: Azure 安全中心常見問題 - 有關現有紀錄分析代理的問題
description: 此常見問題解答已使用日誌分析代理並考慮 Azure 安全中心的客戶的問題,Azure 安全中心是一種可説明您預防、檢測和回應威脅的產品。
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: f6384c1e9e14e38b4c44c5ac79a674839b43b4ca
ms.sourcegitcommit: ced98c83ed25ad2062cc95bab3a666b99b92db58
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/31/2020
ms.locfileid: "80436157"
---
# <a name="faq-for-customers-already-using-azure-monitor-logs"></a>已使用 Azure 監視器紀錄的客戶常見問題解答<a name="existingloganalyticscust"></a>

## <a name="does-security-center-override-any-existing-connections-between-vms-and-workspaces"></a>資訊安全中心是否會覆寫任何虛擬機器及工作區之間現有的連線？

如果 VM 已經將日誌分析代理安裝為 Azure 擴展,則安全中心不會覆蓋現有的工作區連接。 相反地，資訊安全中心會使用現有的工作區。 如果「安全」或「安全中心免費」解決方案已安裝在要報告的工作區上,則 VM 將受到保護。 

如果尚未存在,則在數據收集螢幕中選擇的工作區上安裝安全中心解決方案,並且該解決方案僅應用於相關 VM。 當您新增解決方案時，依預設會該解決方案會自動部署到與您 Log Analytics 工作區連線的所有 Windows 與 Linux 代理程式。 [解決方案目標設定](../operations-management-suite/operations-management-suite-solution-targeting.md)讓您能將範圍套用至解決方案。

> [!TIP]
> 如果日誌分析代理直接安裝在 VM 上(不作為 Azure 擴展),則安全中心不會安裝日誌分析代理,並且安全監視受到限制。

## <a name="does-security-center-install-solutions-on-my-existing-log-analytics-workspaces-what-are-the-billing-implications"></a>資訊安全中心是否會在我現有的 Log Analytics 工作區上安裝解決方案？ 這會對計費造成什麼影響？
當資訊安全中心發現已經有虛擬機器連線到您建立的工作區時，資訊安全中心會根據您的定價層啟用此工作區上的解決方案。 這些解決方案會透過[方案目標設定](../operations-management-suite/operations-management-suite-solution-targeting.md)，只套用至相關的 Azure 虛擬機器，所以收費不會改變。

- **免費層** – 資訊安全中心在工作區安裝 SecurityCenterFree 解決方案。 您不會為免費套餐收費。
- **標準層** – 資訊安全中心在工作區安裝 'Security' 解決方案。

   ![預設工作區的解決方案](./media/security-center-platform-migration-faq/solutions.png)

## <a name="i-already-have-workspaces-in-my-environment-can-i-use-them-to-collect-security-data"></a>我的環境中已經有工作區，我是否能使用這些工作區來收集安全性資料？
如果 VM 已經將日誌分析代理安裝為 Azure 擴展,則安全中心將使用現有的連接工作區。 資訊安全中心解決方案若尚未安裝，此時會安裝至工作區，且透過[解決方案目標設定](../operations-management-suite/operations-management-suite-solution-targeting.md)，此解決方案只會套用至相關的虛擬機器。

當安全中心在 VM 上安裝日誌分析代理時,它使用安全中心創建的預設工作區。

## <a name="i-already-have-security-solution-on-my-workspaces-what-are-the-billing-implications"></a>我的工作區上已經有安全性解決方案。 這會對計費造成什麼影響？
安全&審核解決方案用於為Azure VM啟用安全中心標準層功能。 如果工作區已經安裝有 Security & Audit 解決方案，資訊安全中心會使用現有的解決方案。 收費不會改變。
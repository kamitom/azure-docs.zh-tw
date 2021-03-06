---
title: Azure Application Insights 中的智慧型偵測 | Microsoft Docs
description: Application Insights 會自動深入分析您的 App 遙測，並且警告您有潛在的問題。
ms.topic: conceptual
ms.date: 02/07/2019
ms.openlocfilehash: e232295f9da2a2ae8f3c6fafdd1dc33a42e92e69
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "77671795"
---
# <a name="smart-detection-in-application-insights"></a>Application Insights 中的智慧型偵測
 智慧檢測會自動警告您 Web 應用程式中可能存在的性能問題和故障異常。 它會針對您應用程式傳送給 [Application Insights](../../azure-monitor/app/app-insights-overview.md) 的遙測執行主動式分析。 如果失敗率急遽上升或是用戶端或伺服器效能出現異常模式，您就會收到警示。 這項功能不需要進行任何設定。 只要您的應用程式傳送的遙測足夠，它就能發揮作用。

您可以從您收到的電子郵件和智慧檢測邊欄選項卡訪問智慧檢測發出的檢測。

## <a name="review-your-smart-detections"></a>檢閱智慧型偵測
您可以透過兩種方式探索偵測︰

* **接收來自 Application Insights 的電子郵件** 。 以下是典型範例︰
  
    ![電子郵件警示](./media/proactive-diagnostics/03.png)
  
    按一下大型按鈕以在入口網站中開啟更多詳細資料。
* **應用程式見解中的智慧檢測刀片**。 在 **"調查"** 功能表下選擇**智慧檢測**以查看最近檢測的清單。

![檢視最近的偵測](./media/proactive-diagnostics/04.png)

選擇檢測以查看其詳細資訊。

## <a name="what-problems-are-detected"></a>可偵測到哪些問題？
智慧檢測可檢測並通知各種問題，例如：

* [智慧型偵測 - 失敗異常](../../azure-monitor/app/proactive-failure-diagnostics.md)。 我們使用機器學習服務來為應用程式設定預期的失敗要求率，其會與負載和其他因素相互關聯。 如果失敗率超過預期限制，我們便會傳送警示。
* [智慧型偵測 - 效能異常](../../azure-monitor/app/proactive-performance-diagnostics.md)。 如果作業或相依性持續時間的回應時間變得比歷史基準慢，或如果我們在回應時間或頁面載入時間發現異常模式，您就會收到通知。   
* 一般退化和問題，如[跟蹤解構](https://docs.microsoft.com/azure/azure-monitor/app/proactive-trace-severity)，[記憶體洩漏](https://docs.microsoft.com/azure/azure-monitor/app/proactive-potential-memory-leak)，[異常量異常上升](https://docs.microsoft.com/azure/azure-monitor/app/proactive-exception-volume)[和安全反模式](https://docs.microsoft.com/azure/azure-monitor/app/proactive-application-security-detection-pack)。

(每個通知中的說明連結會帶您前往相關文章。)

## <a name="smart-detection-email-notifications"></a>智慧型偵測電子郵件通知

預設情況下，所有智慧檢測規則（標記為_預覽_的規則除外）配置為在發現檢測時發送電子郵件通知。

開啟智慧型偵測 [設定]**** 刀鋒視窗，然後選取規則，即會開啟 [編輯規則]**** 刀鋒視窗，然後便可設定特定智慧偵測規則的電子郵件通知。

或者，您也可以使用 Azure Resource Manager 範本來變更設定。 如需詳細資訊，請參閱[使用 Azure Resource Manager 範本來管理 Application Insights 智慧偵測規則](https://docs.microsoft.com/azure/azure-monitor/app/proactive-arm-config)。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>後續步驟
這些診斷工具可協助您檢查來自您的應用程式的遙測︰

* [計量瀏覽器](../../azure-monitor/app/metrics-explorer.md)
* [搜索資源管理器](../../azure-monitor/app/diagnostic-search.md)
* [分析 - 功能強大的查詢語言](../../azure-monitor/log-query/get-started-portal.md)

「智慧型偵測」是全自動的。 但是，或許您會想要再設定一些警示？

* [手動設定的度量警示](../../azure-monitor/app/alerts.md)
* [可用性 Web 測試](../../azure-monitor/app/monitor-web-app-availability.md) 


---
title: 如何擴展環境 - Azure 時間序列見解*微軟文檔
description: 瞭解如何使用 Azure 門戶縮放 Azure 時間序列見解環境。
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 01/21/2020
ms.custom: seodec18
ms.openlocfilehash: 23efda2793ef5d323089ee5b72fb1ea873de6b20
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76310980"
---
# <a name="how-to-scale-your-time-series-insights-environment"></a>如何調整您的 Time Series Insights 環境規模

本文介紹如何使用[Azure 門戶](https://portal.azure.com)更改時間序列見解環境的容量。 容量是套用至輸入速率、儲存體容量以及與選取之 SKU 相關聯成本的乘數。

您可以使用 Azure 入口網站在給定的價格 SKU 內增加或減少容量。

不過，不允許變更定價層 SKU。 例如，擁有 S1 價格 SKU 的環境不能轉換成 S2，反之亦然。

## <a name="ga-limits"></a>GA 限制

[!INCLUDE [Azure Time Series Insights GA limits](../../includes/time-series-insights-ga-limits.md)]

## <a name="change-the-capacity-of-your-environment"></a>變更您的環境容量

1. 在 Azure 入口網站中，找到並選取您的時間序列深入解析環境。

1. 在時間序列見解環境的功能表中，選擇 **"存儲配置**"。

   [![配置時間序列洞察容量](media/scale-your-environment/scale-your-environment-configure.png)](media/scale-your-environment/scale-your-environment-configure.png#lightbox)

1. 調整 [容量]**** 滑桿來選取符合您所需輸入速率和儲存容量的容量。 請注意，**輸入速率**、**儲存容量**和**估計成本**會動態更新，以顯示變更的影響。

   [![使用容量滑塊配置環境](media/scale-your-environment/scale-your-environment-slider.png)](media/scale-your-environment/scale-your-environment-slider.png#lightbox)

   或者，您可以在滑桿右邊的文字方塊中，輸入容量乘數的數值。

1. 選取 [儲存]**** 來調整環境。 認可變更之後，系統就會立即顯示進度列指示器。

1. 驗證新容量[是否足以防止限制](time-series-insights-diagnose-and-solve-problems.md)。

## <a name="next-steps"></a>後續步驟

- 如需詳細資訊，請檢閱[了解時間序列深入解析中的保留期](time-series-insights-concepts-retention.md)。

- 瞭解如何[在 Azure 時間序列見解中配置資料保留](time-series-insights-how-to-configure-retention.md)。

- 瞭解如何[規劃您的環境](time-series-insights-environment-planning.md)。
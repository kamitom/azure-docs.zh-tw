---
title: 訪問、監視和管理 GitHub 存儲庫
description: 使用 Azure 邏輯應用創建自動工作流，監視 GitHub 事件並管理 GitHub 存儲庫
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 03/02/2018
tags: connectors
ms.openlocfilehash: b67d2e860e8c69f6f2f43441144ef2f60ed3a104
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "75378444"
---
# <a name="monitor-and-manage-your-github-repo-by-using-azure-logic-apps"></a>使用 Azure 邏輯應用監視和管理 GitHub 存儲庫

GitHub 是網頁式 Git 存放庫主機服務，它提供 Git 中所有分散式版本控制和原始程式碼管理 (SCM) 功能，以及其他功能。

若要開始使用 GitHub 連接器，[請先建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="create-a-connection-to-github"></a>建立 GitHub 的連線

若要在邏輯應用程式中使用 GitHub 連接器，您必須先建立「連線」**，然後提供這些屬性的詳細資料： 

| 屬性 | 必要 | 描述 | 
| -------- | -------- | ----------- | 
| Token | 是 | 提供您的 GitHub 認證。 |

建立連線後，您就可以執行動作，並接聽本文所述的觸發程序。

> [!INCLUDE [Steps to create a connection to GitHub](../../includes/connectors-create-api-github.md)]
> 

## <a name="connector-reference"></a>連接器參考

有關觸發器、操作和限制的技術詳細資訊（由連接器的 OpenAPI（以前的斯瓦格）描述，請查看[連接器的參考頁](/connectors/github/)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
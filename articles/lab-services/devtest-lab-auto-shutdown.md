---
title: 在 Azure 開發人員測試實驗室中管理自動關閉策略 |微軟文檔
description: 瞭解如何為實驗室設置自動關閉策略，以便在虛擬機器不使用時自動關閉。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/17/2020
ms.author: spelluru
ms.openlocfilehash: a2d0b9bdfba1b96ad42e45d54faf106b2361e29d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "76264777"
---
# <a name="configure-autoshutdown-for-lab-and-compute-virtual-machines-in-azure-devtest-labs"></a>在 Azure 開發人員測試實驗室中配置實驗室和計算虛擬機器的自動關機

本文介紹如何在 DevTest 實驗室中為實驗室 VM 配置自動關機設置並計算 VM。 

## <a name="configure-autoshutdown-for-lab-vms-devtest-labs"></a>為實驗室 VM 配置自動關機（開發人員測試實驗室）
Azure DevTest Labs 可讓您管理每個實驗室的原則 (設定)，以控制實驗室的成本並儘可能避免浪費。 本文介紹如何為實驗室帳戶配置自動關閉策略，並為實驗室帳戶中的實驗室配置自動關閉設置。 若要了解如何設定每個實驗室原則，請參閱[在 Azure DevTest Labs 中定義實驗室原則](devtest-lab-set-lab-policy.md)。  

### <a name="set-auto-shut-down-policy-for-a-lab"></a>為實驗室設置自動關閉策略
身為實驗室擁有者，您可以在實驗室中設定所有 VM 的關機排程。 如此一來，您可經由執行未使用 (閒置) 的機器來節省成本。 您可以在所有的實驗室 VM 上集中強制執行關機原則，但也可讓實驗室使用者免於為其個別機器設定排程。 這項功能可讓您設定實驗室排程的原則，從完全無法控制，以至完全控制實驗室使用者。 身為實驗室擁有者，您可以採取下列步驟來設定此原則：

1. 在實驗室首頁上，選取 [設定與原則]****。
2. 左側功能表的 [排程]**** 區段中選取 [自動關機原則]****。
3. 選取其中一個選項。 下列各節提供有關這些選項的更多詳細資訊：所設定的原則只適用於實驗室中建立的新 VM，而不適用於現有的 VM。 

    ![自動關閉策略選項](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-options.png)

### <a name="configure-auto-shutdown-settings"></a>配置自動關機設置
自動關閉策略允許您指定此實驗室的 VM 關閉時間，從而有助於最大限度地減少實驗室浪費。

若要檢視 (及變更) 實驗室的原則，請依照下列步驟操作：

1. 登錄到 Azure[門戶](https://portal.azure.com)。
2. 選擇**所有服務**，然後從清單中選擇**DevTest 實驗室**。
3. 從實驗室清單中，選取所需的實驗室。   
4. 選取 [組態和原則]****。

    ![原則設定窗格](./media/devtest-lab-set-lab-policy/policies-menu.png)
5. 在實驗室的 **"配置和策略"** 窗格中，選擇 **"計畫**"下的**自動關閉**。
   
    ![自動關機](./media/devtest-lab-set-lab-policy/auto-shutdown.png)
6. 選取 [開啟]**** 來啟用此原則，以及選取 [關閉]**** 來停用它。
7. 如果您啟用這個原則，請指定時間 (和時區) 以關閉目前實驗室中的所有 VM。
8. 指定 **"是**"或 **"否**"選項，用於在指定自動關閉時間前 30 分鐘發送通知的選項。 如果您選擇 [是]****，請輸入 Webhook URL 端點或電子郵件地址，指定您要在哪裡張貼或傳送通知。 使用者會收到通知，並且給予延遲關機的選項。 有關詳細資訊，請參閱[通知](#notifications)部分。 
9. 選取 [儲存]****。

    根據預設，這個原則一經啟用，就會套用到目前實驗室的所有 VM。 要從特定 VM 中刪除此設置，請打開 VM 的管理窗格並更改其**自動關機**設置。
    
> [!NOTE]
> 如果在當前計畫時間的 30 分鐘內更新實驗室或特定實驗室虛擬機器的自動關機計畫，則更新後的關機時間將適用于第二天的計畫。 

### <a name="user-sets-a-schedule-and-can-opt-out"></a>使用者可設定排程，且可退出
如果您將實驗室設定為此原則，則實驗室使用者可以覆寫或選擇不使用實驗室排程。 此選項可讓實驗室使用者完全控制其 VM 的自動關機排程。 實驗室使用者在其 VM 自動關機排程頁面中看不到任何變更。

![自動關閉策略選項 - 1](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-1.png)

### <a name="user-sets-a-schedule-and-cannot-opt-out"></a>使用者可設定排程，但無法退出
如果您將實驗室設定為此原則，則實驗室使用者可以覆寫實驗室排程。 不過，他們無法選擇不使用自動關機原則。 此選項可確保您實驗室中的每部機器都會依循自動關機排程。 實驗室使用者可以更新其 VM 的自動關機排程，以及設定關機通知。

![自動關閉策略選項 - 2](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-2.png)

### <a name="user-has-no-control-over-the-schedule-set-by-lab-admin"></a>使用者無法控制實驗室管理員所設定的排程
如果您將實驗室設定為此原則，則實驗室使用者不能覆寫或選擇不使用實驗室排程。 此選項可讓實驗室管理員完全控制實驗室中每部機器的排程。 實驗室使用者只可以設定其 VM 的自動關機通知。

![自動關閉策略選項 - 3](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-3.png)

### <a name="notifications"></a>通知
實驗室擁有者設置自動關閉後，如果任何 VM 將受到影響，將在自動關閉觸發前 30 分鐘向實驗室使用者發送通知。 此選項使實驗室使用者有機會在關閉之前保存他們的工作。 通知還為每個 VM 提供了以下操作的連結：

- 此時跳過自動關閉
- 將自動關閉暫停一小時或 2 小時，以便他們可以在 VM 上繼續工作。

通知通過配置的 Web 掛鉤終結點或實驗室擁有者在自動關閉設置中指定的電子郵件地址發送。 Webhook 允許您構建或設置訂閱某些事件的集成。 觸發其中一個事件時，DevTest Labs 將向 Webhook 的配置 URL 發送 HTTP POST 有效負載。 如需 webhook 的詳細資訊，請參閱[建立 webhook 或 API Azure 函式](../azure-functions/functions-create-a-web-hook-or-api-function.md)。 

我們建議您使用 Web 掛鉤，因為它們受到各種應用（例如，Slack、Azure 邏輯應用等）的廣泛支援，並允許您實現自己的發送通知方式。 例如，本文將介紹如何使用 Azure 邏輯應用從電子郵件中獲取自動關閉通知。 首先，讓我們快速完成在實驗室中啟用自動關閉通知的基本步驟。   

### <a name="create-a-logic-app-that-receives-email-notifications"></a>創建接收電子郵件通知的邏輯應用
[Azure 邏輯應用](../logic-apps/logic-apps-overview.md)提供了許多開箱即用的連接器，使服務與其他用戶端（如 Office 365 和 twitter）集成變得容易。 在高級別上，為電子郵件通知設置邏輯應用的步驟可以分為四個階段： 

- 建立邏輯應用程式。 
- 配置內置範本。
- 與您的電子郵件用戶端集成
- 獲取 Webhook URL。

### <a name="create-a-logic-app"></a>建立邏輯應用程式
要開始，請使用以下步驟在 Azure 訂閱中創建邏輯應用：

1. 選擇 = 在左側功能表上**創建資源**，選擇 **"集成**"並選擇 **"邏輯應用**"。 

    ![新邏輯應用功能表](./media/devtest-lab-auto-shutdown/new-logic-app.png)
2. 在**邏輯應用 - 創建**頁面上，按照以下步驟操作： 
    1. 輸入邏輯應用**的名稱**。
    2. 選擇 Azure**訂閱**。
    3. 建立新的**資源群組**，或選取現有的資源群組。 
    4. 選擇邏輯應用**的位置**。 

        ![新邏輯應用 - 設置](./media/devtest-lab-auto-shutdown/new-logic-app-page.png)
3. 在**通知**中，選擇 **"轉到**通知上的資源"。 

    ![前往資源](./media/devtest-lab-auto-shutdown/go-to-resource.png)
4. 在 **"部署工具"** 類別下選擇**邏輯應用設計器**。

    ![選擇 HTTP 要求/回應](./media/devtest-lab-auto-shutdown/select-http-request-response-option.png)
5. 在**HTTP 要求-回應**頁上，選擇 **"使用此範本**"。 

    ![選擇"使用此範本"選項](./media/devtest-lab-auto-shutdown/select-use-this-template.png)
6. 將以下 JSON 複製到**請求正文 JSON 架構**部分： 

    ```json
    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "properties": {
            "delayUrl120": {
                "type": "string"
            },
            "delayUrl60": {
                "type": "string"
            },
            "eventType": {
                "type": "string"
            },
            "guid": {
                "type": "string"
            },
            "labName": {
                "type": "string"
            },
            "owner": {
                "type": "string"
            },
            "resourceGroupName": {
                "type": "string"
            },
            "skipUrl": {
                "type": "string"
            },
            "subscriptionId": {
                "type": "string"
            },
            "text": {
                "type": "string"
            },
            "vmName": {
                "type": "string"
            }
        },
        "required": [
            "skipUrl",
            "delayUrl60",
            "delayUrl120",
            "vmName",
            "guid",
            "owner",
            "eventType",
            "text",
            "subscriptionId",
            "resourceGroupName",
            "labName"
        ],
        "type": "object"
    }
    ```
    
    ![請求正文 JSON 架構](./media/devtest-lab-auto-shutdown/request-json.png)
7. 選擇 = 設計器中**的新步驟**，然後按照以下步驟操作：
    1. 搜索**Office 365 Outlook - 發送電子郵件**。 
    2. 選擇**從****操作**發送電子郵件。 
    
        ![發送電子郵件選項](./media/devtest-lab-auto-shutdown/select-send-email.png)
    3. 選擇 **"登錄**"以登錄電子郵件帳戶。 
    4. 選擇 **"要"** 欄位，然後選擇擁有者。
    5. 選擇**SUBJECT**，並輸入電子郵件通知的主題。 例如："關閉實驗室的電腦 vmName 名稱：實驗室名稱"。
    6. 選擇**BODY**，並定義電子郵件通知的正文內容。 例如："vmName 計畫在 15 分鐘內關閉。 按一下：URL 跳過此關閉。 延遲關閉一小時：延遲Url60。 延遲關閉 2 小時：延遲 Url120。

        ![請求正文 JSON 架構](./media/devtest-lab-auto-shutdown/email-options.png)
1. 在工具列上選取 [儲存]****。 現在，您可以複製**HTTP POST URL。** 選擇複製按鈕以將 URL 複製到剪貼簿。 

    ![WebHook URL](./media/devtest-lab-auto-shutdown/webhook-url.png)

## <a name="configure-autoshutdown-for-compute-vms"></a>為計算 VM 配置自動關機

1. 在 **"虛擬機器"** 頁上，選擇左側功能表上的 **"自動關閉**"。 
2. 在 **"自動關閉"** 頁上，選擇 **"打開**"以啟用此策略，**然後選擇"關閉**"以禁用它。
3. 如果啟用此策略，請指定應關閉 VM**的時間**（和**時區**）。
4. 指定 **"是**"或 **"否**"選項，用於在指定自動關閉時間前 30 分鐘發送通知的選項。 如果您選擇 [是]****，請輸入 Webhook URL 端點或電子郵件地址，指定您要在哪裡張貼或傳送通知。 使用者會收到通知，並且給予延遲關機的選項。 有關詳細資訊，請參閱[通知](#notifications)部分。 
9. 選取 [儲存]****。

    ![為計算 VM 配置自動關機](./media/devtest-lab-auto-shutdown/comnpute-auto-shutdown.png)

## <a name="next-steps"></a>後續步驟
要瞭解如何設置所有策略，請參閱在[Azure 開發人員測試實驗室中定義實驗室策略](devtest-lab-set-lab-policy.md)。


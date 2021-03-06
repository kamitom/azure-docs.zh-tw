---
title: HTTPS 端點 |Azure 應用商店
description: 設定 HTTPS 端點的潛在客戶管理。
author: qianw211
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 07/31/2019
ms.author: dsindona
ms.openlocfilehash: 6a0131cf94759fc529a52ea33d5392a60c5ef30c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80281590"
---
# <a name="configure-lead-management-using-an-https-endpoint"></a>使用 HTTPS 端點設定潛在客戶管理

如果合作夥伴中心中沒有明確支援客戶關係管理 （CRM） 系統以接收 Azure 應用商店和 AppSource 潛在客戶，則可以使用 MS Flow 中的 HTTPS 終結點來處理這些潛在客戶。 使用 HTTPS 終結點，這些潛在客戶可以作為電子郵件通知發送，也可以寫入 MS Flow 支援的客戶關係管理 （CRM） 系統。 本文中的說明將引導您完成使用 Microsoft Flow 創建新流的基本過程，這將生成將輸入到潛在客戶管理> **HTTPS 終結點 URL**欄位的發佈門戶中的 HTTP POST URL。 此外，還包括有關如何使用名為[Postman](https://www.getpostman.com/downloads/)的工具測試您的流程的說明，該工具可以線上找到。

## <a name="create-a-flow-using-microsoft-flow"></a>使用 Microsoft Flow 建立流程

1. 開啟 [Flow](https://flow.microsoft.com/) 網頁。 選擇 **"登錄**"，或者如果您還沒有帳戶，請選擇 **"免費註冊**"以創建一個免費的流帳戶。

2. 登入並選取功能表列上的 [我的流程]****。

3. 從**空白中選擇 +自動 -**。

    ![我的流 = 自動 - 從空白](./media/commercial-marketplace-lead-management-instructions-https/my-flows-automated.png)

4. 在 *"生成自動流*"視窗中，選擇 **"跳過**"。 

    ![構建自動流 - 跳過](./media/commercial-marketplace-lead-management-instructions-https/build-automated-flow.png)

5. 在 [搜尋連接器與觸發程序]**** 欄位中，輸入「要求」以尋找要求連接器。
6. 在 [觸發程序]** 底下，選取 [收到 HTTP 要求時]****。 

    ![請求連接器 - 觸發器](./media/commercial-marketplace-lead-management-instructions-https/request-connector.png)

7. 在"*何時收到 HTTP 要求"* 視窗副本並將下面的 JSON 架構粘貼到 **"請求正文 JSON 架構**"文字方塊中。 Microsoft 使用此架構來包含潛在客戶資料。

    ![請求連接器 - 觸發器](./media/commercial-marketplace-lead-management-instructions-https/https-request-received.png)

    **JSON 架構**

    ```JSON
    {
      "$schema": "https://json-schema.org/draft-04/schema#",
      "definitions": {},
      "id": "http://example.com/example.json",
      "properties": {
        "ActionCode": {
          "id": "/properties/ActionCode",
          "type": "string"
        },
        "OfferTitle": {
          "id": "/properties/OfferTitle",
          "type": "string"
        },
        "LeadSource": {
          "id": "/properties/LeadSource",
          "type": "string"
        },
        "UserDetails": {
          "id": "/properties/UserDetails",
          "properties": {
            "Company": {
              "id": "/properties/UserDetails/properties/Company",
              "type": "string"
            },
            "Country": {
              "id": "/properties/UserDetails/properties/Country",
              "type": "string"
            },
            "Email": {
              "id": "/properties/UserDetails/properties/Email",
              "type": "string"
            },
            "FirstName": {
              "id": "/properties/UserDetails/properties/FirstName",
              "type": "string"
            },
            "LastName": {
              "id": "/properties/UserDetails/properties/LastName",
              "type": "string"
            },
            "Phone": {
              "id": "/properties/UserDetails/properties/Phone",
              "type": "string"
            },
            "Title": {
              "id": "/properties/UserDetails/properties/Title",
              "type": "string"
            }
          },
          "type": "object"
        }
      },
      "type": "object"
    }
    ```

>[!Note]
>此時，您可以選擇連接到 CRM 系統或配置電子郵件通知。 根據您的選擇，按照其餘說明進行操作。

### <a name="to-connect-to-a-crm-system"></a>若要連線到 CRM 系統

1. 選取 [+ 新步驟] ****。
2. 選擇您選擇的 CRM 系統，通過搜索它說*搜尋連接器和操作*的位置，並在 *"操作"* 部分下選擇它，並創建新記錄。 以下螢幕截圖顯示**Dynamics 365 - 創建新**記錄作為示例。

    ![建立新的記錄](./media/commercial-marketplace-lead-management-instructions-https/create-new-record.png)

3. 提供與 CRM 系統關聯的**組織名稱**。 從 [實體名稱]**** 下拉式清單選取 [潛在客戶]****。

    ![選取潛在客戶](./media/commercial-marketplace-lead-management-instructions-https/select-leads.png)

4. Flow 會顯示一個表單來提供潛在客戶資訊。 您可以通過選擇添加動態內容來映射輸入請求中的項。 下列螢幕擷取畫面顯示 **OfferTitle** 作為範例。

    ![新增動態內容](./media/commercial-marketplace-lead-management-instructions-https/add-dynamic-content.png)

5. 對應您想要的欄位，然後選取 [儲存]****，儲存您的流程。 創建 HTTP POST URL，可在 *"何時收到 HTTP 要求"* 視窗中訪問。 使用位於 HTTP POST URL 右側的複製控制項複製此 URL - 這很重要，這樣您就不會錯誤地錯過整個 URL 的任何部分。 在發佈門戶中配置潛在客戶管理時，根據需要保存此 URL。

    ![收到 HTTP 要求時。](./media/commercial-marketplace-lead-management-instructions-https/when-http-request-received.png)

### <a name="to-set-up-email-notification"></a>若要設定電子郵件通知

1. 現在您已完成 JSON 架構，請選擇 **+ 新步驟**。
2. 在 [選擇動作]**** 底下，選取 [動作]****。
3. 在 **"操作"** 下，選擇**發送電子郵件（Office 365 Outlook）。**

    >[!Note]
    >如果要使用其他電子郵件供應商搜索，請選擇 *"發送電子郵件通知（郵件）"* 作為操作。

    ![新增電子郵件動作](./media/commercial-marketplace-lead-management-instructions-https/https-request-received-send-email.png)

4. 在 **"發送電子郵件視窗"中**，配置以下必要欄位：

   - **至**- 輸入至少一個有效的電子郵件地址，其中將發送潛在客戶。
   - **主旨** - Flow 可讓您選擇新增動態內容，像是以下螢幕擷取畫面中的 **LeadSource**。 首先鍵入欄位名稱，然後從快顯視窗按一下動態內容選取清單。 

        >[!Note] 
        > 添加欄位名稱時，可以使用"："每個欄位名稱進行跟蹤，然後輸入以創建新行。 添加欄位名稱後，可以從動態選取清單中添加每個關聯的參數。

        ![使用動態內容新增電子郵件動作](./media/commercial-marketplace-lead-management-instructions-https/add-email-using-dynamic-content.png)

   - **正文**- 從動態內容選取清單中，在電子郵件正文中添加所需的資訊。 例如，LastName、FirstName、Email 和 Company。 <br> <br> 當您完成設定電子郵件通知時，它看起來會類似以下螢幕擷取畫面中的範例。


       ![新增電子郵件動作](./media/commercial-marketplace-lead-management-instructions-https/send-an-email.png)

5. 選取 [儲存]**** 完成流程。 HTTP POST URL 將在 *"何時收到 HTTP 要求"* 視窗中創建並可訪問。 使用位於 HTTP POST URL 右側的複製控制項複製此 URL - 這很重要，這樣您就不會錯誤地錯過整個 URL 的任何部分。 在發佈門戶中配置潛在客戶管理時，根據需要保存此 URL。

   ![HTTP POST URL ](./media/commercial-marketplace-lead-management-instructions-https/http-post-url.png)

### <a name="testing"></a>測試

您可以使用名為[Postman](https://app.getpostman.com/app/download/win64)的工具（可以線上下載）測試一切工作正常。 這適用于 Windows。 

1. 啟動 Postman 並選擇 **"新** > **請求"** 以設置測試控管。 

   ![請求設置測試控管](./media/commercial-marketplace-lead-management-instructions-https/postman-request.png)

2. 填寫 *"保存請求"* 表單，然後**保存到**您創建的資料夾。

   ![儲存要求](./media/commercial-marketplace-lead-management-instructions-https/postman-save-to-test.png)

3. 從下拉清單中選擇 **"正在關閉"。** 

   ![測試我的流](./media/commercial-marketplace-lead-management-instructions-https/test-my-flow.png)

4. 從您在 MS Flow 中創建的流中粘貼 HTTP POST URL，其中顯示*輸入請求 URL*。

   ![粘貼 HTTP POST URL](./media/commercial-marketplace-lead-management-instructions-https/paste-http-post-url.png)

5. 返回[Flow](https://flow.microsoft.com/)並找到您創建的發送潛在客戶的流，通過從"流"功能表列轉到 **"我的流**"。  選擇流名稱旁邊的 3 個點，然後選擇 **"編輯**"。

   ![我的流 - 編輯](./media/commercial-marketplace-lead-management-instructions-https/my-flows-edit.png)

6. 選擇右上角的 **"測試**"，選擇"我將執行觸發器操作"，然後選擇 **"測試**"。 您將在螢幕頂部看到一個指示，指示測試已啟動

   ![測試流 - 觸發器](./media/commercial-marketplace-lead-management-instructions-https/test-flow-trigger-action.png)

7. 返回您的 Postman 應用，然後選擇"在粘貼 HTTPS URL 的位置旁邊**發送**"。

   ![測試我的流 - 發送](./media/commercial-marketplace-lead-management-instructions-https/postman-send.png)

8. 返回您的流並檢查結果。 如果一切按預期工作，您將看到一條消息，指示它成功。

   ![流 - 檢查結果](./media/commercial-marketplace-lead-management-instructions-https/my-flow-check-results.png)

9. 您還應收到一封電子郵件。 檢查您的電子郵件收件匣。 

    >[!Note] 
    >如果您沒有看到來自測試的電子郵件，請檢查您的垃圾郵件和垃圾郵件資料夾。 下面您將只注意到在配置電子郵件通知時添加的欄位標籤。 如果這是從您的報價生成的實際潛在客戶，您還會在正文和主題行中看到來自"潛在客戶連絡人"的實際資訊。

   ![收到的電子郵件](./media/commercial-marketplace-lead-management-instructions-https/email-received.png)

## <a name="configure-your-offer-to-send-leads-to-the-https-endpoint"></a>設定您的供應項目，以便將潛在客戶傳送至 HTTPS 端點

準備好在發佈門戶中配置產品/服務的首席管理資訊時，請按照以下步驟操作：

1. 導航到產品 **/服務"產品/服務"設置**頁面。
2. 在"潛在客戶管理"部分下選擇 **"連接**"。
3. 在"連接詳細資訊"快顯視窗上，為**潛在客戶目標**選擇**HTTPS 終結點**，然後從通過按照前面的步驟創建的流粘貼到**HTTPS 終結點 URL**欄位中的 HTTP POST URL 中。
4. 選取 [儲存]****。 

>[!Note] 
>您必須完成配置產品/服務的其餘部分併發布它，然後才能收到產品/服務的潛在客戶。

產生潛在客戶時，Microsoft 會將潛在客戶傳送至 Flow；如此一來，這些資訊就會路由傳送至您所設定的 CRM 系統或電子郵件地址。

![引線管理 - 連接](./media/commercial-marketplace-lead-management-instructions-https/lead-management-connect.png)

![連線詳細資料](./media/commercial-marketplace-lead-management-instructions-https/connection-details.png)

![連線詳細資料](./media/commercial-marketplace-lead-management-instructions-https/connection-details-1.png)


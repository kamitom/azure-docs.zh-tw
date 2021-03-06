---
title: 用於 B2B 集成的 EDIFACT 消息
description: 在具有 Enterprise Integration Pack 的 Azure Logic Apps 中交換適用於 B2B 企業整合的 EDIFACT 訊息 (採用 EDI 格式)
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 07/26/2016
ms.openlocfilehash: 3ada12a0cde122fb78815a1d3241d8acb9da2580
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "77651452"
---
# <a name="exchange-edifact-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>在具有 Enterprise Integration Pack 的 Azure Logic Apps 中交換適用於 B2B 企業整合的 EDIFACT 訊息

您必須先建立 EDIFACT 合約並將該合約儲存在您的整合帳戶中，才可以交換 EDIFACT 訊息。 以下是如何建立 EDIFACT 合約的步驟。

> [!NOTE]
> 本頁涵蓋 Azure Logic Apps 的 EDIFACT 功能。 如需詳細資訊，請參閱 [X12](logic-apps-enterprise-integration-x12.md)。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)  
* 至少已經在整合帳戶中定義兩個[夥伴](logic-apps-enterprise-integration-partners.md)

> [!NOTE]
> 建立合約時，您對夥伴接收或傳送之訊息中的內容必須與合約類型相符。

在您[建立整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)並[新增夥伴](logic-apps-enterprise-integration-partners.md)之後，您可以依照下列步驟建立 EDIFACT 合約。

## <a name="create-an-edifact-agreement"></a>建立 EDIFACT 合約 

1. 登錄到 Azure[門戶](https://portal.azure.com "Azure 入口網站")。 

2. 在主要 Azure 功能表上，選取 [所有服務]****。 在搜尋方塊中輸入「整合」，然後選取 [整合帳戶]****。

   ![尋找您的整合帳戶](./media/logic-apps-enterprise-integration-edifact/edifact-0.png)

   > [!TIP]
   > 如果 [所有服務]**** 未出現，您可能必須先展開功能表。 在摺疊功能表的頂端，選取 [顯示文字標籤]****。

3. 在 [整合帳戶]**** 底下，選取要在其中建立合約的整合帳戶。

   ![選取您要在其中建立合約的整合帳戶](./media/logic-apps-enterprise-integration-edifact/edifact-1-4.png)

4. 選擇 [合約]****。 如果您沒有 [合約] 圖格，請先新增圖格。   

   ![選擇 [合約] 圖格](./media/logic-apps-enterprise-integration-edifact/edifact-1-5.png)

5. 在 [合約] 頁面上，選擇 [新增]****。

   ![選擇 [新增]](./media/logic-apps-enterprise-integration-edifact/edifact-agreement-2.png)

6. 在 [新增]**** 之下，輸入合約的 [名稱]****。 針對 [合約類型]****，選取 **EDIFACT**。 選取合約的 [主機夥伴]****、[主機身分識別]****、[來賓夥伴]**** 和 [來賓身分識別]****。

   ![提供合約詳細資料](./media/logic-apps-enterprise-integration-edifact/edifact-1.png)

   | 屬性 | 描述 |
   | --- | --- |
   | 名稱 |合約的名稱 |
   | 合約類型 | 應該是 EDIFACT |
   | 主控夥伴 |合約需要主控夥伴和來賓夥伴。 主機夥伴代表設定合約的組織。 |
   | 主控身分識別 |主控夥伴的識別碼 |
   | 來賓夥伴 |合約需要主控夥伴和來賓夥伴。 來賓夥伴代表與主控夥伴有生意往來的組織。 |
   | 來賓身分識別 |來賓合作夥伴的識別碼 |
   | 接收設定 |這些屬性會套用到合約所接收的所有訊息。 |
   | 傳送設定 |這些屬性會套用到合約所傳送的所有訊息。 |
   ||| 

## <a name="configure-how-your-agreement-handles-received-messages"></a>設定合約處理所收到訊息的方式

您現在已經設定合約屬性，您可以設定此合約如何識別及處理您透過此合約從夥伴接收的內送訊息。

1. 在 [新增]**** 之下，選取 [接收設定]****。
根據您與其交換訊息之夥伴所簽署的合約，設定這些屬性。 如需屬性說明，請參閱本節中的資料表。

   [接收設定]**** 分成下列各區段：識別碼、通知、結構描述、控制編號、驗證和內部設定。

   ![設定 [接收設定]](./media/logic-apps-enterprise-integration-edifact/edifact-2.png)  

2. 完成後，請務必選擇 [確定]**** 以儲存設定。

您的合約現在已準備好處理符合您所選設定的內送訊息。

### <a name="identifiers"></a>識別碼

| 屬性 | 描述 |
| --- | --- |
| UNB6.1 (接收者參考密碼) |輸入範圍 1 到 14 個字元的英數字元值。 |
| UNB6.2 (接收者參考辨識符號) |輸入最少 1 個字元，最多 2 個字元的英數字元值。 |

### <a name="acknowledgments"></a>通知

| 屬性 | 描述 |
| --- | --- |
| 訊息回條 (CONTRL) |選取此核取方塊，將技術 (CONTRL) 通知傳回給交換傳送者。 此通知會根據合約的傳送設定傳送給交換傳送者。 |
| 通知 (CONTRL) |選取此核取方塊，將功能 (CONTRL) 通知傳回給交換傳送者。此通知會根據合約的 [傳送設定] 傳送給交換傳送者。 |

### <a name="schemas"></a>結構描述

| 屬性 | 描述 |
| --- | --- |
| UNH2.1 (TYPE) |選取交易集類型。 |
| UNH2.2 (VERSION) |輸入訊息版本號碼。 (最少 1 個字元；最多 3 個字元)。 |
| UNH2.3 (RELEASE) |輸入訊息版次號碼。 (最少 1 個字元；最多 3 個字元)。 |
| UNH2.5 (ASSOCIATED ASSIGNED CODE) |輸入指定的代碼。 (最少 6 個字元。 必須是英數字元)。 |
| UNG2.1 (APP SENDER ID) |輸入最少 1 個字元，最多 35 個字元的英數字元值。 |
| UNG2.2 (APP SENDER CODE QUALIFIER) |輸入最多 4 個字元的英數字元值。 |
| SCHEMA |從相關聯的整合帳戶選取您要使用的先前上傳結構描述。 |

### <a name="control-numbers"></a>控制編號

| 屬性 | 描述 |
| --- | --- |
| 不允許交換控制編號重複 |若要封鎖重複交換，請選取此屬性。 如果選取，EDIFACT 解碼動作會檢查所接收交換的交換控制編號 (UNB5) 是否不符合先前處理的交換控制編號。 如果偵測到相符項目，則不會處理交換。 |
| 每隔下列天數檢查是否有重複的 UNB5 |如果您選擇不允許重複的交換控制編號，您可以在提供此設定的適當值，指定要執行檢查的天數。 |
| 不允許群組控制編號重複 |若要封鎖有重複群組控制編號 (UNG5) 的交換，請選取此屬性。 |
| 不允許交易集控制編號重複 |若要封鎖有重複交易集控制編號 (UNH1) 的交換，請選取此屬性。 |
| EDIFACT 通知控制編號 |若要指定用於通知中的交易集參考編號，請輸入前置詞、參考編號範圍和後置詞的值。 |

### <a name="validation"></a>驗證

當您完成每個驗證資料列時，將會自動新增另一個驗證資料列。 如果您未指定任何規則，則驗證會使用「預設」資料列。

| 屬性 | 描述 |
| --- | --- |
| 訊息類型 |選取 EDI 訊息類型。 |
| EDI 驗證 |依照結構描述的 EDI 屬性、長度限制、空白資料元素和尾端分隔符號定義，在資料類型上執行 EDI 驗證。 |
| 擴充驗證 |如果資料類型不是 EDI，則在資料元素要求時才進行驗證，且允許重複、列舉和資料元素長度驗證 (最小/最大)。 |
| 允許前置/尾端零 |保留任何額外的前置或尾端零及空格字元。 請勿移除這些字元。 |
| 修剪前置/尾端零 |移除前置或尾端零及空格字元。 |
| 尾端分隔符號原則 |產生尾端分隔符號。 <p>選取 [不允許]****，禁止在已接收的交換中使用尾端分隔符號。 如果交換具有尾端分隔符號，則會被宣告為無效。 <p>選取 [選用]**** 以接受具有或不具有結尾分隔字元的交換。 <p>如果收到的交換必須具備尾端分隔符號，請選取 [強制性]****。 |

### <a name="internal-settings"></a>內部設置

| 屬性 | 描述 |
| --- | --- |
| 如果允許尾端分隔符號，則建立空的 XML 標籤 |選取此核取方塊，讓交換傳送者在尾端分隔符號包含空的 XML 標籤。 |
| 將交換分割為交易集 - 發生錯誤時暫停交易集|套用適當的信封至交易集，將交換中每個交易集剖析為個別的 XML 文件。 只暫停無法通過驗證的交易集。 |
| 將交換分割為交易集 - 發生錯誤時暫停交換|套用適當的信封，將交換中每個交易集剖析為個別的 XML 文件。 如果交換中有一或多個交易集無法通過驗證，則暫停整個交換。 | 
| 保留交換 - 發生錯誤時暫停交易集 |讓交換維持不變，建立整個批次交換的 XML 文件。 只暫停未通過驗證的交易集，而繼續處理所有其他交易集。 |
| 保留交換 - 發生錯誤時暫停交換 |讓交換維持不變，建立整個批次交換的 XML 文件。 如果交換中有一或多個交易集無法通過驗證，則暫停整個交換。 |

## <a name="configure-how-your-agreement-sends-messages"></a>設定合約傳送訊息的方式

您可以設定此合約如何識別及處理您透過此合約傳送給夥伴的外寄訊息。

1.  在 [新增]**** 之下，選取 [傳送設定]****。
根據您與其交換訊息之夥伴所簽署的合約，設定這些屬性。 如需屬性說明，請參閱本節中的資料表。

    [傳送設定]**** 分成下列各區段：識別碼、通知、結構描述、信封、字元集和分隔符號、控制編號以及驗證。

    ![設定 [傳送設定]](./media/logic-apps-enterprise-integration-edifact/edifact-3.png)    

2. 完成後，請務必選擇 [確定]**** 以儲存設定。

您的合約現在已準備好處理符合您所選設定的外寄訊息。

### <a name="identifiers"></a>識別碼

| 屬性 | 描述 |
| --- | --- |
| UNB1.2 (語法版本) |選取介於 **1** 與 **4** 之間的值。 |
| UNB2.3 (傳送者反向路由位址) |輸入最少 1 個字元，最多 14 個字元的英數字元值。 |
| UNB3.3 (接收者反向路由位址) |輸入最少 1 個字元，最多 14 個字元的英數字元值。 |
| UNB6.1 (接收者參考密碼) |輸入最少 1 個字元，最多 14 個字元的英數字元值。 |
| UNB6.2 (接收者參考辨識符號) |輸入最少 1 個字元，最多 2 個字元的英數字元值。 |
| UNB7 (應用程式參考識別碼) |輸入最少 1 個字元，最多 14 個字元的英數字元值。 |

### <a name="acknowledgment"></a>通知

| 屬性 | 描述 |
| --- | --- |
| 訊息回條 (CONTRL) |如果主機夥伴預計會收到技術 (CONTRL) 通知，請選取此核取方塊。 此設定指定傳送訊息的主控夥伴向來賓夥伴要求通知。 |
| 通知 (CONTRL) |如果主控夥伴預計會收到功能 (CONTRL) 通知，請選取此核取方塊。 此設定指定傳送訊息的主控夥伴向來賓夥伴要求通知。 |
| 針對接受的交易集產生 SG1/SG4 迴圈 |如果您選擇要求功能通知，選取此核取方塊，可在已接受交易集的功能 CONTRL 通知中強制產生 SG1/SG4 迴圈。 |

### <a name="schemas"></a>結構描述

| 屬性 | 描述 |
| --- | --- |
| UNH2.1 (TYPE) |選取交易集類型。 |
| UNH2.2 (VERSION) |輸入訊息版本號碼。 |
| UNH2.3 (RELEASE) |輸入訊息版次號碼。 |
| SCHEMA |選取要使用的結構描述。 結構描述位於您的整合帳戶中。 若要存取結構描述，請先將整合帳戶連結至邏輯應用程式。 |

### <a name="envelopes"></a>信封

| 屬性 | 描述 |
| --- | --- |
| UNB8 (處理優先順序代碼) |輸入不超過 1 個字元的字母值。 |
| UNB10 (通訊協議) |輸入最少 1 個字元，最多 40 個字元的英數字元值。 |
| UNB11 (測試指示器) |選取此核取方塊，指出產生的交換是測試資料 |
| 套用 UNA 區段 (字串服務建議) |選取此核取方塊，以針對要傳送的交換產生 UNA 區段。 |
| 套用 UNG 區段 (功能群組標頭) |選取此核取方塊，以在傳送給來賓夥伴之訊息的功能群組標頭中建立群組區段。 下列值用來建立 UNG 區段︰ <p>針對 **UNG1** 輸入英數值，最少一個字元，最多六個字元。 <p>針對 **UNG2.1** 輸入英數值，最少一個字元，最多 35 個字元。 <p>針對 **UNG2.2**，輸入英數值，最多四個字元。 <p>針對 **UNG3.1** 輸入英數值，最少一個字元，最多 35 個字元。 <p>針對 **UNG3.2**，輸入英數值，最多四個字元。 <p>針對 **UNG6** 輸入英數值，最少一個字元，最多三個字元。 <p>針對 **UNG7.1** 輸入英數值，最少一個字元，最多三個字元。 <p>針對 **UNG7.2** 輸入英數值，最少一個字元，最多三個字元。 <p>針對 **UNG7.3** 輸入英數值，最少一個字元，最多 6 個字元。 <p>針對 **UNG8** 輸入英數值，最少一個字元，最多 14 個字元。 |

### <a name="character-sets-and-separators"></a>字元集和分隔符號

除了字元集，您還可以為每個訊息類型輸入一組不同的分隔符號。 如果指定的訊息結構描述未指定字元集，則會使用預設字元集。

| 屬性 | 描述 |
| --- | --- |
| UNB1.1 (系統識別碼) |選取要套用於外送交換的 EDIFACT 字元集。 |
| 結構描述 |從下拉式清單中選取結構描述。 在您完成每個資料列時，便會自動新增一個資料列。 針對選取的結構描述，根據下面的分隔符號說明，選取您要使用的分隔符號集。 |
| 輸入類型 |從下拉式清單中選取輸入類型。 |
| 元件分隔符號 |若要分隔複合的資料元素，請輸入單一字元。 |
| 資料元素分隔符號 |若要分隔複合資料元素內的簡單資料元素，請輸入單一字元。 |
| 區段結束字元 |若要指出 EDI 區段的結尾，請輸入單一字元。 |
| 後置詞 |選取與區段識別項一起使用的字元。 如果指定尾碼，則區段結束字元資料元素可以空白。 如果區段結束字元留空，則必須指定尾碼。 |

### <a name="control-numbers"></a>控制編號

| 屬性 | 描述 |
| --- | --- |
| UNB5 (交換控制編號) |輸入前置詞、交換控制編號的值範圍，以及後置詞。 這些值用來產生外送交換。 前置詞和後置詞均為選擇性，而控制編號則為必要。 控制編號會隨著每則新訊息遞增；前置詞和後置詞維持不變。 |
| UNG5 (群組控制編號) |輸入前置詞、交換控制編號的值範圍，以及後置詞。 這些值用來產生群組控制編號。 前置詞和後置詞均為選擇性，而控制編號則為必要。 控制編號會隨著每則新訊息遞增，直到達到最大值為止；前置詞和後置詞維持不變。 |
| UNH1 (訊息標頭參考編號) |輸入前置詞、交換控制編號的值範圍，以及後置詞。 這些值會用來產生訊息標頭參考編號。 前置詞和後置詞均為選擇性，而參考編號則為必要。 參考編號會隨著每則新訊息遞增；前置詞和後置詞維持不變。 |

### <a name="validation"></a>驗證

當您完成每個驗證資料列時，將會自動新增另一個驗證資料列。 如果您未指定任何規則，則驗證會使用「預設」資料列。

| 屬性 | 描述 |
| --- | --- |
| 訊息類型 |選取 EDI 訊息類型。 |
| EDI 驗證 |依照結構描述、長度限制、空白資料元素和尾端分隔符號的 EDI 屬性定義在資料類型上執行 EDI 驗證。 |
| 擴充驗證 |如果資料類型不是 EDI，則在資料元素要求時才進行驗證，且允許重複、列舉和資料元素長度驗證 (最小/最大)。 |
| 允許前置/尾端零 |保留任何額外的前置或尾端零及空格字元。 請勿移除這些字元。 |
| 修剪前置/尾端零 |移除前置或尾端零字元。 |
| 尾端分隔符號原則 |產生尾端分隔符號。 <p>選取 [不允許]****，禁止在已傳送的交換中使用尾端分隔符號。 如果交換具有尾端分隔符號，則會被宣告為無效。 <p>選取 [選擇性]**** 以傳送包含或不含尾端分隔符號的交換。 <p>如果傳送的交換必須具備尾端分隔符號，請選取 [強制性]****。 |

## <a name="find-your-created-agreement"></a>尋找您建立的合約

1.  完成所有合約屬性的設定之後，請在 [新增]**** 頁面中選擇 [確定]****，以完成合約建立並回到整合帳戶。

    您新增的合約現在顯示於您的 [合約]**** 清單中。

2.  您也可以在整合帳戶概觀中檢視您的合約。 在整合帳戶功能表上，選擇 [概觀]****，然後選取 [合約]**** 圖格。 

    ![選擇 [合約] 圖格](./media/logic-apps-enterprise-integration-edifact/edifact-4.png)   

## <a name="connector-reference"></a>連接器參考

有關此連接器的更多技術詳細資訊（如連接器的 Swagger 檔所述的操作和限制），請參閱[連接器的參考頁](https://docs.microsoft.com/connectors/edifact/)。

> [!NOTE]
> 對於[整合服務環境 （ISE）](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)中的邏輯應用，此連接器的 ISE 標記版本使用[ISE 消息限制](../logic-apps/logic-apps-limits-and-config.md#message-size-limits)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
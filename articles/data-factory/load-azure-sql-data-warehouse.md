---
title: 將資料載入 Azure SQL 資料倉儲
description: 使用 Azure Data Factory 將資料複製到 Azure SQL 資料倉儲中
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 06/22/2018
ms.openlocfilehash: dc6d8c9da749e7ee60713f4357dc6d46051d89f2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80131280"
---
# <a name="load-data-into-azure-sql-data-warehouse-by-using-azure-data-factory"></a>使用 Azure Data Factory 將資料載入 Azure SQL 資料倉儲中

[Azure SQL 資料倉儲](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)是一種雲端式的相應放大資料庫，可處理巨量關聯式與非關聯式資料。 SQL 資料倉儲是以巨量平行處理 (MPP) 架構為基礎，最適用於企業資料倉儲工作負載。 它透過單獨調整儲存體和計算的彈性，來提供雲端彈性。

現在，當您使用 Azure Data Factory 時，開始使用 Azure SQL 資料倉儲會比以往更為簡單。 Azure Data Factory 是完全受控的雲端式資料整合服務。 您可以使用此服務，在建置分析解決方案時於 SQL 資料倉儲中填入現有系統的資料並節省時間。

Azure Data Factory 可針對將資料載入 Azure SQL 資料倉儲的作業提供下列優勢：

* **容易設定**：不需要編寫指令碼的直覺式 5 步驟精靈。
* **豐富的資料存儲支援**：對豐富的本地資料存儲和基於雲的資料存儲的內置支援。 如需詳細清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)的資料表。
* **安全且符合規範**：資料會透過 HTTPS 或 ExpressRoute 來傳送。 具有全域服務，可確保資料絕不會離開地理界限。
* **PolyBase 的無與倫比使用效能**：Polybase 是將資料移至 Azure SQL 資料倉儲的最有效方式。 使用暫存 Blob 功能來讓您能夠從所有類型的資料存放區 (包括 Azure Blob 儲存體和 Data Lake Store) 高速載入資料。 （預設情況下，Polybase 支援 Azure Blob 存儲和 Azure 資料湖存儲。有關詳細資訊，請參閱[複製活動性能](copy-activity-performance.md)。

本文將示範如何使用 Data Factory 複製資料工具，將資料從 Azure SQL Database 載入 Azure SQL 資料倉儲中__。 您可以依照類似的步驟，從其他類型的資料存放區複製資料。

> [!NOTE]
> 如需詳細資訊，請參閱[使用 Azure Data Factory 將資料複製到 Azure SQL 資料倉儲或從該處複製資料](connector-azure-sql-data-warehouse.md)。

## <a name="prerequisites"></a>Prerequisites

* Azure 訂用帳戶：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* Azure SQL 資料倉儲：此資料倉儲會保存從 SQL 資料庫所複製過來的資料。 如果您沒有 Azure SQL 資料倉儲，請參閱[建立 SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-get-started-tutorial.md)中的指示。
* Azure SQL Database：本教學課程會從具有 Adventure Works LT 資料範例的 Azure SQL Database 複製資料。 您可以遵循[建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)中的指示來建立 SQL 資料庫。
* Azure 儲存體帳戶：大量複製作業會使用 Azure 儲存體作為「暫存」__ Blob。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-account-create.md)中的指示。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 在左側功能表上，選擇 **"創建資源** > **資料 + 分析** > **資料工廠**" ：

   ![在 [新增] 窗格中選取資料處理站](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. 在 [新增資料處理站]**** 頁面中，為下圖所示的欄位提供值：

   ![新增資料處理站頁面](./media/load-azure-sql-data-warehouse/new-azure-data-factory.png)

    * **名稱**：輸入 Azure 資料處理站的全域唯一名稱。 如果您收到「資料處理站名稱 \"LoadSQLDWDemo\" 無法使用」的錯誤，請為資料處理站輸入其他名稱。 例如，您可以使用_**您的名稱**_**ADFTutorialDataFactory**。 請嘗試再次建立資料處理站。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。
    * **訂用帳戶**：選取用來在其中建立資料處理站的 Azure 訂用帳戶。 
    * **資源群組**：從下拉式清單中選取現有資源群組，或選取 [新建]**** 選項，然後輸入資源群組的名稱。 若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/management/overview.md)。  
    * **版本**：選取 [V2]****。
    * **位置**：選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區可位於其他位置和區域。 這些資料存放區包含 Azure Data Lake Store、Azure 儲存體、Azure SQL Database 等等。

3. 選取 [建立]****。
4. 建立完成後，請移至資料處理站。 您會看到如下圖所示的 [Data Factory]**** 首頁：

   ![Data Factory 首頁](./media/load-azure-sql-data-warehouse/data-factory-home-page.png)

   選取 [編寫與監視]**** 圖格，以在另一個索引標籤中啟動資料整合應用程式。

## <a name="load-data-into-azure-sql-data-warehouse"></a>將資料載入 Azure SQL 資料倉儲

1. 在 [開始使用]**** 頁面中，選取 [複製資料]**** 圖格以啟動複製資料工具：

   ![複製資料工具圖格](./media/load-azure-sql-data-warehouse/copy-data-tool-tile.png)
1. 在 [屬性]**** 頁面中，對 [工作名稱]**** 欄位指定 [CopyFromSQLToSQLDW]****，然後選取 [下一步]****：

    ![屬性頁面](./media/load-azure-sql-data-warehouse/copy-data-tool-properties-page.png)

1. 在 [來源資料存放區]**** 頁面中，完成下列步驟：

    a. 按一下 [+ 建立新連線]****：

    ![來源資料存放區頁面](./media/load-azure-sql-data-warehouse/new-source-linked-service.png)

    b. 從資源庫選取 [Azure SQL Database]****，然後選取 [繼續]****。 您可以在搜尋方塊中輸入 "SQL"，以篩選連接器。

    ![選取 Azure SQL DB](./media/load-azure-sql-data-warehouse/select-azure-sql-db-source.png)

    c. 在 **"新建連結服務"** 頁中，從下拉清單中選擇伺服器名稱和資料庫名稱，並指定使用者名和密碼。 按一下 [測試連線]**** 以驗證設定，然後選取 [完成]****。

    ![設定 Azure SQL DB](./media/load-azure-sql-data-warehouse/configure-azure-sql-db.png)

    d. 選取新建立的連結服務作為來源，然後按 [下一步]****。

    ![選取來源連結服務](./media/load-azure-sql-data-warehouse/select-source-linked-service.png)

1. 在 [選取要從中複製資料的資料表或使用自訂查詢]**** 頁面中，輸入 **SalesLT** 以篩選資料表。 選擇 [(全選)]**** 方塊以讓所有資料表都用於複製，然後選取 [下一步]****：

    ![選取來源資料表](./media/load-azure-sql-data-warehouse/select-source-tables.png)

1. 在 [目的地資料存放區]**** 頁面中，完成下列步驟：

    a. 按一下 [+ 建立新連線]**** 以新增連線

    ![接收資料存放區頁面](./media/load-azure-sql-data-warehouse/new-sink-linked-service.png)

    b. 從資源庫選取 [Azure SQL 資料倉儲]****，然後選取 [下一步]****。

    ![選取 Azure SQL 資料倉儲](./media/load-azure-sql-data-warehouse/select-azure-sql-dw-sink.png)

    c. 在 **"新建連結服務"** 頁中，從下拉清單中選擇伺服器名稱和資料庫名稱，並指定使用者名和密碼。 按一下 [測試連線]**** 以驗證設定，然後選取 [完成]****。

    ![設定 Azure SQL 資料倉儲](./media/load-azure-sql-data-warehouse/configure-azure-sql-dw.png)

    d. 選取新建立的連結服務作為接收端，然後按 [下一步]****。

    ![選取接收連結服務](./media/load-azure-sql-data-warehouse/select-sink-linked-service.png)

1. 在 [資料表對應]**** 頁面上檢閱內容，然後選取 [下一步]****。 隨即會顯示智慧型資料表對應。 來源資料表會根據資料表名稱來對應至目的地資料表。 如果有某個來源資料表未存在於目的地中，Azure Data Factory 預設會建立具有相同名稱的目的地資料表。 您也可以將來源資料表對應至現有的目的地資料表。

   > [!NOTE]
   > 當來源為 SQL Server 或 Azure SQL Database 時，系統便會自動為 SQL 資料倉儲接收建立資料表。 如果您從其他來源資料存放區複製資料，就必須在執行資料複製之前，先在接收 Azure SQL 資料倉儲中預先建立結構描述。

   ![資料表對應頁面](./media/load-azure-sql-data-warehouse/table-mapping.png)

1. 在 [結構描述對應]**** 頁面上檢閱內容，然後選取 [下一步]****。 系統會根據資料行名稱來進行智慧型資料表對應。 如果您讓 Data Factory 自動建立資料表，而來源和目的地存放區之間有不相容的問題時，系統便會轉換資料類型。 如果來源和目的地資料行之間有不支援的資料類型轉換，您就會在對應的資料表旁看到一則錯誤訊息。

    ![結構描述對應頁面](./media/load-azure-sql-data-warehouse/schema-mapping.png)

1. 在 [設定]**** 頁面中，完成下列步驟：

    a. 在 [暫存設定]**** 區段中，按一下 [+ 新增]**** 以新增暫存儲存體。 此儲存體可用來暫存資料，之後再使用 PolyBase 將資料載入到 SQL 資料倉儲。 複製完成後，系統會自動清除 Azure 儲存體中的暫存資料。

    ![設定暫存](./media/load-azure-sql-data-warehouse/configure-staging.png)

    b. 在 [新增連結服務]**** 頁面中，選取您的儲存體帳戶，然後選取 [完成]****。

    ![設定 Azure 儲存體](./media/load-azure-sql-data-warehouse/configure-blob-storage.png)

    c. 在 [進階設定]**** 區段中，取消選取 [使用類型預設值]**** 選項，然後選取 [下一步]****。

    ![設定 PolyBase](./media/load-azure-sql-data-warehouse/configure-polybase.png)

1. 在 [摘要]**** 頁面中檢閱設定，然後選取 [下一步]****：

    ![摘要頁面](./media/load-azure-sql-data-warehouse/summary-page.png)
1. 在**部署頁面**上選取 [監視]**** 來監視管線 (工作)：

    ![部署頁面](./media/load-azure-sql-data-warehouse/deployment-page.png)
1. 請注意，系統會自動選取左側的 [監視]**** 索引標籤。 "**操作"** 列包含用於查看活動運行詳細資訊和重新運行管道的連結：

    ![監視管線回合](./media/load-azure-sql-data-warehouse/pipeline-monitoring.png)
1. 若要檢視與此管線執行相關聯的活動執行，請選取 [動作]**** 資料行中的 [檢視活動執行]**** 連結。 若要切換回 [管線執行] 檢視，請選取頂端的 [管線]**** 連結。 選取 [重新整理]**** 可重新整理清單。

    ![監視活動回合](./media/load-azure-sql-data-warehouse/activity-monitoring.png)

1. 若要監視每個複製活動的執行詳細資料，請選取活動監控檢視中 [動作]**** 底下的 [詳細資料]**** 連結。 您可以監視的詳細資料包括從來源複製到接收的資料量、資料輸送量、執行步驟與對應的持續期間，以及所使用的設定：

    ![監視活動執行詳細資料](./media/load-azure-sql-data-warehouse/monitor-activity-run-details.png)

## <a name="next-steps"></a>後續步驟

前往下列文章，以了解 Azure SQL 資料倉儲支援：

> [!div class="nextstepaction"]
>[Azure SQL 資料倉儲連接器](connector-azure-sql-data-warehouse.md)

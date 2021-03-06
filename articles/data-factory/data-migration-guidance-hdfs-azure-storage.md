---
title: 將資料從本地 Hadoop 群集遷移到 Azure 存儲
description: 瞭解如何使用 Azure 資料工廠將資料從本地 Hadoop 群集遷移到 Azure 存儲。
services: data-factory
ms.author: yexu
author: dearandyxu
ms.reviewer: ''
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 8/30/2019
ms.openlocfilehash: afccbdbbfd5b8ddeefa621448d6170d937b518f0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "74931438"
---
# <a name="use-azure-data-factory-to-migrate-data-from-an-on-premises-hadoop-cluster-to-azure-storage"></a>使用 Azure 資料工廠將資料從本地 Hadoop 群集遷移到 Azure 存儲 

Azure 資料工廠提供了一種性能強、可靠且經濟高效的機制，用於將資料從本地 HDFS 大規模遷移到 Azure Blob 存儲或 Azure 資料湖存儲 Gen2。 

資料工廠提供了將資料從本地 HDFS 遷移到 Azure 的兩種基本方法。 您可以根據您的方案選擇方法。 

- **資料工廠 DistCp 模式**（建議）：在資料工廠中，可以使用[DistCp（](https://hadoop.apache.org/docs/current3/hadoop-distcp/DistCp.html)分散式副本）按原樣將檔案複製到 Azure Blob 存儲（包括[暫存副本](https://docs.microsoft.com/azure/data-factory/copy-activity-performance#staged-copy)）或 Azure 資料存儲第 2 代。 使用與 DistCp 集成的資料工廠利用現有的強大群集實現最佳的拷貝輸送量。 您還可以從資料工廠獲得靈活的調度和統一的監控體驗。 根據資料工廠配置，複製活動會自動構造 DistCp 命令，將資料提交到 Hadoop 群集，然後監視複製狀態。 我們建議使用資料工廠 DistCp 模式將資料從本地 Hadoop 群集遷移到 Azure。
- **資料工廠本機集成運行時模式**：DistCp 不是所有方案中的選項。 例如，在 Azure 虛擬網路環境中，DistCp 工具不支援 Azure ExpressRoute 專用對等互連 Azure 存儲虛擬網路終結點。 此外，在某些情況下，您不希望使用現有的 Hadoop 群集作為遷移資料的引擎，因此不會對群集造成重負載，這可能會影響現有 ETL 作業的性能。 相反，您可以將資料工廠集成運行時的本機功能用作將資料從本地 HDFS 複製到 Azure 的引擎。

本文提供有關這兩種方法的以下資訊：
> [!div class="checklist"]
> * 效能 
> * 複製彈性
> * 網路安全性
> * 高級解決方案架構 
> * 實施最佳實踐  

## <a name="performance"></a>效能

在資料工廠 DistCp 模式下，輸送量與單獨使用 DistCp 工具相同。 資料工廠 DistCp 模式可最大化現有 Hadoop 群集的容量。 可以使用 DistCp 進行大型群集間或群集內複製。 

DistCp 使用 MapReduce 來影響其分發、錯誤處理和恢復以及報告。 它將檔和目錄的清單擴展到任務映射的輸入中。 每個任務複製在源清單中指定的檔分區。 您可以使用與 DistCp 集成的資料工廠構建管道，以充分利用網路頻寬、存儲 IOPS 和頻寬，以最大限度地提高環境的資料移動輸送量。  

資料工廠本機集成運行時模式還允許在不同級別上並行化。 您可以使用並行性充分利用網路頻寬、存儲 IOPS 和頻寬來最大化資料移動輸送量：

- 單個複製活動可以利用可擴展的計算資源。 使用自託管集成運行時，您可以手動擴展電腦或擴展到多台電腦（[最多四個節點](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime#high-availability-and-scalability)）。 單個複製活動跨所有節點對其檔集進行分區。 
- 單個複製活動使用多個執行緒從資料存儲讀取和寫入。 
- 資料工廠控制流可以並行啟動多個複製活動。 例如，可以使用[For Each 迴圈](https://docs.microsoft.com/azure/data-factory/control-flow-for-each-activity)。 

有關詳細資訊，請參閱[複製活動性能指南](https://docs.microsoft.com/azure/data-factory/copy-activity-performance)。

## <a name="resilience"></a>復原能力

在資料工廠 DistCp 模式下，您可以使用不同的 DistCp 命令列參數（例如，`-i`忽略故障或`-update`，在原始檔案和目的檔案大小不同時寫入資料）以進行不同的恢復能力級別。

在資料工廠本機集成運行時模式下，在單個複製活動運行中，資料工廠具有內置重試機制。 它可以處理資料存儲或基礎網路中的一定級別的瞬態故障。 

當從本地 HDFS 到 Blob 存儲以及從本地 HDFS 到資料湖存儲 Gen2 執行二進位複製時，資料工廠會自動執行檢查點。 如果複製活動運行失敗或超時，則在隨後的重試（請確保重試計數為 > 1）時，副本將從最後一個失敗點恢復，而不是從開頭開始。

## <a name="network-security"></a>網路安全性 

預設情況下，資料工廠通過使用 HTTPS 協定上的加密連接將資料從本地 HDFS 傳輸到 Blob 存儲或 Azure 資料存儲 Gen2。 HTTPS 提供傳輸過程中的資料加密，並防止竊聽和中間人攻擊。 

或者，如果您不希望通過公共互聯網傳輸資料，為了更高的安全性，您可以通過 ExpressRoute 通過專用對等互連鏈路傳輸資料。 

## <a name="solution-architecture"></a>方案架構

此圖像描繪了在公共互聯網上遷移的資料：

![顯示通過公共網路遷移資料的解決方案體系結構的圖表](media/data-migration-guidance-hdfs-to-azure-storage/solution-architecture-public-network.png)

- 在此體系結構中，資料通過公共 Internet 使用 HTTPS 安全地傳輸。 
- 我們建議在公共網路環境中使用資料工廠 DistCp 模式。 您可以利用強大的現有群集來實現最佳的複製輸送量。 您還可以從資料工廠獲得靈活的調度和統一的監控體驗。
- 對於此體系結構，必須在公司防火牆後面的 Windows 電腦上安裝資料工廠自託管集成運行時，才能將 DistCp 命令提交到 Hadoop 群集並監視複製狀態。 由於機器不是移動資料的引擎（僅用於控制目的），因此機器的容量不會影響資料移動的輸送量。
- 支援 DistCp 命令的現有參數。

此圖像描述通過專用連結遷移資料： 

![顯示通過私人網路絡遷移資料的解決方案體系結構的圖表](media/data-migration-guidance-hdfs-to-azure-storage/solution-architecture-private-network.png)

- 在此體系結構中，資料通過 Azure ExpressRoute 通過專用對等互連連結遷移。 資料從不通過公共互聯網遍歷。
- DistCp 工具不支援與 Azure 存儲虛擬網路終結點進行 ExpressRoute 專用對等互連。 我們建議您通過集成運行時使用資料工廠的本機功能來遷移資料。
- 對於此體系結構，必須在 Azure 虛擬網路中的 Windows VM 上安裝資料工廠自託管的集成運行時。 您可以手動擴展 VM 或擴展到多個 VM 以充分利用網路和存儲 IOPS 或頻寬。
- 建議從每個 Azure VM（安裝了資料工廠自承載的集成運行時）開始，Standard_D32s_v3具有 32 個 vCPU 和 128 GB 記憶體。 您可以在資料移轉期間監視 VM 的 CPU 和記憶體使用方式，以查看是否需要擴展 VM 以提高性能或縮減 VM 以降低成本。
- 還可以通過將最多四個 VM 節點與單個自託管的集成運行時關聯來橫向擴展。 針對自託管集成運行時運行的單個複製作業會自動對檔集進行分區，並使用所有 VM 節點並行複製檔。 為獲得高可用性，我們建議您從兩個 VM 節點開始，以避免在資料移轉期間出現單點故障方案。
- 使用此體系結構時，您可以使用初始快照資料移轉和增量資料移轉。

## <a name="implementation-best-practices"></a>實施最佳實踐

我們建議您在實施資料移轉時遵循這些最佳實踐。

### <a name="authentication-and-credential-management"></a>身份驗證和憑據管理 

- 要對 HDFS 進行身份驗證，可以使用[Windows（Kerberos）或匿名](https://docs.microsoft.com/azure/data-factory/connector-hdfs#linked-service-properties)。 
- 支援多種身份驗證類型以連接到 Azure Blob 存儲。  我們強烈建議對[Azure 資源使用託管標識](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#managed-identity)。 託管標識基於 Azure 活動目錄 （Azure AD） 中自動託管的資料工廠標識為基礎，允許您配置管道，而無需在連結的服務定義中提供憑據。 或者，可以使用[服務主體](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#service-principal-authentication)、[共用訪問簽名](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#shared-access-signature-authentication)或[存儲帳戶金鑰](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#account-key-authentication)對 Blob 存儲進行身份驗證。 
- 還支援多種身份驗證類型以連接到資料湖存儲 Gen2。  我們強烈建議對[Azure 資源使用託管標識](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage#managed-identity)，但也可以使用[服務主體](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage#service-principal-authentication)或[存儲帳戶金鑰](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage#account-key-authentication)。 
- 如果不對 Azure 資源使用託管標識，我們強烈建議[將憑據存儲在 Azure 金鑰保存庫中](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault)，以便更輕鬆地集中管理和旋轉金鑰，而無需修改資料工廠連結的服務。 這也是[CI/CD 的最佳做法](https://docs.microsoft.com/azure/data-factory/continuous-integration-deployment#best-practices-for-cicd)。 

### <a name="initial-snapshot-data-migration"></a>初始快照資料移轉 

在資料工廠 DistCp 模式下，您可以創建一個複製活動以提交 DistCp 命令，並使用不同的參數來控制初始資料移轉行為。 

在資料工廠本機集成運行時模式下，我們建議進行資料分區，尤其是在遷移超過 10 TB 的資料時。 要對資料進行分區，請使用 HDFS 上的資料夾名稱。 然後，每個資料工廠複製作業可以一次複製一個資料夾分區。 您可以同時運行多個資料工廠複製作業，以便提高輸送量。

如果任何複製作業由於網路或資料存儲暫時性問題而失敗，則可以重新運行失敗的複製作業，以便從 HDFS 重新載入該特定分區。 正在載入其他分區的其他複製作業不受影響。

### <a name="delta-data-migration"></a>增量資料移轉 

在資料工廠 DistCp 模式下，可以使用 DistCp 命令列參數`-update`，在原始檔案和目的檔案大小不同時寫入資料，以便進行增量資料移轉。

在資料工廠本機整合模式下，從 HDFS 標識新檔或更改檔的最性能方法是使用時間分區命名約定。 當您在 HDFS 中的資料經過時間分區時，檔或資料夾名稱中的時間片資訊（例如 */yyyy/mm/dd/file.csv），* 您的管道可以輕鬆地確定哪些檔和資料夾可以增量複製。

或者，如果 HDFS 中的資料未經過時間分區，資料工廠可以使用其**上次修改日期**值標識新檔或更改的檔。 資料工廠掃描 HDFS 中的所有檔，僅複製上次修改的時間戳記大於設置值的新檔和更新檔。 

如果 HDFS 中包含大量檔，則初始檔掃描可能需要很長時間，而不管有多少檔與篩選器條件匹配。 在這種情況下，我們建議您首先使用用於初始快照遷移的相同分區對資料進行分區。 然後，檔掃描可以並行進行。

### <a name="estimate-price"></a>估計價格 

請考慮以下管道將資料從 HDFS 遷移到 Azure Blob 存儲： 

![顯示定價管道的圖表](media/data-migration-guidance-hdfs-to-azure-storage/pricing-pipeline.png)

讓我們假設以下資訊： 

- 資料總容量為 1 PB。
- 使用資料工廠本機集成運行時模式遷移資料。
- 1 PB 分為 1，000 個分區，每個副本移動一個分區。
- 每個複製活動都配置了一個與四台電腦關聯的自託管集成運行時，並且可實現 500-MBps 輸送量。
- 對於每個併發設置為**4，** 聚合輸送量為 2 GBps。
- 總共需要 146 小時才能完成遷移。

以下是基於我們的假設的估計價格： 

![顯示定價計算的表](media/data-migration-guidance-hdfs-to-azure-storage/pricing-table.png)

> [!NOTE]
> 這是一個假設的定價示例。 您的實際定價取決於環境中的實際輸送量。
> 不包括 Azure Windows VM（安裝了自承載的集成運行時）的價格。

### <a name="additional-references"></a>其他參考

- [HDFS 連接器](https://docs.microsoft.com/azure/data-factory/connector-hdfs)
- [Azure Blob 存儲連接器](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage)
- [Azure Data Lake Storage Gen2 連接器](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage)
- [複製活動性能調優指南](https://docs.microsoft.com/azure/data-factory/copy-activity-performance)
- [建立和設定自我裝載整合執行階段](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime)
- [自託管集成運行時高可用性和可擴充性](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime#high-availability-and-scalability)
- [資料移動安全性考量](https://docs.microsoft.com/azure/data-factory/data-movement-security-considerations)
- [在 Azure Key Vault 中儲存認證](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault)
- [根據時間分區的檔案名以增量方式複製檔](https://docs.microsoft.com/azure/data-factory/tutorial-incremental-copy-partitioned-file-name-copy-data-tool)
- [根據上次修改日期複製新檔和更改的檔](https://docs.microsoft.com/azure/data-factory/tutorial-incremental-copy-lastmodified-copy-data-tool)
- [資料工廠定價頁面](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/)

## <a name="next-steps"></a>後續步驟

- [使用 Azure 資料工廠從多個容器複製檔](solution-template-copy-files-multiple-containers.md)
---
title: 讀取副本 - MariaDB 的 Azure 資料庫
description: 瞭解 MariaDB Azure 資料庫中的讀取副本：選擇區域、棄置站台、連接到副本、監視複製和停止複製。
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 39edaa32b0695f4ab83206cd5701629f12295a0f
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79527906"
---
# <a name="read-replicas-in-azure-database-for-mariadb"></a>「適用於 MariaDB 的 Azure 資料庫」中的讀取複本

讀取複本功能可讓您將資料從適用於 MariaDB 的 Azure 資料庫伺服器複寫到唯讀伺服器。 您可以從主要伺服器複寫到最多五個複本。 使用 MariaDB 引擎的二進位日誌 （binlog） 檔基於位置的複製技術，使用全域事務 ID （GTID） 非同步更新副本。 若要深入了解 binlog 複寫，請參閱 [binlog 複寫概觀](https://mariadb.com/kb/en/library/replication-overview/) \(英文\)。

副本是您管理的新伺服器，類似于 MariaDB 伺服器的常規 Azure 資料庫。 針對每個讀取複本，系統每月會針對在虛擬核心中所佈建的計算量，以及在儲存體中所佈建的容量 (以 GB 為單位) 向您收費。

要瞭解有關 GTID 複製的更多詳細資訊，請參閱[MariaDB 複製文檔](https://mariadb.com/kb/en/library/gtid/)。

## <a name="when-to-use-a-read-replica"></a>何時應該使用讀取複本

讀取複本功能可針對需大量讀取的工作負載，協助改善效能及調整能力。 讀取工作負載可隔離到複本，而寫入工作負載可以導向到主要伺服器。

常見的案例是讓 BI 與分析工作負載使用讀取複本做為報告的資料來源。

由於複本是唯讀狀態，因此不會直接降低主要伺服器上的寫入容量負擔。 這項功能不是以寫入密集的工作負載為目標。

讀取副本功能使用非同步複製。 此功能不適用於同步複寫案例。 主要伺服器和複本之間將會有顯著的延遲。 複本上的資料最終仍會與主要伺服器上的資料保持一致。 請針對可接受此延遲的工作負載使用此功能。

## <a name="cross-region-replication"></a>跨區域複製
您可以在與主伺服器不同的區域中創建讀取副本。 跨區域複製對於災害復原規劃或使資料更接近使用者等方案非常有用。

> [!NOTE]
> 跨區域複製處於預覽狀態。

可以在[MariaDB 區域](https://azure.microsoft.com/global-infrastructure/services/?products=mariadb)的任何 Azure 資料庫中擁有主伺服器。  主伺服器可以在其配對區域或通用副本區域中具有副本。 下圖顯示了哪些副本區域可用，具體取決於您的主區域。

[![讀取副本區域](media/concepts-read-replica/read-replica-regions.png)](media/concepts-read-replica/read-replica-regions.png#lightbox)

### <a name="universal-replica-regions"></a>通用副本區域
您可以在以下任一區域中創建讀取副本，而不管主伺服器位於何處。 支援的通用副本區域包括：

澳大利亞東部、澳大利亞東南部、美國中部、東亞、美國東部、美國東部、日本東部、日本西部、韓國中部、韓國、美國中北部、北歐、美國中南部、東南亞、英國南部、英國西部、西歐、美國西部。

*西部 US 2 暫時不可用，作為跨區域副本位置。

### <a name="paired-regions"></a>配對的區域
除了通用副本區域外，還可以在主伺服器的 Azure 配對區域中創建讀取副本。 如果不知道區域的對，可以從[Azure 配對區域一文](../best-practices-availability-paired-regions.md)中學習詳細資訊。

如果使用跨區域副本進行災害復原規劃，我們建議您在配對區域棄置站台，而不是在其他區域中棄置站台。 配對區域可避免同時更新，並優先考慮物理隔離和資料駐留。  

但是，需要考慮以下限制： 

* 區域可用性：MariaDB 的 Azure 資料庫在美國西部 2、法國中部、阿聯酋北部和德國中部可用。 但是，它們配對的區域不可用。
    
* 單向對：某些 Azure 區域僅朝一個方向配對。 這些地區包括西印度、巴西南部和美國政府弗吉尼亞州。 
   這意味著印度西部的主伺服器可以在印度南部棄置站台。 但是，印度南部的主伺服器無法在西印度棄置站台。 這是因為西印度的次生地區是南印度，但南印度的次生地區不是西印度。

## <a name="create-a-replica"></a>建立複本

如果主伺服器沒有現有的副本伺服器，主伺服器將首先重新開機以準備複製。

啟動棄置站台工作流時，將創建 MariaDB 伺服器的空白 Azure 資料庫。 新的伺服器會具有主要伺服器上的資料。 建立時間取決於主要伺服器上的資料量，以及距離上次每週完整備份的時間。 時間的範圍可能介於數分鐘到數小時。

> [!NOTE]
> 如果您沒有在伺服器上設定儲存體警示，建議您這麼做。 警示會在伺服器接近儲存體限制時向您發出通知，因為此狀況會影響複寫。

了解如何[在 Azure 入口網站中建立讀取複本](howto-read-replicas-portal.md)。

## <a name="connect-to-a-replica"></a>連線到複本

在創建時，副本將繼承主伺服器的防火牆規則或 VNet 服務終結點。 之後，這些規則獨立于主伺服器。

複本會從主要伺服器繼承系統管理員帳戶。 系統會將主要伺服器上的所有使用者帳戶複寫到讀取複本。 您只能使用主要伺服器上可用的使用者帳戶來連線到讀取複本。

可以使用副本的主機名稱和有效使用者帳戶連接到副本，就像在 MariaDB 伺服器的常規 Azure 資料庫中一樣。 對於具有管理員使用者名**myadmin**的名為**myreplica**的伺服器，您可以使用 mysql CLI 連接到副本：

```bash
mysql -h myreplica.mariadb.database.azure.com -u myadmin@myreplica -p
```

在出現提示時，請輸入使用者帳戶的密碼。

## <a name="monitor-replication"></a>監視複寫

MariaDB 的 Azure 資料庫在 Azure 監視器中提供了**複製滯後（以秒為單位**）指標。 此計量僅適用於複本。

此指標是使用 MariaDB 命令中可用的`seconds_behind_master`指標計算`SHOW SLAVE STATUS`的。

設置警報，以在複寫延遲達到工作負載無法接受的值時通知您。

## <a name="stop-replication"></a>停止複寫

您可以停止主要伺服器與複本之間的複寫。 當主要伺服器和讀取複本之間的複寫停止時，複本就會成為獨立伺服器。 獨立伺服器中的資料是起始「停止複寫」命令時，複本上所包含的可用資料。 獨立伺服器不會跟上主要伺服器。

當您選擇停止複製到副本時，它將丟失到其以前的主副本和其他副本的所有連結。 主副本與其副本之間沒有自動容錯移轉。

> [!IMPORTANT]
> 獨立伺服器無法再次設定為複本。
> 在您停止讀取複本上的複寫之前，請確定該複本上已經有您所需要的所有資料。

了解如何[停止複寫至複本](howto-read-replicas-portal.md)。

## <a name="considerations-and-limitations"></a>考量與限制

### <a name="pricing-tiers"></a>定價層

目前只有「一般用途」與「記憶體最佳化」定價層提供讀取複本。

### <a name="master-server-restart"></a>主要伺服器重新啟動

為沒有現有副本的主複本建立副本時，主副本將首先重新開機以準備複製。 考慮到這一點，並在非尖峰時間執行這些操作。

### <a name="new-replicas"></a>新複本

讀取副本將作為 MariaDB 伺服器的新 Azure 資料庫創建。 現有伺服器無法設定為複本。 您無法為另一個讀取複本建立複本。

### <a name="replica-configuration"></a>複本設定

系統會使用與主要伺服器相同的伺服器設定來建立複本。 棄置站台後，可以獨立于主伺服器更改多個設置：計算生成、vCore、存儲、備份保留期和 MariaDB 引擎版本。 定價層也可以個別變更，但不能變更為基本層，或從基本層變更為別的層。

> [!IMPORTANT]
> 在將主要伺服器設定更新為新值之前，應將複本的設定更新為相等或更大的值。 此動作可確保複本可以跟上主要伺服器上所做的變更。

棄置站台時，防火牆規則、虛擬網路規則和參數設置將從主伺服器繼承到副本。 之後，副本的規則是獨立的。

### <a name="stopped-replicas"></a>已停止的複本

如果停止主伺服器和讀取副本之間的複製，則停止的副本將成為接受讀取和寫入的獨立伺服器。 獨立伺服器無法再次設定為複本。

### <a name="deleted-master-and-standalone-servers"></a>已刪除的主要和獨立伺服器

刪除主要伺服器時，會在所有讀取複本上停止複寫。 這些副本自動成為獨立伺服器，可以接受讀取和寫入。 主要伺服器本身會被刪除。

### <a name="user-accounts"></a>使用者帳戶

系統會將主要伺服器上的使用者複寫到讀取複本。 您只能使用主要伺服器上可用的使用者帳戶來連線到讀取複本。

### <a name="server-parameters"></a>伺服器參數

若要防止資料不同步，以及避免潛在的資料遺失或損毀，則會在使用讀取複本時，有些伺服器參數會被鎖定而無法更新。

以下伺服器參數在主伺服器和副本伺服器上都鎖定：
- [`innodb_file_per_table`](https://mariadb.com/kb/en/library/innodb-system-variables/#innodb_file_per_table) 
- [`log_bin_trust_function_creators`](https://mariadb.com/kb/en/library/replication-and-binary-log-system-variables/#log_bin_trust_function_creators)

參數[`event_scheduler`](https://mariadb.com/kb/en/library/server-system-variables/#event_scheduler)鎖定在副本伺服器上。

### <a name="other"></a>其他

- 不支援建立複本的複本。
- 記憶體中表可能會導致副本不同步。這是 MariaDB 複製技術的限制。
- 請確定主要伺服器資料表有主索引鍵。 缺少主索引鍵可能會造成主體和複本之間產生複寫延遲。

## <a name="next-steps"></a>後續步驟

- 了解如何[使用 Azure 入口網站來建立及管理讀取複本](howto-read-replicas-portal.md)
- 瞭解如何使用[Azure CLI 和 REST API 創建和管理讀取副本](howto-read-replicas-cli.md)

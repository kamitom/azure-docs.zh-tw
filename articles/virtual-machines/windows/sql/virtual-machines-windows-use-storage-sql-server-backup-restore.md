---
title: 如何使用 Azure 儲存體進行 SQL Server 備份與還原 | Microsoft Docs
description: 了解如何將 SQL Server 備份到「Azure 儲存體」。 說明將 SQL 資料庫備份到「Azure 儲存體」的好處。
services: virtual-machines-windows
documentationcenter: ''
author: MikeRayMSFT
manager: craigg
tags: azure-service-management
ms.assetid: 0db7667d-ef63-4e2b-bd4d-574802090f8b
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/31/2017
ms.author: mikeray
ms.openlocfilehash: cb19dc7262721e672bd3f54b32db9188dad7fee0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "70101883"
---
# <a name="use-azure-storage-for-sql-server-backup-and-restore"></a>使用 Azure 儲存體進行 SQL Server 備份與還原
## <a name="overview"></a>總覽
從 SQL Server 2012 SP1 CU2 開始，您現在已可以將 SQL Server 備份直接寫入 Azure Blob 儲存體服務中。 您可以使用此功能，搭配內部部署 SQL Server 資料庫或 Azure 虛擬機器中的 SQL Server 資料庫，從 Azure Blob 服務備份或還原。 備份至雲端提供的好處包括可用性、無限制的地理區域備援異地儲存體，以及輕鬆地與雲端之間來回移轉資料。 您可以使用 Transact-SQL 或 SMO 發出 BACKUP 或 RESTORE 陳述式。

SQL Server 2016 導入了新功能；您可以使用 [檔案快照集備份](https://msdn.microsoft.com/library/mt169363.aspx) 來執行幾乎即時的備份與非常快速的還原。

本主題說明為何您可能選擇使用 Azure 儲存體來進行 SQL 備份，然後說明相關的元件。 您可以使用本文章結尾所提供的資源來存取逐步解說和其他資訊，以開始搭配 SQL Server 備份來使用此服務。

## <a name="benefits-of-using-the-azure-blob-service-for-sql-server-backups"></a>使用 Azure Blob 服務進行 SQL Server 備份的好處
備份 SQL Server 時，您會面臨數個挑戰。 這些挑戰包括儲存體管理、儲存體故障風險、異地儲存體存取以及硬體組態。 這些挑戰當中有許多都是透過使用 Azure Blob 存放區服務進行 SQL Server 備份來因應。 請考量下列好處：

* **容易使用**：將您的備份存放在 Azure Blob 中可以是一個既方便、有彈性又容易存取的異地選項。 為您的 SQL Server 備份建立異地儲存體相當簡單，只需將現有的指令碼/工作修改成使用 **BACKUP TO URL** 語法即可。 異地儲存體通常應該遠離實際執行資料庫位置，防止單一災害同時影響異地和實際執行資料庫位置。 藉由選擇 [異地複寫 Azure Blob](../../../storage/common/storage-redundancy.md)，在發生可能影響整個區域的災害時，您會有額外的一層保護。
* **備份封存**：「Azure Blob 儲存體」服務提供比常用的磁帶選項更好的替代方案來封存備份。 磁帶儲存體可能需要實際運輸至異地設施並採取措施來保護媒體。 將備份存放在 Azure Blob 儲存體提供了立即、高度可用且持久的封存選項。
* **受控硬體**：使用 Azure 服務時，沒有硬體管理的額外負荷。 Azure 服務會管理硬體，並提供地埋區域備援複寫，以提供備援及硬體故障的防護。
* **無限制的儲存體**：藉由啟用直接備份到 Azure Blob 的功能，您可以存取幾乎無限制的儲存體。 或者，您也可以選擇備份到 Azure 虛擬機器磁碟，所受的限制會取決於機器大小。 您可以連結到 Azure 虛擬機器以進行備份的磁碟數是有限制的。 超大執行個體的限制為 16 個磁碟，而較小執行個體的限制則更低。
* **備份可用性**：存放在 Azure Blob 中的備份可供隨時隨地使用，並且可供輕鬆存取來還原到內部部署的 SQL Server 或在「Azure 虛擬機器」中執行的另一個 SQL Server，而不需要進行資料庫連結/中斷連結或是下載並連結 VHD。
* **成本**：只需針對使用的服務付費。 成為符合成本效益的異地與備份封存選項。 有關詳細資訊，請參閱[Azure 定價計算機](https://go.microsoft.com/fwlink/?LinkId=277060 "定價計算機")和[Azure 定價文章](https://go.microsoft.com/fwlink/?LinkId=277059 "定價文章")。
* **儲存體快照集**：當資料庫檔案是存放在 Azure Blob 中且您使用的是 SQL Server 2016 時，您可以使用 [檔案快照集備份](https://msdn.microsoft.com/library/mt169363.aspx) 來執行幾乎即時的備份與非常快速的還原。

如需詳細資訊，請參閱 [SQL Server 備份及還原與 Azure Blob 儲存體服務](https://go.microsoft.com/fwlink/?LinkId=271617)。

接下來兩節將介紹 Azure Blob 儲存體服務，包括必要的 SQL Server 元件。 了解元件及其互動，對於順利從 Azure Blob 儲存體服務使用備份與還原來說相當重要。

## <a name="azure-blob-storage-service-components"></a>Azure Blob 儲存體服務元件
備份到 Azure Blob 儲存體服務時，會使用下列 Azure 元件。

| 元件 | 描述 |
| --- | --- |
| **存儲帳戶** |儲存體帳戶是所有儲存體服務的起點。 若要存取 Azure Blob 儲存體服務，請先建立 Azure 儲存體帳戶。 如需 Azure Blob 儲存體服務的詳細資訊，請參閱 [如何使用 Azure Blob 儲存體服務](https://azure.microsoft.com/develop/net/how-to-guides/blob-storage/) |
| **容器** |容器可提供一組 Blob 的分組，並可存放不限數目的 Blob。 若要將 SQL Server 備份寫入 Azure Blob 服務，您必須至少建立根容器。 |
| **Blob** |任何類型和大小的檔案。 Blob 可通過以下 URL 格式定址 **：HTTPs：//[存儲帳戶]blob.core.windows.net/[容器]/[blob]。** 如需有關分頁 Blob 的詳細資訊，請參閱 [了解區塊 Blob 和分頁 Blob](https://msdn.microsoft.com/library/azure/ee691964.aspx) |

## <a name="sql-server-components"></a>SQL Server 元件
備份到 Azure Blob 儲存體服務時，會使用下列 SQL Server 元件。

| 元件 | 描述 |
| --- | --- |
| **Url** |URL 會指定唯一備份檔案的統一資源識別項 (URI)。 URL 用來提供 SQL Server 備份檔的位置和名稱。 此 URL 必須指向實際的 Blob，而非只有容器。 如果 Blob 不存在，就會建立 Blob。 如果指定現有的 Blob，則 BACKUP 會失敗，除非指定了 > WITH FORMAT 選項。 以下是您會在 BACKUP 命令中指定的 URL 範例： **http[s]://[storageaccount].blob.core.windows.net/[container]/[FILENAME.bak]**。 建議使用 HTTPS，但並非必要。 |
| **憑據** |連接 Azure Blob 儲存體服務及向其進行驗證所需的資訊會存放為認證。  為了讓 SQL Server 將備份寫入 Azure Blob 或從 Azure Blob 還原，必須建立 SQL Server 認證。 如需詳細資訊，請參閱 [SQL Server 認證](https://msdn.microsoft.com/library/ms189522.aspx)。 |

> [!NOTE]
> SQL Server 2016 已更新為支援區塊 Blob。 有關詳細資訊[，請參閱教程：將 Microsoft Azure Blob 存儲服務與 SQL Server 2016 資料庫一起使用](https://msdn.microsoft.com/library/dn466438.aspx)。
> 
> 

## <a name="next-steps"></a>後續步驟
1. 建立 Azure 訂用帳戶 (如果您還沒有帳戶的話)。 如果您正在評估 Azure，請考慮使用 [免費試用版](https://azure.microsoft.com/free/)。
2. 接著，完成下列其中一個教學課程，這些教學課程會逐步引導您建立儲存體帳戶及執行還原。
   
   * **SQL Server 2014**：[教學課程：SQL Server 2014 備份及還原至 Microsoft Azure Blob 儲存體服務](https://msdn.microsoft.com/library/jj720558\(v=sql.120\).aspx)。
   * **SQL Server 2016**：[Tutorial： Using the Microsoft Azure Blob storage service with SQL Server 2016 databases](https://msdn.microsoft.com/library/dn466438.aspx)
3. 檢閱其他文件，從 [使用 Microsoft Azure Blob 儲存體服務進行 SQL Server 備份及還原](https://msdn.microsoft.com/library/jj919148.aspx)開始。

如果您有任何問題，請檢閱 [SQL Server 備份至 URL 的最佳做法和疑難排解](https://msdn.microsoft.com/library/jj919149.aspx)主題。

如需其他 SQL Server 備份與還原選項，請參閱 [Azure 虛擬機器中的 SQL Server 備份和還原](virtual-machines-windows-sql-backup-recovery.md)。


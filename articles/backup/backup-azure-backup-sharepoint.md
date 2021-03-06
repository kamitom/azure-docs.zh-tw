---
title: 使用 DPM 將 SharePoint 伺服器場備份到 Azure
description: 這篇文章概述 SharePoint 伺服器陣列至 Azure 的 DPM/Azure 備份伺服器保護
ms.topic: conceptual
ms.date: 03/09/2020
ms.openlocfilehash: 0199495e3b0eb002e58c096ed9abf05d46f43f97
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80054120"
---
# <a name="back-up-a-sharepoint-farm-to-azure-with-dpm"></a>使用 DPM 將 SharePoint 伺服器場備份到 Azure

您可以使用 System Center Data Protection Manager (DPM)，將 SharePoint 伺服器陣列備份到 Microsoft Azure，其方法與備份其他資料來源極為類似。 Azure 備份提供靈活的備份排程來建立每日、每週、每月或每年備份點，並可讓您針對各種備份點執行保留原則選項。 DPM 可讓您儲存本機磁碟複本來快速達成復原時間目標 (RTO)，也可以將複本儲存到 Azure 來進行經濟實惠的長期保留。

使用 DPM 將 SharePoint 備份到 Azure 的過程與本地將 SharePoint 備份到 DPM 的過程非常相似。 本文將注意 Azure 的特定注意事項。

## <a name="sharepoint-supported-versions-and-related-protection-scenarios"></a>SharePoint 支援的版本與相關保護案例

如需支援的 SharePoint 版本清單以及備份所需的 DPM 版本清單，請參閱 [What can DPM back up?](https://docs.microsoft.com/system-center/dpm/dpm-protection-matrix?view=sc-dpm-2019#applications-backup) (DPM 可以備份的內容)。

## <a name="before-you-start"></a>開始之前

您需要先確定幾件事，再將 SharePoint 伺服器陣列備份至 Azure。

### <a name="prerequisites"></a>Prerequisites

繼續之前，請確定 [使用 Microsoft Azure 備份來保護工作負載的所有必要條件](backup-azure-dpm-introduction.md#prerequisites-and-limitations) 已滿足。 一些滿足必要條件的工作包括︰建立備份保存庫、下載保存庫認證、安裝 Azure 備份代理程式，以及向保存庫註冊 DPM/Azure 備份伺服器。

其他先決條件和限制可在 DPM 文章中找到備份[共用點](https://docs.microsoft.com/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#prerequisites-and-limitations)。

## <a name="configure-backup"></a>設定備份

若要備份 SharePoint 伺服器陣列，您要使用 ConfigureSharePoint.exe 設定 SharePoint 保護，然後在 DPM 中建立保護群組。 有關說明，請參閱在 DPM 文檔中[配置備份](https://docs.microsoft.com//system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#configure-backup)。

## <a name="monitoring"></a>監視

要監視備份作業，請按照[監視 DPM 備份](https://docs.microsoft.com/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#monitoring)中的說明操作

## <a name="restore-sharepoint-data"></a>還原 SharePoint 資料

要瞭解如何從具有 DPM 的磁片還原 SharePoint 項，請參閱[還原 SharePoint 資料](https://docs.microsoft.com/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#restore-sharepoint-data)。

## <a name="restore-a-sharepoint-database-from-azure-by-using-dpm"></a>使用 DPM 從 Azure 中還原 SharePoint 資料庫

1. 若要復原 SharePoint 內容資料庫，請瀏覽各種復原點 (如上所示)，並選取要還原的復原點。

    ![DPM SharePoint Protection8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)
2. 按兩下 SharePoint 復原點以顯示可用的 SharePoint 目錄資訊。

   > [!NOTE]
   > 由於 SharePoint 伺服器陣列在 Azure 中受長期保留保護，因此 DPM 伺服器上沒有可用的目錄資訊 (元資料)。 如此一來，每當需要復原時間點 SharePoint 內容資料庫，您就需要重新編目 SharePoint 伺服器陣列。
   >
   >
3. 按一下 [重新編目] ****。

    ![DPM SharePoint Protection10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    [雲端重新編目] **** 狀態視窗隨即會開啟。

    ![DPM SharePoint Protection11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    完成編目後，狀態會變更為 [成功] **。 按一下 **關閉**。

    ![DPM SharePoint Protection12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)
4. 按一下 DPM [復原] **** 索引標籤中顯示的 SharePoint 物件，以取得內容資料庫結構。 在項目上按一下滑鼠右鍵，然後按一下 [復原] ****。

    ![DPM SharePoint Protection13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)
5. 此時，依照本文前述的復原步驟，從磁碟復原 SharePoint 內容資料庫。

## <a name="switching-the-front-end-web-server"></a>切換前端網頁伺服器

如果您有多個前端 Web 服務器，並且想要切換 DPM 用於保護伺服器場的伺服器，請按照[切換前端 Web 服務器](https://docs.microsoft.com/system-center/dpm/back-up-sharepoint?view=sc-dpm-2019#switching-the-front-end-web-server)中的說明操作。

## <a name="next-steps"></a>後續步驟

* [Azure 備份伺服器和 DPM - 常見問題解答](backup-azure-dpm-azure-server-faq.md)
* [針對 System Center Data Protection Manager 疑難排解](backup-azure-scdpm-troubleshooting.md)

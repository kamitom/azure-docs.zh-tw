---
title: 在 Azure 備份伺服器中使用新式備份儲存體
description: 了解 Azure 備份伺服器中的新功能。 本文說明如何升級您的備份伺服器安裝。
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: c6346d7b0275a00271c1787b378a63b8365edf2d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "74172383"
---
# <a name="add-storage-to-azure-backup-server"></a>在 Azure 備份伺服器中新儲存體

Azure 備份伺服器 V2 和更新版本支援 Modern Backup Storage，可節省 50% 的儲存空間、備份速度快三倍，且更具儲存效率。 它也提供可感知工作負載的儲存體。

> [!NOTE]
> 若要使用新式備份儲存體，您必須在 Windows Server 2016 上執行備份伺服器 V2 或 V3 或在 Windows Server 2019 上執行 V3。
> 如果您在舊版 Windows Server 上執行備份伺服器 V2，Azure 備份伺服器將無法利用新式備份儲存體。 相反地，它保護工作負載的方式會和備份伺服器 V1 一樣。 如需詳細資訊，請參閱備份伺服器版本[保護對照表](backup-mabs-protection-matrix.md)。
>
> 為了實現增強的備份性能，我們建議在 Windows Server 2019 上部署具有分層存儲的 MABS v3。 有關配置分層存儲的步驟，請參閱 DPM 文章"[使用分層存儲設置 MBS"。](https://docs.microsoft.com/system-center/dpm/add-storage?view=sc-dpm-2019#set-up-mbs-with-tiered-storage)

## <a name="volumes-in-backup-server"></a>備份伺服器中的磁碟區

備份伺服器 V2 和更新版本接受儲存體磁碟區。 當您新增磁碟區時，備份伺服器會將磁碟區格式化為新式備份儲存體所需要的復原檔案系統 (ReFS)。 若要新增磁碟區，並於稍後需要時加以擴充，建議您使用此工作流程：

1. 在 VM 上設定備份伺服器。
2. 在儲存集區的虛擬磁碟上建立磁碟區：
    1. 在儲存集區中新增磁碟，並建立簡單配置的虛擬磁碟。
    2. 新增其他磁碟，並擴充虛擬磁碟。
    3. 在虛擬磁碟上建立磁碟區。
3. 在備份伺服器中新增磁碟區。
4. 設定可感知工作負載的儲存體。

## <a name="create-a-volume-for-modern-backup-storage"></a>為新式備份儲存體建立磁碟區

以磁碟區作為磁碟儲存體來使用備份伺服器 V2 或更新版本可協助您掌控儲存體。 磁碟區可以是單一磁碟。 不過，如果您日後想要擴充儲存體，請從使用儲存體空間所建立的磁碟中建立磁碟區。 如果您想要擴充磁碟區以供儲存備份，這麼做會有所幫助。 本節會提供最佳做法，讓您了解如何建立具有此設定的磁碟區。

1. 在伺服器管理員中，選擇**檔和存儲服務** > **卷** > **存儲池**。 在 [實體磁碟]**** 底下，選取 [新增儲存集區]****。

    ![建立新的儲存集區](./media/backup-mabs-add-storage/mabs-add-storage-1.png)

2. 在 [工作]**** 下拉式方塊中，選取 [新增虛擬磁碟]****。

    ![新增虛擬磁碟](./media/backup-mabs-add-storage/mabs-add-storage-2.png)

3. 選取儲存集區，然後選取 [新增實體磁碟]****。

    ![新增實體磁碟](./media/backup-mabs-add-storage/mabs-add-storage-3.png)

4. 選取實體磁碟，然後選取 [擴充虛擬磁碟]****。

    ![擴充虛擬磁碟](./media/backup-mabs-add-storage/mabs-add-storage-4.png)

5. 選取虛擬磁碟，然後選取 [新增磁碟區]****。

    ![建立新的磁碟區](./media/backup-mabs-add-storage/mabs-add-storage-5.png)

6. 在 [選取伺服器和磁碟]**** 對話方塊中，選取伺服器和新的磁碟。 然後，選擇 **"下一步**"。

    ![選取伺服器和磁碟](./media/backup-mabs-add-storage/mabs-add-storage-6.png)

## <a name="add-volumes-to-backup-server-disk-storage"></a>在備份伺服器磁碟儲存體中新增磁碟區

> [!NOTE]
>
> - 只在集區中新增一個磁碟，讓資料行計數保持為 1。 您之後則可視需要新增磁碟。
> - 如果將多個磁碟新增至存放集區，磁碟數目就會儲存為資料行數目。 當您新增更多磁碟時，它們只能是資料行數目的倍數。

若要在備份伺服器中新增磁碟區，請於 [管理]**** 窗格重新掃描儲存體，然後選取 [新增]****。 隨即會出現可供為備份伺服器儲存體新增的所有磁碟區清單。 在可用的磁碟區新增到已選取的磁碟區清單後，您可以為他們提供易記名稱，以方便您管理這些磁碟區。 若要將這些磁碟區格式化為 ReFS，讓備份伺服器可以利用新式備份儲存體的好處，請選取 [確定]****。

![新增可用的磁碟區](./media/backup-mabs-add-storage/mabs-add-storage-7.png)

## <a name="set-up-workload-aware-storage"></a>設定可感知工作負載的儲存體

在可感知工作負載的儲存體中，您可以選取會優先儲存某些工作負載類型的磁碟區。 例如，您可以將支援大量每秒輸入/輸出作業 (IOPS) 的高度耗費資源磁碟區，設定為僅儲存需要頻繁且大量備份的工作負載。 舉例來說，具有交易記錄的 SQL Server。 備份頻率不高的其他工作負載 (像是 VM)，則可備份到低成本磁碟區。

### <a name="update-dpmdiskstorage"></a>Update-DPMDiskStorage

可以使用 PowerShell Cmdlet 更新-DPMDisk 存儲設置工作負載感知存儲，該存儲更新 Azure 備份伺服器上存儲池中的卷的屬性。

語法：

`Parameter Set: Volume`

```powershell
Update-DPMDiskStorage [-Volume] <Volume> [[-FriendlyName] <String> ] [[-DatasourceType] <VolumeTag[]> ] [-Confirm] [-WhatIf] [ <CommonParameters>]
```

下列螢幕擷取畫面會顯示 PowerShell 視窗中的 Update-DPMDiskStorage Cmdlet。

![PowerShell 視窗中的 Update-DPMDiskStorage 命令](./media/backup-mabs-add-storage/mabs-add-storage-8.png)

您使用 PowerShell 所進行的變更會反映在備份伺服器管理員主控台中。

![管理員主控台中的磁碟和磁碟區](./media/backup-mabs-add-storage/mabs-add-storage-9.png)

## <a name="migrate-legacy-storage-to-modern-backup-storage"></a>將舊式儲存體移轉至新式備份儲存體

在安裝或升級為備份伺服器 V2，並將作業系統升級為 Windows Server 2016 之後，請將您的保護群組更新為使用新式備份儲存體。 根據預設，系統不會變更保護群組。 保護群組會繼續依照一開始的設定方式運作。

您可以選擇是否將保護群組更新為使用新式備份儲存體。 若要更新保護群組，請使用保留資料選項來停止保護所有資料來源。 然後，將資料來源新增至新的保護群組。

1. 在管理員主控台中選取 [保護]**** 功能。 在 [保護群組成員]**** 清單中，以滑鼠右鍵按一下成員，然後選取 [停止保護成員]****。

   ![停止保護成員](https://docs.microsoft.com/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-stop-protection1.png)

2. 在 [從群組中移除]**** 對話方塊中，檢閱儲存集區中已使用的磁碟空間和可用空間。 預設值是讓復原點留在磁碟上，並讓復原點按照所關聯的保留原則來到期。 按一下 [確定]****。

   如果您想要立即將已使用的磁碟空間歸還給可用的儲存集區，請選取 [刪除磁碟上的複本]**** 核取方塊，以刪除與該成員相關聯的備份資料 (與復原點)。

   ![[從群組中移除] 對話方塊](https://docs.microsoft.com/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-retain-data.png)

3. 建立使用新式備份儲存體的保護群組。 納入未受保護的資料來源。

## <a name="add-disks-to-increase-legacy-storage"></a>新增磁碟以增加舊式儲存體

如果您想要在備份伺服器中使用舊式儲存體，您可能需要新增磁碟以增加舊式儲存體。

若要新增磁碟儲存體：

1. 在"管理員主控台"中，選擇**管理** > **磁片存儲** > **添加**。

    ![[新增磁碟儲存體] 對話方塊](https://docs.microsoft.com/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-add-disk-storage.png)

2. 在 [新增磁碟儲存體]**** 對話方塊中選取 [新增磁碟]****。

3. 在可用磁碟清單中選取您要新增的磁碟，選取 [新增]****，然後選取 [確定]****。

## <a name="next-steps"></a>後續步驟

在安裝備份伺服器之後，請了解如何準備您的伺服器或開始保護工作負載。

- [準備備份伺服器工作負載](backup-azure-microsoft-azure-backup.md)
- [使用備份伺服器來備份 VMware 伺服器](backup-azure-backup-server-vmware.md)
- [使用備份伺服器來備份 SQL Server](backup-azure-sql-mabs.md)

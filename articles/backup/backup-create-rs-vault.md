---
title: 建立回復服務保存庫
description: 在本文中,瞭解如何創建存儲備份和恢復點的恢復服務保管庫。
ms.reviewer: sogup
ms.topic: conceptual
ms.date: 05/30/2019
ms.openlocfilehash: 439f102e8f13bff63ab388be8f10df07ab2dc7d2
ms.sourcegitcommit: b129186667a696134d3b93363f8f92d175d51475
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2020
ms.locfileid: "80672856"
---
# <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

復原服務保存庫是一個實體，可儲存一段時間以來建立的備份和復原點。 復原服務保存庫也包含與受保護虛擬機器相關聯的備份原則。

若要建立復原服務保存庫：

1. 在 [Azure 入口網站](https://portal.azure.com/)中登入您的訂用帳戶。

2. 在左側功能表上，選取 [所有服務]  。

    ![選取所有服務](./media/backup-create-rs-vault/click-all-services.png)

3. 在 [所有服務]  對話方塊中，輸入 **Recovery Services**。 資源清單會根據您的輸入進行篩選。 在資源清單中，選取 [復原服務保存庫]  。

    ![輸入並選擇復原服務保存庫](./media/backup-create-rs-vault/all-services.png)

    隨即會在訂用帳戶中出現 [復原服務保存庫] 清單。

4. 在 [復原服務保存庫]  儀表板上，選取 [新增]  。

    ![新增復原服務保存庫](./media/backup-create-rs-vault/add-button-create-vault.png)

    [復原服務保存庫]  對話方塊隨即開啟。 提供 [名稱]  、[訂用帳戶]  、[資源群組]  和 [位置]  的值。

    ![設定復原服務保存庫](./media/backup-create-rs-vault/create-new-vault-dialog.png)

   - **名稱**：輸入可識別保存庫的易記名稱。 該名稱必須是 Azure 訂用帳戶中唯一的名稱。 指定的名稱至少要有兩個字元，但不能超過 50 個字元。 名稱開頭必須是字母，且只能包含字母、數字和連字號。
   - **訂用帳戶**：選擇要使用的訂用帳戶。 如果您是唯一一個訂用帳戶的成員，就會看到該名稱。 如果您不確定要使用哪個訂用帳戶，請使用預設 (建議) 的訂用帳戶。 只有在您的公司或學校帳戶與多個 Azure 訂用帳戶相關聯時，才會有多個選擇。
   - **資源群組**：使用現有資源群組，或建立新的群組。 若要查看您訂用帳戶中可用的資源群組清單，請選取 [使用現有的]****﹐然後從下拉式清單方塊中選取資源。 若要建立新的資源群組，請選取 [新建]  ，然後輸入名稱。 如需資源群組的完整資訊，請參閱 [Azure Resource Manager 概觀](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。
   - **位置**：選取保存庫的地理區域。 若要要建立保存庫來保護虛擬機器，則保存庫**必須**與虛擬機器位於相同區域。

      > [!IMPORTANT]
      > 如果不確定您 VM 的位置，請關閉對話方塊。 移至入口網站中的虛擬機器清單。 如果您在數個區域中有虛擬機器，請在每個區域中建立復原服務保存庫。 請在第一個位置建立保存庫，然後再建立另一個位置的保存庫。 不需要指定用來儲存備份資料的儲存體帳戶。 復原服務保存庫和 Azure 備份服務會自動處理該事宜。
      >
      >

5. 當您準備好建立復原服務保存庫時，選取 [建立]  。

    ![建立復原服務保存庫](./media/backup-create-rs-vault/click-create-button.png)

    建立復原服務保存庫可能需要一些時間。 請監視入口網站右上角 [通知]  區域中的狀態通知。 保存庫建立之後，就可以在 [復原服務保存庫] 的清單中看到。 如果您沒有看到保存庫，請選取 [重新整理]  。

     ![重新整理備份保存庫的清單](./media/backup-create-rs-vault/refresh-button.png)

## <a name="set-storage-redundancy"></a>設定儲存體備援

Azure 備份會自動處理保管庫的存儲。 您需要指定如何複製該存儲。

1. 從 [復原服務保存庫]**** 刀鋒視窗，按一下 [新增保存庫]。 在 **「設置」** 部分下,按一下「**屬性**」。
2. 在**屬性**中,在 **「備份配置」** 下,按一下 **「更新**」。

3. 選擇儲存複製類型,然後單擊 **「儲存**」 。

     ![為新保存庫設定儲存體組態](./media/backup-try-azure-backup-in-10-mins/recovery-services-vault-backup-configuration.png)

   - 我們建議您使用 Azure 作為主備份存儲終結點,請繼續使用預設**的異地冗餘**設置。
   - 如果您未使用 Azure 做為主要的備份儲存體端點，則選擇 [本地備援]****，以減少 Azure 儲存體成本。
   - 詳細瞭解[地理](../storage/common/storage-redundancy-grs.md)和[本地](../storage/common/storage-redundancy-lrs.md)冗餘。

> [!NOTE]
> 在配置保管庫中的備份之前,必須更改恢復服務保管庫的**存儲複製類型**(本地冗餘/異地冗餘)。 設定備份後,將關閉選項,並且無法變更**儲存複製類型**。

## <a name="set-cross-region-restore"></a>設定交叉區域還原

作為還原選項之一,跨區域還原 (CRR) 允許您在輔助區域(即[Azure 配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions))中還原 Azure VM。 這個選項允許您:

- 當有審計或合規性要求時進行演習
- 如果主區域中發生災難,請還原 VM 或其磁碟。

要選擇此功能,請從 **「備份設定」** 邊欄選項卡中選擇 **「啟用跨區域還原**」。。

對於此過程,存在定價影響,因為它是在存儲級別。

>[!NOTE]
>開始之前：
>
>- 檢視[支援類型與區域清單的支援矩陣](backup-support-matrix.md#cross-region-restore)。
>- 跨區域還原 (CRR) 功能現在在所有 Azure 公共區域中預覽。
>- CRR 是任何 GRS 保管庫的保管庫級加入加入功能(預設情況下關閉)。
>- 請使用以下指令將訂閱上載此功能:<br>
>  `Register-AzProviderFeature -FeatureName CrossRegionRestore -ProviderNamespace Microsoft.RecoveryServices`
>- 如果您在公共有限預覽期間加入此功能,審核審批電子郵件將包括定價政策詳細資訊。
>- 加入加入后,備份專案可能需要長達 48 小時才能在輔助區域中可用。
>- 目前,CRR 僅支援備份管理類型 - ARM Azure VM(不支援經典 Azure VM)。  當其他管理類型支援 CRR 時 **,它們將自動**註冊。

### <a name="configure-cross-region-restore"></a>設定跨區域還原

使用 GRS 冗餘創建的保管庫包括配置跨區域還原功能的選項。 每個 GRS 保管庫都將有一個橫幅,該橫幅將鏈接到文檔。 要為保管庫配置 CRR,請轉到備份配置邊欄選項卡,該邊欄選項卡包含啟用此功能的選項。

 ![備份設定橫幅](./media/backup-azure-arm-restore-vms/banner.png)

1. 從門戶轉到恢復服務保管庫>設置>屬性。
2. 按一**下 此保管庫中啟用跨區域還原**以啟用該功能。

   ![按下此保存庫中的啟用跨區域還原之前](./media/backup-azure-arm-restore-vms/backup-configuration1.png)

   ![按下此保存庫中的啟用跨區域還原後](./media/backup-azure-arm-restore-vms/backup-configuration2.png)

瞭解如何[檢視輔助區域中的備份項目](backup-azure-arm-restore-vms.md#view-backup-items-in-secondary-region)。

瞭解如何[在輔助區域中還原](backup-azure-arm-restore-vms.md#restore-in-secondary-region)。

讓您如何[監視輔助區域還原工作](backup-azure-arm-restore-vms.md#monitoring-secondary-region-restore-jobs)。

## <a name="modifying-default-settings"></a>變更預設設定

我們強烈建議您在在保管庫中配置備份之前查看**儲存複製類型****和安全設置**的預設設置。

- 預設情況下 **,儲存複製類型**設定為**異地冗餘**。 配置備份後,將禁用修改選項。 按照[以下步驟](https://docs.microsoft.com/azure/backup/backup-create-rs-vault#set-storage-redundancy)查看和修改設置。

- 預設情況下,**軟刪除**在新創建的保管庫中**啟用**,以保護備份數據免遭意外或惡意刪除。 按照[以下步驟](https://docs.microsoft.com/azure/backup/backup-azure-security-feature-cloud#disabling-soft-delete)查看和修改設置。

## <a name="next-steps"></a>後續步驟

[瞭解](backup-azure-recovery-services-vault-overview.md)恢復服務保管庫。
[瞭解](backup-azure-delete-vault.md)刪除恢復服務保管庫。

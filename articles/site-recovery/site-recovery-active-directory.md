---
title: 使用 Azure 網站恢復設定活動目錄/DNS 災難復原
description: 本文說明如何使用 Azure Site Recovery 針對 Active Directory 和 DNS 實作災害復原解決方案。
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: mayg
ms.openlocfilehash: 2cf4f22be2a4407d73fcc7bb340fad647c8aa145
ms.sourcegitcommit: 980c3d827cc0f25b94b1eb93fd3d9041f3593036
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/02/2020
ms.locfileid: "80546508"
---
# <a name="set-up-disaster-recovery-for-active-directory-and-dns"></a>設定 Active Directory 和 DNS 的災害復原

企業應用程式 (例如 SharePoint、Dynamics AX 和 SAP) 須倚賴 Active Directory 和 DNS 基礎結構才能正確運作。 為應用程式設定災難恢復時,通常需要在恢復其他應用程式元件之前恢復活動目錄和域名系統 (DNS),以確保正確的應用程式功能。

您可以使用 [Site Recovery](site-recovery-overview.md) 來建立 Active Directory 的災害復原計劃。 發生中斷的狀況時，您可以起始容錯移轉。 您可以讓 Active Directory 在短短幾分鐘內啟動並執行。 如果您已在主要站台 (例如 SharePoint 和 SAP) 中為多個應用程式部署 Active Directory，您可能會想要容錯移轉至完整的網站。 您可以先使用 Site Recovery 進行 Active Directory 的容錯移轉。 接著，您可以使用應用程式專屬復原計劃進行其他應用程式的容錯移轉。

本文說明如何建立 Active Directory 的災害復原解決方案。 其中包括先決條件，以及容錯移轉指示。 開始之前，您應該先熟悉 Active Directory 與 Site Recovery。

## <a name="prerequisites"></a>Prerequisites

- 如果您要複寫至 Azure，請[準備 Azure 資源](tutorial-prepare-azure.md)，包括訂用帳戶、Azure 虛擬網路、儲存體帳戶和復原服務保存庫。
- 檢視所有元件[支援要求](site-recovery-support-matrix-to-azure.md)。

## <a name="replicate-the-domain-controller"></a>複寫網域控制站

- 您必須在至少一個承載域控制器或 DNS 的虛擬機器 (VM) 上設置網站恢復複製。
- 如果您的環境中有多個網域控制站，則您也必須在目標站台上設定額外的網域控制站。 額外的網域控制站可位於 Azure 中，或位於次要內部部署資料中心。
- 如果您有只有少數幾個應用程式和一個網域控制站，您可以將整個站台一併容錯移轉。 在這種情況下,我們建議使用網站恢復將域控制器複製到目標網站(在 Azure 中或在輔助本地數據中心中)。 相同的複寫網域控制站或 DNS 虛擬機器也可用於[測試容錯移轉](#test-failover-considerations)。
- 如果您的環境中有許多應用程式和多個網域控制站，或您打算一次容錯移轉數個應用程式，建議您除了使用 Site Recovery 複寫網域控制站虛擬機器之外，也應在目標站台 (位於 Azure 或次要內部部署資料中心) 上設定額外的網域控制站。 對於[測試故障轉移](#test-failover-considerations),可以使用由網站恢復複製的域控制器。 對於容錯移轉，您可以在目標站台上使用額外的網域控制站。

## <a name="enable-protection-with-site-recovery"></a>使用 Site Recovery 啟用保護

您可以使用 Site Recovery 來保護裝載網域控制站或 DNS 的虛擬機器。

### <a name="protect-the-vm"></a>保護 VM

使用 Site Recovery 複寫的網域控制站會用於[測試容錯移轉](#test-failover-considerations)。 請確定它符合下列需求︰

1. 網域控制站是通用類別目錄伺服器。
1. 域控制器應是測試故障轉移期間所需角色的靈活單主機操作 (FSMO) 角色擁有者。 否則在容錯移轉之後，將必須[收回](https://support.microsoft.com/help/255504/using-ntdsutil-exe-to-transfer-or-seize-fsmo-roles-to-a-domain-control)這些角色。

### <a name="configure-vm-network-settings"></a>進行 VM 網路設定

對於裝載網域控制站或 DNS 的虛擬機器，使用 Site Recovery，在複寫虛擬機器的 [計算和網路]**** 設定下進行網路設定。 這可確保虛擬機器在容錯移轉之後會連結至正確的網路。

## <a name="protect-active-directory"></a>保護 Active Directory

### <a name="site-to-site-protection"></a>站對站保護

請在次要網站上建立網域控制站。 將伺服器提升為域控制器角色時,請指定主網站上正在使用的同一域的名稱。 您可以使用 **Active Directory 網站和服務** 嵌入式管理單元來設定網站要新增至的網站連結物件上的設定。 藉由設定網站連結，您可以控制兩個以上的網站之間執行複寫的時機和頻率。 如需詳細資訊，請參閱[排程網站之間的複寫](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731862(v=ws.11))。

### <a name="site-to-azure-protection"></a>站對 Azure 保護

首先，在 Azure 虛擬網路中建立網域控制站。 當您將伺服器提升至網域控制站角色時，請指定主要網站中所使用的相同網域名稱。

然後，重新設定虛擬網路的 DNS 伺服器以在 Azure 中使用此 DNS 伺服器。

:::image type="content" source="./media/site-recovery-active-directory/azure-network.png" alt-text="Azure 網路":::

### <a name="azure-to-azure-protection"></a>Azure 對 Azure 保護

首先，在 Azure 虛擬網路中建立網域控制站。 當您將伺服器提升至網域控制站角色時，請指定主要網站中所使用的相同網域名稱。

然後，重新設定虛擬網路的 DNS 伺服器以在 Azure 中使用此 DNS 伺服器。

## <a name="test-failover-considerations"></a>測試容錯移轉考量

為了避免對生產工作負載的影響,測試故障轉移發生在與生產網路隔離的網路中。

大部分的應用程式都需要網域控制站或 DNS 伺服器才能運作。 因此，在應用程式容錯移轉之前，您必須在要用於測試容錯移轉的隔離網路中建立網域控制站。 為此，最簡單的方法是使用 Site Recovery 來複寫裝載網域控制站或 DNS 的虛擬機器。 然後，在執行應用程式復原計劃的測試容錯移轉前，您應先執行網域控制站虛擬機器的測試容錯移轉。

1. 使用 Site Recovery [複寫](vmware-azure-tutorial.md)裝載網域控制站或 DNS 的虛擬機器。
1. 建立隔離的網路。 在 Azure 中建立的任何虛擬網路，依預設都會與其他網路隔離。 建議您對此網路使用您的生產網路所使用的相同 IP 範圍。 請勿在此網路上啟用網站對網站連線能力。
1. 提供隔離網路中 DNS IP 位址。 請使用您想要讓 DNS 虛擬機器取得的 IP 位址。 如果您要複寫至 Azure，請提供用於容錯移轉之虛擬機器的 IP 位址。 若要輸入 IP 位址，請在複寫虛擬機器的 [計算和網路]**** 設定中選取 [目標 IP]**** 設定。

   :::image type="content" source="./media/site-recovery-active-directory/azure-test-network.png" alt-text="Azure 測試網路":::

   > [!TIP]
   > Site Recovery 會嘗試在名稱相同的子網路中建立測試虛擬機器，並使用虛擬機器的 [計算與網路]**** 設定中提供的 IP 位址。 如果針對測試容錯移轉提供的 Azure 虛擬網路中沒有名稱相同的子網路，則會在依字母順序的第一個子網路中建立測試虛擬機器。
   >
   > 如果目標 IP 位址是所選子網路的一部分，則 Site Recovery 會使用目標 IP 位址嘗試建立測試容錯移轉虛擬機器。 如果目標 IP 不是所選子網路的一部分，則會使用所選子網路中的下一個可用 IP 建立測試容錯移轉虛擬機器。

### <a name="test-failover-to-a-secondary-site"></a>對次要網站執行測試容錯移轉

1. 如果您要複寫至其他內部部署網站，且您是使用 DHCP，請[針對測試容錯移轉設定 DNS 和 DHCP](hyper-v-vmm-test-failover.md#prepare-dhcp)。
1. 對隔離網路中執行的網域控制站虛擬機器進行測試容錯移轉。 請使用網域控制站虛擬機器的最新可用_應用程式一致_復原點來執行測試容錯移轉。
1. 針對包含應用程式執行所在之虛擬機器的復原計劃執行測試容錯移轉。
1. 測試完成之後，請在網域控制站虛擬機器上_清除測試容錯移轉_。 此步驟會刪除針對測試容錯移轉所建立的網域控制站。

### <a name="remove-references-to-other-domain-controllers"></a>移除對其他網域控制站的參考

在起始測試容錯移轉時，請勿在測試網路中納入所有網域控制站。 若要移除對您生產環境中其他網域控制站的參考，您可能必須[收回 FSMO Active Directory 角色](https://support.microsoft.com/help/255504/using-ntdsutil-exe-to-transfer-or-seize-fsmo-roles-to-a-domain-control)，並針對遺漏的網域控制站執行[中繼資料清理](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816907(v=ws.10))。

### <a name="issues-caused-by-virtualization-safeguards"></a>因虛擬化保護措施而產生的問題

> [!IMPORTANT]
> 本節所述的部分組態不是標準或預設的網域控制站組態。 如果您不想對生產網域控制站進行這些變更，您可以建立專門供 Site Recovery 用於測試容錯移轉的網域控制站。 您只需對該網域控制站進行這些變更即可。

從 Windows Server 2012 開始，[Active Directory Domain Services (AD DS) 已內建額外保護措施](/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100)。 如果基礎虛擬機器管理程式平台支援**VM-GenerationID,** 這些安全措施有助於保護虛擬化域控制器免受更新序列號 (USN) 回滾的影響。 Azure 支援 **VM-GenerationID**。 因此，在 Azure 虛擬機器上執行 Windows Server 2012 或更新版本的網域控制站，具有額外的保護措施。

當 **VM-GenerationID** 重設時，AD DS 資料庫的 **InvocationID** 值也會重設。 此外,將丟棄相對 ID (RID)`SYSVOL`池,並將資料夾標記為非權威。 有關詳細資訊,請參閱[主動目錄網域服務虛擬化](/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100)[和安全性虛擬化分散式檔案系統複製 (DFSR) 簡介](https://techcommunity.microsoft.com/t5/storage-at-microsoft/safely-virtualizing-dfsr/ba-p/424671)。

容錯移轉至 Azure 可能會導致 **VM-GenerationID** 重設。 重設 **VM-GenerationID** 後，在網域控制站虛擬機器於 Azure 中啟動時將會執行額外的保護措施。 這可能導致登錄域控制器虛擬機器時出現重大延遲。

此網域控制站只會用於測試容錯移轉，因此不需要虛擬化保護措施。 為了確保網域控制器虛擬機器的**VM 產生 ID**值不變更,可以在本地域`DWORD`控制器中將以下 值更改為**4:**

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\gencounter\Start`

#### <a name="symptoms-of-virtualization-safeguards"></a>虛擬化保護措施的徵兆

如果在測試容錯移轉之後觸發了虛擬化保護措施，您可能會看到下列一或多個徵兆︰

- **產生識別碼**值變更:

  :::image type="content" source="./media/site-recovery-active-directory/Event2170.png" alt-text="世代識別碼變更":::

- **呼叫 ID**值變更:

  :::image type="content" source="./media/site-recovery-active-directory/Event1109.png" alt-text="呼叫識別碼":::

- `SYSVOL`資料夾和`NETLOGON`共用不可用。

  :::image type="content" source="./media/site-recovery-active-directory/sysvolshare.png" alt-text="SYSVOL 資料夾分享":::

  :::image type="content" source="./media/site-recovery-active-directory/Event13565.png" alt-text="NtFrs SYSVOL 資料夾":::

- DFSR 資料庫會被刪除。

  :::image type="content" source="./media/site-recovery-active-directory/Event2208.png" alt-text="DFSR 資料庫會被刪除":::

### <a name="troubleshoot-domain-controller-issues-during-test-failover"></a>在測試容錯移轉期間，對網域控制站問題進行疑難排解

> [!IMPORTANT]
> 本節所述的部分組態不是標準或預設的網域控制站組態。 如果您不想對生產網域控制站進行這些變更，您可以建立 Site Recovery 測試容錯移轉專用的網域控制站。 您只需對此專用的網域控制站進行變更即可。

1. 在指令提示符下,執行以下指令以檢查是否`SYSVOL`分享`NETLOGON`資料夾 與資料夾:

    `NET SHARE`

1. 在命令提示字元上執行下列命令，以確保網域控制站正常運作：

    `dcdiag /v > dcdiag.txt`

1. 在輸出記錄中尋找下列文字。 若有此文字，即可確認網域控制站運作正常。

    - `passed test Connectivity`
    - `passed test Advertising`
    - `passed test MachineAccount`

如果符合上述條件，表示網域控制站應該是正常運作。 若非如此，請完成下列步驟：

1. 執行網域控制站的權威復原。 記住下列資訊：

    - 儘管我們不建議使用[檔案複製服務 (FRS)](https://techcommunity.microsoft.com/t5/storage-at-microsoft/the-end-is-nigh-for-frs-8211-updated-for-ws2016/ba-p/425379)進行複製,但如果使用 FRS 複製,請按照授權還原的步驟執行。 其程序請見[使用 BurFlags 登錄機碼重新初始化檔案複寫服務](https://support.microsoft.com/kb/290762)。

      如需 BurFlags 的詳細資訊，請參閱部落格文章 [D2 和 D4：有何功用？](/archive/blogs/janelewis/d2-and-d4-what-is-it-for)。

    - 如果您使用 DFSR 複寫，請完成權威還原的步驟。 此過程在 Force 中描述了[DFSR 複製的 SYSVOL 資料夾(如 FRS 的' D4/D2" 的權威與非權威同步](https://support.microsoft.com/kb/2218556)。

      您也可以使用 PowerShell 函式。 如需詳細資訊，請參閱 [DFSR-SYSVOL 授權/非權威還原 PowerShell 函式](/archive/blogs/thbouche/dfsr-sysvol-authoritative-non-authoritative-restore-powershell-functions)。

1. 將內部部署網域控制站中的下列登錄機碼設為 **0**，以略過初始同步需求。 如果`DWORD`不存在,則可以在**參數**節點下創建它。

   `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters\Repl Perform Initial Synchronizations`

   如需詳細資訊，請參閱[疑難排解 DNS 事件識別碼 4013：DNS 伺服器無法載入 AD 整合的 DNS 區域](https://support.microsoft.com/kb/2001093)。

1. 停用讓通用類別目錄伺服器可用來驗證使用者登入的需求。 若要這麼做，請在內部部署網域控制站中，將下列登錄機碼設為 **1**。 如果`DWORD`不存在,則可以在**Lsa**節點下創建它。

   `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\IgnoreGCFailures`

   有關詳細資訊,請參閱[全域目錄的工作原理](/previous-versions/windows/it-pro/windows-server-2003/cc737410(v=ws.10))。

### <a name="dns-and-domain-controller-on-different-machines"></a>不同電腦上的 DNS 和網域控制站

如果您在相同的 VM 上執行網域控制站和 DNS，您可以略過此程序。

如果 DNS 與網域控制站不在相同的 VM 上，您必須為測試容錯移轉建立 DNS VM。 您可以使用全新的 DNS 伺服器，並建立所有的必要區域。 例如,如果活動目錄域為`contoso.com`,則可以創建名稱的 DNS`contoso.com`區域。 DNS 中對應至 Active Directory 的項目必須更新，如下所示：

1. 確定在復原計劃中的任何其他虛擬機器啟動之前，下列設定均已完成：

   - 區域必須在樹系根名稱之後命名。
   - 區域必須是檔案備份的。
   - 區域必須能夠進行安全和非安全更新。
   - 裝載網域控制站之虛擬機器的解析程式應指向 DNS 虛擬機器的 IP 位址。

1. 在裝載網域控制站的 VM 上執行下列命令：

   `nltest /dsregdns`

1. 執行下列命令，以在 DNS 伺服器上新增區域、允許非安全更新，以及將適用於該區域的項目新增至 DNS：

   ```cmd
   dnscmd /zoneadd contoso.com  /Primary

   dnscmd /recordadd contoso.com  contoso.com. SOA %computername%.contoso.com. hostmaster. 1 15 10 1 1

   dnscmd /recordadd contoso.com %computername%  A <IP_OF_DNS_VM>

   dnscmd /config contoso.com /allowupdate 1
   ```

## <a name="next-steps"></a>後續步驟

[詳細瞭解](site-recovery-workload.md)使用 Azure 網站恢復保護企業工作負荷。

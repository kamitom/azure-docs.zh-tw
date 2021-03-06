---
title: 不使用 Azure 代理程式重設本機 Windows 密碼 | Microsoft Docs
description: 在 Azure 客體代理程式未安裝或運作於 VM 的情況下，如何重設本機 Windows 使用者帳戶的密碼
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.assetid: cf353dd3-89c9-47f6-a449-f874f0957013
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: genli
ms.openlocfilehash: becbf88aeda164f7d916cbc1f1ace89262cc1a3f
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "77921618"
---
# <a name="reset-local-windows-password-for-azure-vm-offline"></a>重設離線 Azure VM 的本機 Windows 密碼
您可以使用 [Azure 入口網站或 Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 在 Azure 中重設 VM 的本機 Windows 密碼 (假設已安裝 Azure 客體代理程式)。 這個方法是為 Azure VM 重設密碼的主要方式。 如果您遇到 Azure 客體代理程式沒有回應，或無法在上傳自訂映像後進行安裝等問題，您可以手動重設 Windows 密碼。 本文將詳細說明如何將來源 OS 虛擬磁碟連接至另一部 VM，以重設本機帳戶密碼。 本文中所述的步驟不適用於 Windows 網域控制站。 

> [!WARNING]
> 只能使用此程序做為最後手段。 一律先嘗試使用 [Azure 入口網站或 Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 重設密碼。

## <a name="overview-of-the-process"></a>程序概觀
無法存取 Azure 客體代理程式時，在 Azure 中為 Windows VM 執行本機密碼重設的核心步驟如下所示︰

1. 停止受影響的 VM。
1. 為 VM 的 OS 磁片創建快照。
1. 從快照創建 OS 磁片的副本。
1. 將複製的作業系統磁片附加並裝載到另一個 Windows VM，然後在磁片上創建一些設定檔。 這些檔將説明您重置密碼。
1. 卸載複製的 OS 磁片並從故障排除 VM 中分離。
1. 將作業系統磁片交換為受影響的 VM。

## <a name="detailed-steps-for-the-vm-with-resource-manager-deployment"></a>部署資源管理器的 VM 的詳細步驟

> [!NOTE]
> 這些步驟不適用於 Windows 網域控制站。 僅適用於獨立伺服器或網域成員伺服器。

一律先嘗試使用 [Azure 入口網站或 Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 重設密碼，再嘗試下列步驟。 在開始之前，確定您有 VM 的備份。

1. 拍攝受影響 VM 的 OS 磁片快照，從快照創建磁片，然後將磁片附加到疑難排解 VM。 有關詳細資訊，請參閱使用[Azure 門戶將 OS 磁片附加到恢復 VM 來排除 Windows VM 的疑難排解](troubleshoot-recovery-disks-portal-windows.md)。
2. 使用遠端桌面連線到故障排除 VM。
3. 在來源 VM 磁碟機的 `\Windows\System32\GroupPolicy` 中建立 `gpt.ini` (如果存在 gpt.ini，請將它重新命名為 gpt.ini.bak)︰
   
   > [!WARNING]
   > 確保您不會在 C:\Windows (疑難排解 VM 的 OS 磁碟機) 中不小心建立下列檔案。 在連接成為資料磁碟的來源 VM OS 磁碟機中建立下列檔案。
   
   * 將下列幾行新增至您建立的 `gpt.ini` 檔案：
     
     ```
     [General]
     gPCFunctionalityVersion=2
     gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
     Version=1
     ```
     
     ![建立 gpt.ini](./media/reset-local-password-without-agent/create-gpt-ini.png)

4. 在 `\Windows\System32\GroupPolicy\Machines\Scripts\` 中建立 `scripts.ini`。 確定已顯示隱藏的資料夾。 如有需要，請建立 `Machine` 或 `Scripts` 資料夾。
   
   * 將下列幾行新增至您建立的 `scripts.ini` 檔案：
     
     ```
     [Startup]
     0CmdLine=C:\Windows\System32\FixAzureVM.cmd
     0Parameters=
     ```
     
     ![建立 scripts.ini](./media/reset-local-password-without-agent/create-scripts-ini.png)

5. 使用下列內容在 `\Windows\System32` 中建立 `FixAzureVM.cmd`，並以您自己的值取代 `<username>` 和 `<newpassword>`：
   
    ```
    net user <username> <newpassword> /add
    net localgroup administrators <username> /add
    net localgroup "remote desktop users" <username> /add
    ```

    ![建立 FixAzureVM.cmd](./media/reset-local-password-without-agent/create-fixazure-cmd.png)
   
    定義新的密碼時，必須符合針對 VM 設定的密碼複雜性需求。

6. 在 Azure 門戶中，從故障排除 VM 中分離磁片。

7. [更改受影響的 VM 的作業系統磁片](troubleshoot-recovery-disks-portal-windows.md#swap-the-os-disk-for-the-vm)。

8. 執行新的 VM 後，使用您在 `FixAzureVM.cmd` 指令碼指定中的新密碼，透過遠端桌面連接到 VM。

9. 從新 VM 的遠端工作階段，移除下列檔案以清理環境︰
    
    * 從 %windir%\System32
      * 移除 FixAzureVM.cmd
    * 從 %windir%*系統32\群組原則\機器\腳本
      * 移除 scripts.ini
    * 從 %windir%\System32\GroupPolicy
      * 移除 gpt.ini (如果 gpt.ini 早已存在，且您已將它重新命名為 gpt.ini.bak，請將此 .bak 檔案重新命名為 gpt.ini)

## <a name="detailed-steps-for-classic-vm"></a>經典 VM 的詳細步驟

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

> [!NOTE]
> 這些步驟不適用於 Windows 網域控制站。 僅適用於獨立伺服器或網域成員伺服器。

一律先嘗試使用 [Azure 入口網站或 Azure PowerShell](https://docs.microsoft.com/previous-versions/azure/virtual-machines/windows/classic/reset-rdp) 重設密碼，再嘗試下列步驟。 在開始之前，確定您有 VM 的備份。 

1. 在 Azure 入口網站中刪除受影響的 VM。 刪除 VM 只會刪除中繼資料 (Azure 內 VM 的參考)。 刪除 VM 時會保留虛擬磁碟：
   
   * 選擇 Azure 門戶中的 VM，然後按一下"*刪除*" ：
     
     ![刪除現有的 VM](./media/reset-local-password-without-agent/delete-vm-classic.png)

2. 將來源 VM 的 OS 磁碟連接到疑難排解 VM。 疑難排解 VM 必須位於與來源 VM 的作業系統磁碟相同的區域 (例如 `West US`)：
   
   1. 在 Azure 入口網站中選取疑難排解 VM。 按一下*磁片* | *附加現有*：
     
      ![連接現有磁碟](./media/reset-local-password-without-agent/disks-attach-existing-classic.png)
     
   2. 選取 [VHD 檔案]**，然後選取包含來源 VM 的儲存體帳戶：
     
      ![選取儲存體帳戶](./media/reset-local-password-without-agent/disks-select-storage-account-classic.png)
     
   3. 選中標記為 *"顯示經典存儲帳戶*"的框，然後選擇源容器。 來源容器通常是 vhd**：
     
      ![選取儲存體容器](./media/reset-local-password-without-agent/disks-select-container-classic.png)

      ![選取儲存體容器](./media/reset-local-password-without-agent/disks-select-container-vhds-classic.png)
     
   4. 選取要連接的 OS vhd。 按一下 [選取]**，完成此程序：
     
      ![選取來源虛擬磁碟](./media/reset-local-password-without-agent/disks-select-source-vhd-classic.png)

   5. 按一下"確定"以連接磁片

      ![連接現有磁碟](./media/reset-local-password-without-agent/disks-attach-okay-classic.png)

3. 使用遠端桌面連接到疑難排解 VM，並確定看得見來源 VM 的 OS 磁碟︰

   1. 在 Azure 入口網站中選取疑難排解 VM，然後按一下 [連接]**。

   2. 開啟下載的 RDP 檔案。 輸入疑難排解 VM 的使用者名稱和密碼。

   3. 在檔案總管中，尋找您所連接的資料磁碟。 如果來源 VM 的 VHD 是連接到疑難排解 VM 的唯一資料磁碟，則應該是 F: 磁碟機︰
     
      ![檢視連接的資料磁碟](./media/reset-local-password-without-agent/troubleshooting-vm-file-explorer-classic.png)

4. 在`gpt.ini``\Windows\System32\GroupPolicy`源 VM 的磁碟機上創建（如果存在`gpt.ini`，請重命名為`gpt.ini.bak`）：
   
   > [!WARNING]
   > 確保不會在 中的 OS 磁碟機中`C:\Windows`意外創建以下檔，以便進行故障排除 VM。 在連接成為資料磁碟的來源 VM OS 磁碟機中建立下列檔案。
   
   * 將下列幾行新增至您建立的 `gpt.ini` 檔案：
     
     ```
     [General]
     gPCFunctionalityVersion=2
     gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
     Version=1
     ```
     
     ![建立 gpt.ini](./media/reset-local-password-without-agent/create-gpt-ini-classic.png)

5. 在 `\Windows\System32\GroupPolicy\Machines\Scripts\` 中建立 `scripts.ini`。 確定已顯示隱藏的資料夾。 如有需要，請建立 `Machine` 或 `Scripts` 資料夾。
   
   * 將下列幾行新增至您建立的 `scripts.ini` 檔案：

     ```
     [Startup]
     0CmdLine=C:\Windows\System32\FixAzureVM.cmd
     0Parameters=
     ```
     
     ![建立 scripts.ini](./media/reset-local-password-without-agent/create-scripts-ini-classic.png)

6. 使用下列內容在 `\Windows\System32` 中建立 `FixAzureVM.cmd`，並以您自己的值取代 `<username>` 和 `<newpassword>`：
   
    ```
    net user <username> <newpassword> /add
    net localgroup administrators <username> /add
    net localgroup "remote desktop users" <username> /add
    ```

    ![建立 FixAzureVM.cmd](./media/reset-local-password-without-agent/create-fixazure-cmd-classic.png)
   
    定義新的密碼時，必須符合針對 VM 設定的密碼複雜性需求。

7. 在 Azure 入口網站中，從疑難排解 VM 卸離磁碟：
   
   1. 在 Azure 入口網站中選取疑難排解 VM，按一下 [磁碟]**。
   
   2. 選擇步驟 2 中附加的資料磁片，按一下 **"分離**"，然後按一下 **"確定**"。

     ![卸離磁碟](./media/reset-local-password-without-agent/data-disks-classic.png)
     
     ![卸離磁碟](./media/reset-local-password-without-agent/detach-disk-classic.png)

8. 從來源 VM 的 OS 磁碟建立 VM：
   
     ![從範本建立 VM](./media/reset-local-password-without-agent/create-new-vm-from-template-classic.png)

     ![從範本建立 VM](./media/reset-local-password-without-agent/choose-subscription-classic.png)

     ![從範本建立 VM](./media/reset-local-password-without-agent/create-vm-classic.png)

## <a name="complete-the-create-virtual-machine-experience"></a>完成創建虛擬機器體驗

1. 執行新的 VM 後，使用您在 `FixAzureVM.cmd` 指令碼指定中的新密碼，透過遠端桌面連接到 VM。

2. 從新 VM 的遠端工作階段，移除下列檔案以清理環境︰
    
    * 從 `%windir%\System32`
      * 刪除`FixAzureVM.cmd`
    * 從 `%windir%\System32\GroupPolicy\Machine\Scripts`
      * 刪除`scripts.ini`
    * 從 `%windir%\System32\GroupPolicy`
      * 刪除`gpt.ini`（如果`gpt.ini`以前存在，並將其重命名為`gpt.ini.bak`，將`.bak`檔重命名回`gpt.ini`）

## <a name="next-steps"></a>後續步驟
如果您仍然無法使用遠端桌面進行連接，請參閱 [RDP 疑難排解指南](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 [詳細的 RDP 疑難排解指南](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)會探討疑難排解方法，而不是特定的步驟。 您也可以[開啟 Azure 支援要求](https://azure.microsoft.com/support/options/)，以取得實際操作協助。

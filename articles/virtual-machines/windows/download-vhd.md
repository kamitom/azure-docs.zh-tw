---
title: 從 Azure 下載 Windows VHD
description: 使用 Azure 入口網站來下載 Windows VHD。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 01/13/2019
ms.author: cynthn
ms.openlocfilehash: d1c98fa4f3572c40279978d787b1719746478a06
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "75940441"
---
# <a name="download-a-windows-vhd-from-azure"></a>從 Azure 下載 Windows VHD

本文說明如何使用 Azure 入口網站，從 Azure 下載 Windows 虛擬硬碟 (VHD) 檔案。

## <a name="optional-generalize-the-vm"></a>可選：通用 VM

如果要使用 VHD 作為[映射](tutorial-custom-images.md)來創建其他 VM，則應使用[Sysprep](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation)對作業系統進行通用化。 

要使用 VHD 作為映射來創建其他 VM，請對 VM 進行一般化。

1. 如果尚未這樣做，請登錄到[Azure 門戶](https://portal.azure.com/)。
2. [連接到 VM](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 
3. 在 VM 上，以系統管理員身分開啟 [命令提示字元] 視窗。
4. 切換至 *%windir%\system32\sysprep* 目錄並執行 sysprep.exe。
5. 在 [系統準備工具] 對話方塊中，選取 [進入系統全新體驗 (OOBE)]****，並確認已選取 [一般化]****。
6. 在"關機選項"中，選擇 **"關機**"，然後按一下"**確定**"。 


## <a name="stop-the-vm"></a>停止 VM

如果 VHD 連接至執行中的 VM，便無法從 Azure 下載該 VHD。 您必須先停止 VM，才能下載 VHD。 

1. 在 Azure 入口網站的中樞功能表中，按一下 [虛擬機器]****。
1. 從清單中選取 VM。
1. 在 VM 的刀鋒視窗中，按一下 [停止]****。


## <a name="generate-download-url"></a>生成下載 URL

要下載 VHD 檔，您需要生成[共用訪問簽名 （SAS）](../../storage/common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) URL。 產生 URL 時，會將到期時間指派給 URL。

1. 在 VM 的頁面上，按一下左側功能表中的 **"磁片**"。
1. 選擇 VM 的作業系統磁片。
1. 在磁片的頁面上，從左側功能表中選擇 **"磁片匯出**"。
1. URL 的預設過期時間是*3600*秒。 將 Windows OS 磁片的此數增加到**36000。**
1. 按一下 [產生 URL]****。

> [!NOTE]
> 到期時間會從預設設定增加，以提供足夠的時間來下載 Windows Server 作業系統的大型 VHD 檔案。 根據您的連線速度而定，包含 Windows Server 作業系統的 VHD 檔案可能會花數小時的時間下載。 如果您正在下載資料磁碟的 VHD，預設的時間便已足夠。 
> 
> 

## <a name="download-vhd"></a>下載 VHD

1. 在產生的 URL 之下，按一下 [下載 VHD 檔案]。
1. 您可能需要在瀏覽器中按一下 **"保存**"即可開始下載。 VHD 檔案的預設名稱是 *abcd*。

## <a name="next-steps"></a>後續步驟

- 了解如何[將 VHD 檔案上傳至 Azure](upload-generalized-managed.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 
- [從存儲帳戶中的非託管磁片創建託管磁片](attach-disk-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
- [使用 PowerShell 管理 Azure 磁碟](tutorial-manage-data-disk.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。


---
title: 從上傳的通用 VHD 創建 VM
description: 在 Resource Manager 部署模型中，將一般化 VHD 上傳至 Azure 並使用它來建立新 VM。
services: virtual-machines-windows
author: cynthn
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: article
ms.date: 12/12/2019
ms.author: cynthn
ms.openlocfilehash: 3c482caf2407c89ffdb6c55c9184c31e2e3197c4
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "75464945"
---
# <a name="upload-a-generalized-vhd-and-use-it-to-create-new-vms-in-azure"></a>將一般化 VHD 上傳，並使用它在 Azure 中建立新的 VM

本文會逐步引導您使用 PowerShell 將一般化 VM 的 VHD 上傳至 Azure，從 VHD 建立映像和從該映像建立新的 VM。 您可以上傳從內部部署虛擬化工具或另一個雲端匯出的 VHD。 針對新的 VM 使用[受控磁碟](managed-disks-overview.md)可簡化 VM 管理，當 VM 中包含可用性設定組時，還可提供更高的可用性。 

如需範例指令碼，請參閱[用來將 VHD 上傳至 Azure 並建立新 VM 的範例指令碼](../scripts/virtual-machines-windows-powershell-upload-generalized-script.md)。

## <a name="before-you-begin"></a>開始之前

- 將任何 VHD 上傳至 Azure 之前，您應該遵循[準備 Windows VHD 或 VHDX 以上傳至 Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
- 請先檢閱[規劃移轉至受控磁碟](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks)，再開始移轉至[受控磁碟](managed-disks-overview.md)。

 
## <a name="generalize-the-source-vm-by-using-sysprep"></a>使用 Sysprep 將來源 VM 一般化

如果尚未將 VHD 上載到 Azure 之前，需要先對 VM 進行系統設置。 Sysprep 會移除您的所有個人帳戶資訊以及其他項目，並準備電腦以做為映像。 如需 Sysprep 的詳細資訊，請參閱 [Sysprep 概觀](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview)。

請確定 Sysprep 支援電腦上執行的伺服器角色。 如需詳細資訊，請參閱 [Sysprep Support for Server Roles (伺服器角色的 Sysprep 支援)](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)。

> [!IMPORTANT]
> 如果您打算在第一次將 VHD 上傳至 Azure 之前，先執行 Sysprep，請確定您已[準備好 VM](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 
> 
> 

1. 登入 Windows 虛擬機器。
2. 以系統管理員身分開啟 [命令提示字元] 視窗。 將目錄變更到 %windir%\system32\sysprep，然後執行 `sysprep.exe`。
3. 在 [系統準備工具]**** 對話方塊中，選取 [進入系統全新體驗 (OOBE)]****，並確認已啟用 [一般化]**** 核取方塊。
4. 針對 [關機選項]****，選取 [關機]****。
5. 選取 [確定]****。
   
    ![啟動 Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
6. Sysprep 完成時，會關閉虛擬機器。 不要重新啟動 VM。


## <a name="upload-the-vhd"></a>上傳 VHD 

您現在可以將 VHD 直接上載到託管磁片。 有關說明，請參閱[使用 Azure PowerShell 將 VHD 上載到 Azure。](disks-upload-vhd-to-managed-disk-powershell.md)



將 VHD 上載到託管磁片後，需要使用[Get-AzDisk](https://docs.microsoft.com/powershell/module/az.compute/get-azdisk)獲取託管磁片。

```azurepowershell-interactive
$disk = Get-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'myDiskName'
```

## <a name="create-the-image"></a>建立映像
從通用作業系統託管磁片創建託管映射。 使用您自己的資訊取代下列值。

首先，設置一些變數：

```powershell
$location = 'East US'
$imageName = 'myImage'
$rgName = 'myResourceGroup'
```

使用託管磁片創建映射。

```azurepowershell-interactive
$imageConfig = New-AzImageConfig `
   -Location $location
$imageConfig = Set-AzImageOsDisk `
   -Image $imageConfig `
   -OsState Generalized `
   -OsType Windows `
   -ManagedDiskId $disk.Id
```

建立映像。

```azurepowershell-interactive
$image = New-AzImage `
   -ImageName $imageName `
   -ResourceGroupName $rgName `
   -Image $imageConfig
```

## <a name="create-the-vm"></a>建立 VM

現在您已有映像，您可以從映像建立一個或多個新的 VM。 該範例會根據 *myResourceGroup* 中的 *myImage* 建立名為 *myVM* 的 VM。


```powershell
New-AzVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Image $image.Id `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNSG" `
    -PublicIpAddressName "myPIP" `
    -OpenPorts 3389
```


## <a name="next-steps"></a>後續步驟

登入至新的虛擬機器。 如需詳細資訊，請參閱 [如何連接和登入執行 Windows 的 Azure 虛擬機器](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 


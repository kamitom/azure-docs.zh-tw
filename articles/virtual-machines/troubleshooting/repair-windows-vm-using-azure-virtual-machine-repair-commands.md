---
title: 使用 Azure 虛擬機器修復命令修復 Windows VM |微軟文檔
description: 本文詳細介紹了如何使用 Azure VM 修復命令將磁片連接到其他 Windows VM 以修復任何錯誤，然後重建原始 VM。
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: virtual-machines
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 09/10/2019
ms.author: v-miegge
ms.openlocfilehash: 2055558ef80a641084a7cf9d299281497d282936
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80060680"
---
# <a name="repair-a-windows-vm-by-using-the-azure-virtual-machine-repair-commands"></a>使用 Azure 虛擬機器修復命令修復 Windows VM

如果 Azure 中的 Windows 虛擬機器 （VM） 遇到啟動或磁片錯誤，則可能需要對磁片本身執行緩解。 常見的例子是應用程式更新無效，導致 VM 無法成功開機。 本文詳細介紹了如何使用 Azure VM 修復命令將磁片連接到其他 Windows VM 以修復任何錯誤，然後重建原始 VM。

> [!IMPORTANT]
> 本文中的腳本僅適用于使用[Azure 資源管理器](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)的 VM。

## <a name="repair-process-overview"></a>維修流程概述

現在可以使用 Azure VM 修復命令來更改 VM 的作業系統磁片，並且不再需要刪除和重新創建 VM。

按照以下步驟解決 VM 問題：

1. 啟動 Azure Cloud Shell
2. 運行 az 擴展添加/更新。
3. 運行 az vm 修復創建。
4. 運行 az vm 修復運行。
5. 運行 az vm 修復還原。

有關其他文檔和說明，請參閱[az vm 修復](https://docs.microsoft.com/cli/azure/ext/vm-repair/vm/repair)。

## <a name="repair-process-example"></a>修復過程示例

> [!NOTE]
> * 運行腳本需要來自 VM（埠 443）的出站連接。
> * 一次只能運行一個腳本。
> * 無法取消正在運行的腳本。
> * 腳本可以運行的最大時間是 90 分鐘，之後它將超時。

1. 啟動 Azure Cloud Shell

   Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它包括預先安裝並配置為與帳戶一起使用的常見 Azure 工具。

   要打開雲外殼，請選擇從代碼塊的右上角**嘗試它**。 您還可以通過訪問[https://shell.azure.com](https://shell.azure.com)在單獨的瀏覽器選項卡中打開雲外殼。

   選擇 **"複製"** 以複製代碼塊，然後將代碼粘貼到雲外殼中，然後選擇 **"輸入**"以運行它。

   如果您偏好在本機安裝和使用 CLI，本快速入門需要有 Azure CLI 2.0.30 版或更新版本。 執行 ``az --version`` 以尋找版本。 如果需要安裝或升級 Azure CLI，請參閱安裝[Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。

2. 如果這是您第一次使用這些命令，`az vm repair`則添加 vm 修復 CLI 擴展。

   ```azurecli-interactive
   az extension add -n vm-repair
   ```

   如果以前使用過這些命令，`az vm repair`請對 vm 修復擴展應用任何更新。

   ```azurecli-interactive
   az extension update -n vm-repair
   ```

3. 執行 `az vm repair create`。 此命令將為非功能 VM 創建 OS 磁片的副本，創建修復 VM 並附加磁片。

   ```azurecli-interactive
   az vm repair create -g MyResourceGroup -n myVM --repair-username username --repair-password password!234 --verbose
   ```

4. 執行 `az vm repair run`。 此命令將通過修復 VM 在連接的磁片上運行指定的修復腳本。

   ```azurecli-interactive
   az vm repair run  –g MyResourceGroup –n MyVM -–run-on-repair --run-id 2 --verbose
   ```

5. 執行 `az vm repair restore`。 此命令將修復的 OS 磁片與 VM 的原始 OS 磁片交換。

   ```azurecli-interactive
   az vm repair restore -g MyResourceGroup -n MyVM --verbose
   ```

## <a name="verify-and-enable-boot-diagnostics"></a>確認並啟用開機診斷

下列範例會在資源群組 ``myResourceGroup`` 中的 VM ``myVMDeployed`` 上啟用診斷擴充：

Azure CLI

```azurecli-interactive
az vm boot-diagnostics enable --name myVMDeployed --resource-group myResourceGroup --storage https://mystor.blob.core.windows.net/
```

## <a name="next-steps"></a>後續步驟

* 如果連接至 VM 時發生問題，請參閱[針對 Azure VM 的 RDP 連接進行疑難排解](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/troubleshoot-rdp-connection)。
* 有關訪問在 VM 上運行的應用程式的問題，請參閱[在 Azure 中解決虛擬機器上的應用程式連接問題](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/troubleshoot-app-connection)。
* 如需使用 Resource Manager 的詳細資訊，請參閱 [Azure Resource Manager 概觀](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。

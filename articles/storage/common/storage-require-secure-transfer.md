---
title: 需要安全傳輸以確保安全連線
titleSuffix: Azure Storage
description: 瞭解如何要求安全傳輸請求到 Azure 存儲。 當您需要存儲帳戶的安全轉移時，來自不安全連線的任何請求都將被拒絕。
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/12/2019
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.openlocfilehash: 3b2d78bd929e23d49a57f337022f6678114bb5fe
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "75457438"
---
# <a name="require-secure-transfer-to-ensure-secure-connections"></a>需要安全傳輸以確保安全連線

您可以通過設置存儲帳戶**的安全轉移要求**屬性來配置存儲帳戶以接受來自安全連線的請求。 當您需要安全傳輸時，來自不安全連線的任何請求都被拒絕。 Microsoft 建議您始終需要安全轉移所有存儲帳戶。

當需要安全傳輸時，必須通過 HTTPS 調用 Azure 存儲 REST API 操作。 通過 HTTP 提出的任何請求都被拒絕。

當存儲帳戶需要安全傳輸時，通過 SMB 連接到 Azure 檔共用而不進行加密失敗。 不安全連線的示例包括通過 SMB 2.1、SMB 3.0 未加密或某些版本的 Linux SMB 用戶端建立的連接。

預設情況下，在 Azure 門戶中創建存儲帳戶時啟用**了所需的安全傳輸**屬性。 但是，當您使用 SDK 創建存儲帳戶時，它將被禁用。

> [!NOTE]
> 因為 Azure 儲存體針對自訂網域名稱並不支援 HTTPS，當您使用自訂網域名稱時，將不會套用此選項。 不支援傳統儲存體帳戶。

## <a name="require-secure-transfer-in-the-azure-portal"></a>需要在 Azure 門戶中安全傳輸

在[Azure 門戶](https://portal.azure.com)中創建存儲帳戶時，可以打開 **"安全轉移"必需**屬性。 您也可以為現有的儲存體帳戶啟用它。

### <a name="require-secure-transfer-for-a-new-storage-account"></a>針對新的儲存體帳戶要求使用安全傳輸

1. 在 Azure 入口網站中開啟 [建立儲存體帳戶]**** 窗格。
1. 在 [需要安全傳輸]**** 下，選取 [已啟用]****。

   ![[建立儲存體帳戶] 刀鋒視窗](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_1.png)

### <a name="require-secure-transfer-for-an-existing-storage-account"></a>針對現有儲存體帳戶要求使用安全傳輸

1. 在 Azure 入口網站中選取現有儲存體帳戶。
1. 在 [儲存體帳戶] 功能表窗格中，選取 [設定]**** 底下的 [組態]****。
1. 在 [需要安全傳輸]**** 下，選取 [已啟用]****。

   ![[儲存體帳戶] 功能表窗格](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_2.png)

## <a name="require-secure-transfer-from-code"></a>需要從代碼安全傳輸

要以程式設計方式要求安全轉移，請設置存儲帳戶上的_支援 HttpsTrafficOnly_屬性。 可以使用存儲資來源提供者 REST API、用戶端庫或工具設置此屬性：

* [REST API](/rest/api/storagerp/storageaccounts)
* [電源外殼](/powershell/module/az.storage/set-azstorageaccount)
* [Cli](/cli/azure/storage/account)
* [NodeJS](https://www.npmjs.com/package/azure-arm-storage/)
* [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage)
* [Python SDK](https://pypi.org/project/azure-mgmt-storage)
* [Ruby SDK](https://rubygems.org/gems/azure_mgmt_storage)

## <a name="require-secure-transfer-with-powershell"></a>需要使用 PowerShell 進行安全傳輸

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

此範例需要 Azure PowerShell 模組 Az 0.7 版或更新版本。 執行 `Get-Module -ListAvailable Az` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-Az-ps)。

執行 `Connect-AzAccount` 來建立與 Azure 的連線。

 使用下列命令列檢查設定：

```powershell
Get-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}"
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : False
...

```

使用下列命令列啟用設定：

```powershell
Set-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}" -EnableHttpsTrafficOnly $True
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : True
...

```

## <a name="require-secure-transfer-with-azure-cli"></a>需要使用 Azure CLI 進行安全傳輸

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 使用以下命令檢查設置：

```azurecli-interactive
az storage account show -g {ResourceGroupName} -n {StorageAccountName}
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": false,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

使用以下命令啟用該設置：

```azurecli-interactive
az storage account update -g {ResourceGroupName} -n {StorageAccountName} --https-only true
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": true,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

## <a name="next-steps"></a>後續步驟

[Blob 存儲的安全建議](../blobs/security-recommendations.md)

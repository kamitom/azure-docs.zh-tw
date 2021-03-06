---
title: 資料加密 ─ Azure 門戶 ─ MySQL 的 Azure 資料庫
description: 瞭解如何使用 Azure 門戶為 MySQL 為 Azure 資料庫設置和管理數據加密。
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: acf3e6273f98d98d5da55cfb5b044677116c44dc
ms.sourcegitcommit: b0ff9c9d760a0426fd1226b909ab943e13ade330
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/01/2020
ms.locfileid: "80520815"
---
# <a name="data-encryption-for-azure-database-for-mysql-by-using-the-azure-portal"></a>通過使用 Azure 門戶,MySQL Azure 資料庫的資料加密

瞭解如何使用 Azure 門戶為 MySQL 的 Azure 資料庫設置和管理數據加密。

## <a name="prerequisites-for-azure-cli"></a>Azure CLI 的先決條件

* 您必須具有 Azure 訂用帳戶，並且是該訂用帳戶的系統管理員。
* 在 Azure 金鑰保管庫中,創建用於客戶管理的密鑰的密鑰保管庫和密鑰。
* 金鑰保存庫必須具有以下屬性才能用作客戶管理的金鑰:
  * [軟刪除](../key-vault/key-vault-ovw-soft-delete.md)

    ```azurecli-interactive
    az resource update --id $(az keyvault show --name \ <key_vault_name> -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
    ```

  * [清除受保護](../key-vault/key-vault-ovw-soft-delete.md#purge-protection)

    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
    ```

* 金鑰必須具有以下屬性才能用作客戶管理的金鑰:
  * 沒有到期日
  * 未停用
  * 能夠執行取得、包裝金鑰、解除包裝金鑰作業

## <a name="set-the-right-permissions-for-key-operations"></a>為金鑰設定正確的權限

1. 在金鑰保存庫中,選擇**存取策略** > **來加入存取原則**。

   ![突出顯示了金鑰保管庫的螢幕截圖,並突出顯示了存取策略和添加存取策略](media/concepts-data-access-and-security-data-encryption/show-access-policy-overview.png)

2. 選擇 **「密鑰權限**」,然後選擇 **「獲取**、**換行**、**解包**」 和 **「主體**」,這是 MySQL 伺服器的名稱。 如果在現有主體清單中找不到伺服器主體,則需要註冊它。 首次嘗試設置數據加密時,系統會提示您註冊伺服器主體,但該加密失敗。

   ![存取策略概述](media/concepts-data-access-and-security-data-encryption/access-policy-wrap-unwrap.png)

3. 選取 [儲存]  。

## <a name="set-data-encryption-for-azure-database-for-mysql"></a>為 MySQL 為 Azure 資料庫設定資料加密

1. 在 MySQL 的 Azure 資料庫中,選擇**數據加密**以設置客戶管理的密鑰。

   ![MySQL Azure 資料庫的螢幕截圖,並突出顯示資料加密](media/concepts-data-access-and-security-data-encryption/data-encryption-overview.png)

2. 您可以選擇金鑰保管庫和密鑰對,也可以輸入金鑰標識符。

   ![MySQL Azure 資料庫的螢幕截圖,突顯資料加密選項](media/concepts-data-access-and-security-data-encryption/setting-data-encryption.png)

3. 選取 [儲存]  。

4. 為確保所有檔(包括暫時檔案)完全加密,請重新啟動伺服器。

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>對回復伺服器或複本伺服器使用資料加密

使用存儲在金鑰保管庫中的客戶託管金鑰對MySQL的Azure資料庫進行加密後,還將加密任何新創建的伺服器副本。 您可以通過本地或異地還原操作或通過副本(本地/跨區域)操作製作此新副本。 因此,對於加密的 MySQL 伺服器,可以使用以下步驟創建加密還原的伺服器。

1. 在伺服器上,選擇 **「概述** > **還原**」。。

   ![MySQL Azure 資料庫的螢幕截圖,突出顯示了概述和還原](media/concepts-data-access-and-security-data-encryption/show-restore.png)

   或者,對於啟用複製的伺服器,在 **"設置"** 標題下,選擇 **"複製**"。

   ![MySQL Azure 資料庫的螢幕截圖,並突出顯示複製](media/concepts-data-access-and-security-data-encryption/mysql-replica.png)

2. 還原操作完成後,創建的新伺服器將使用主伺服器的密鑰進行加密。 但是,伺服器上的功能和選項被禁用,並且無法訪問伺服器。 這可以防止任何數據操作,因為新伺服器的標識尚未被授予訪問密鑰保管庫的許可權。

   ![MySQL Azure 資料庫的螢幕截圖,突顯不可存取狀態](media/concepts-data-access-and-security-data-encryption/show-restore-data-encryption.png)

3. 要使伺服器可訪問,請重新驗證還原伺服器上的密鑰。 選擇**資料加密** > **重新驗證金鑰**。

   > [!NOTE]
   > 第一次重新驗證的嘗試將失敗,因為需要授予新伺服器的服務主體對密鑰保管庫的訪問許可權。 要生成服務主體,請選擇 **「重新驗證密鑰**」,該鍵將顯示錯誤,但會生成服務主體。 之後,請參閱[本文前面的步驟](#set-the-right-permissions-for-key-operations)。

   ![MySQL Azure 資料庫的螢幕截圖,重新驗證步驟突出顯示](media/concepts-data-access-and-security-data-encryption/show-revalidate-data-encryption.png)

   您必須授予金鑰保管庫對新伺服器的訪問許可權。

4. 註冊服務主體后,再次重新驗證密鑰,伺服器將恢復其正常功能。

   ![MySQL Azure 資料庫的螢幕截圖,顯示還原的功能](media/concepts-data-access-and-security-data-encryption/restore-successful.png)

## <a name="next-steps"></a>後續步驟

 要瞭解有關資料加密的詳細資訊,請參閱[使用客戶管理的金鑰進行 MySQL 資料加密的 Azure 資料庫](concepts-data-encryption-mysql.md)。

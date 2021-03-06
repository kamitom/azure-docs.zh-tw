---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 760bb5b62e9bba9b7a83f99760f7fe5d8c399dfb
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "67173930"
---
1. 在容錯移轉叢集管理員中，展開 [角色]****，然後醒目提示您的可用性群組。  

2. 在 [資源]**** 索引標籤上，以滑鼠右鍵按一下接聽程式名稱，然後按一下 [屬性]****。

3. 按一下 **"依賴項"** 選項卡。如果列出了多個資源，請驗證 IP 位址是否具有 OR，而不是 AND。  

4. 按一下 [確定]****。

5. 以滑鼠右鍵按一下接聽程式名稱，然後按一下 [上線]****。

6. 接聽程式上線後，在 [資源]**** 索引標籤上，以滑鼠右鍵按一下可用性群組，然後按一下 [屬性]****。
   
    ![設定可用性群組資源](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678772.gif)

7. 建立對接聽程式名稱資源 (非 IP 位址資源名稱) 的相依性，然後按一下 [確定]****。
   
    ![新增對接聽程式名稱的相依性](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678773.gif)

8. 啟動 SQL Server Management Studio，然後連線到主要複本。

9. 轉到 **"始終打開高** > **可用性組** > **\<可用性組可用性組\>名稱** > **可用性組攔截器**"。  
    您在容錯移轉叢集管理員中建立的接聽程式名稱應會顯示。

10. 以滑鼠右鍵按一下接聽程式名稱，然後按一下 [屬性]****。

11. 在 [連接埠]**** 方塊中，使用您稍早所用的 $EndpointPort 來指定可用性群組接聽程式的連接埠號碼 (在本教學課程中，預設值為 1433)，然後按一下 [確定]****。


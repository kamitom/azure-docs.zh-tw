---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 11/06/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 729e757c69887bbdce324e2d8383c970995dc94a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "73903647"
---
## <a name="sign-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 登入 Azure 入口網站。

> [!NOTE]
> 如果在預覽期間註冊使用共用圖像庫，則可能需要重新註冊`Microsoft.Compute`提供程式。 開放[雲外殼](https://shell.azure.com/bash)和類型：`az provider register -n Microsoft.Compute`

## <a name="create-an-image-gallery"></a>建立映像資源庫

映像資源庫是用於啟用映像共用的主要資源。 資源庫名稱允許的字元為大寫或小寫字母、數字、點和句點。 庫名稱不能包含破折號。  資源庫名稱在您的訂用帳戶內必須是唯一的。 

下列範例會在 *myGalleryRG* 資源群組中建立名為 *myGallery* 的資源庫。

1. 選取 Azure 入口網站左上角的 [建立資源]****。
1. 在搜索框中使用"**共用圖像庫**"類型，並在結果中選擇 **"共用圖像庫**"。
1. 在 **"共用圖像庫**"頁中，按一下"**創建**"。
1. 選擇正確的訂閱。
1. 在 **"資源"組中**，選擇 **"創建新"** 並鍵入名稱的*myGalleryRG。*
1. 在 **"名稱**"*中，鍵入我的庫*以表示庫的名稱。
1. 保留**區域**的預設值。
1. 您可以鍵入庫的簡短說明，如*用於測試的"我的圖像庫"。* 然後按一下 **"審閱 + 創建**"。
1. 驗證通過後，選擇 **"創建**"。
1. 部署完成後，選擇 **"轉到資源**"。
   
## <a name="create-an-image-definition"></a>建立映像定義 

圖像定義為圖像創建邏輯分組。 它們用於管理有關在其中創建的映射版本的資訊。 圖像定義名稱可以由大寫字母或小寫字母、數位、點、破折號和句點組成。 有關可以為圖像定義指定的值的詳細資訊，請參閱[圖像定義](https://docs.microsoft.com/azure/virtual-machines/windows/shared-image-galleries#image-definitions)。

在庫內創建庫圖像定義。 在此示例中，庫圖像名為*myImage定義*。

1. 在新圖像庫的頁面上，選擇從頁面頂部**添加新的圖像定義**。 
1. 對於**圖像定義名稱**，鍵入*myImage 定義*。
1. 對於**作業系統**，根據源 VM 選擇正確的選項。
1. 對於**VM 生成**，根據源 VM 選擇選項。 在大多數情況下，這將是第 1*代*。 有關詳細資訊，請參閱第[2 代 VM 的支援](https://docs.microsoft.com/azure/virtual-machines/windows/generation-2)。
1. 對於**作業系統狀態**，根據源 VM 選擇選項。 有關詳細資訊，請參閱[通用和專用](../articles/virtual-machines/linux/shared-image-galleries.md#generalized-and-specialized-images)。
1. 對於**發行者**，鍵入*我的發行者*。 
1. 對於**優惠**，鍵入*myOffer*。
1. 對於**SKU，** 鍵入*mySKU*。
1. 完成後，選擇 **"審閱 + 創建**"。
1. 圖像定義通過驗證後，選擇 **"創建**"。
1. 部署完成後，選擇 **"轉到資源**"。


## <a name="create-an-image-version"></a>建立映像版本

從託管映射創建映射版本。 在此範例中，映像版本為 *1.0.0*，且它會被複寫到「美國中西部」** 和「美國中南部」** 資料中心。 選擇複製的目的地區域時，請記住，還必須將*源*區域作為複製目標。

映像版本允許的字元是數字及句點。 數字必須在 32 位元整數的範圍內。 格式：*主要版本*。*次要版本*。*補丁*。

創建映射版本的步驟略有不同，具體取決於源是通用映射還是專用 VM 的快照。 

### <a name="option-generalized"></a>選項：通用化

1. 在圖像定義的頁面中，從頁面頂部選擇 **"添加版本**"。
1. 在 **"區域**"中，選擇存儲託管映射的區域。 圖像版本需要在與從其創建的託管映射相同的區域中創建。
1. 對於**名稱**，鍵入*1.0.0*。 圖像版本名稱應遵循*主要*。*次要*。使用整數*的修補程式*格式。 
1. 在**源圖像**中，從下拉清單中選擇源託管圖像。
1. 在 **"從最新排除"** 中，保留預設值 *"否*"。
1. 對於**生命結束日期**，從日曆中選擇一個將來幾個月的日期。
1. 在**複製**中，將**預設副本計數**保留為 1。 您需要複製到源區域，因此將第一個副本保留為預設值，然後選擇第二個副本區域為美國*東部*。
1. 完成後，選擇 **"審閱 + 創建**"。 Azure 將驗證配置。
1. 當映射版本通過驗證時，選擇 **"創建**"。
1. 部署完成後，選擇 **"轉到資源**"。

將圖像複製到所有目的地區域可能需要一段時間。

### <a name="option-specialized"></a>選項： 專業

1. 在圖像定義的頁面中，從頁面頂部選擇 **"添加版本**"。
1. 在 **"區域**"中，選擇快照存儲的區域。 圖像版本需要在與從源創建的源相同的區域中創建。
1. 對於**名稱**，鍵入*1.0.0*。 圖像版本名稱應遵循*主要*。*次要*。使用整數*的修補程式*格式。 
1. 在**OS 磁片快照**中，從下拉清單中從源 VM 中選擇快照。 如果源 VM 具有要包含的資料磁片，請從下拉清單中選擇正確的**LUN**編號，然後為**資料磁片快照選擇資料磁片的快照**。 
1. 在 **"從最新排除"** 中，保留預設值 *"否*"。
1. 對於**生命結束日期**，從日曆中選擇一個將來幾個月的日期。
1. 在**複製**中，將**預設副本計數**保留為 1。 您需要複製到源區域，因此將第一個副本保留為預設值，然後選擇第二個副本區域為美國*東部*。
1. 完成後，選擇 **"審閱 + 創建**"。 Azure 將驗證配置。
1. 當映射版本通過驗證時，選擇 **"創建**"。
1. 部署完成後，選擇 **"轉到資源**"。

## <a name="share-the-gallery"></a>共用庫

我們建議您在圖像庫級別共用存取權限。 下面將引導您分享您剛剛創建的庫。

1. 打開[Azure 門戶](https://portal.azure.com)。
1. 在左側的功能表中，選擇 **"資源組**"。 
1. 在資源組清單中，選擇**myGalleryRG**。 資源組的邊欄選項卡將打開。
1. 在**myGalleryRG**頁面左側的功能表中，選擇**存取控制 （IAM）。** 
1. 在 **"添加角色指派**"下，選擇 **"添加**"。 將打開"**添加角色指派**"窗格。 
1. 在**角色**下，選擇 **"讀取器**"。
1. 在 **"分配對 "的存取權限**下，保留**Azure AD 使用者、組或服務主體**的預設值。
1. 在 **"選擇**"下，鍵入要邀請的人員的電子郵件地址。
1. 如果使用者在組織外部，您將看到消息**此使用者將發送一封電子郵件，使他們能夠與 Microsoft 協作。** 選擇具有電子郵件地址的使用者，然後按一下"**保存**"。

如果使用者不在組織之外，他們將獲得加入組織的電子郵件邀請。 使用者需要接受邀請，然後他們將能夠看到庫及其資源清單中的所有圖像定義和版本。


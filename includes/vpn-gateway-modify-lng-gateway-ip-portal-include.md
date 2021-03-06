---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 52084b065ef65a69a6691b6646d1e199f011910d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "67173814"
---
### <a name="to-modify-the-local-network-gateway-ip-address---no-gateway-connection"></a><a name="gwipnoconnection"></a>修改區域網路閘道 IP 位址 - 沒有閘道連線

使用範例來修改沒有閘道連線的區域網路閘道。 修改此值時，您也可以同時修改位址首碼。

1. 在 [區域網路閘道] 資源的 [設定]**** 區段中，按一下 [組態]****。
2. 在 [IP 位址]**** 方塊中，修改 IP 位址。
3. 按一下 [儲存] **** 來儲存這些設定。

### <a name="to-modify-the-local-network-gateway-ip-address---existing-gateway-connection"></a><a name="gwipwithconnection"></a>修改區域網路閘道 IP 位址 - 現有閘道連線

若要修改具有連線的區域網路閘道，您必須先移除連線。 連線移除之後，您可以修改閘道 IP 位址，然後重新建立新連線。 您也可以同時修改位址首碼。 這會導致您 VPN 連線的停機時間。 修改閘道 IP 位址時，您不需要刪除 VPN 閘道。 您只需要移除連線。
 
#### <a name="1-remove-the-connection"></a>1. 卸下連接。

1. 在 [區域網路閘道] 資源的 [設定]**** 區段中，按一下 [連線]****。
2. 按一下連線那一行上的 [...]****，然後按一下 [刪除]****。
3. 按一下 [Save] **** 儲存您的設定。

#### <a name="2-modify-the-ip-address"></a>2. 修改 IP 位址。

您也可以同時修改位址首碼。

1. 在 [IP 位址]**** 方塊中，修改 IP 位址。
2. 按一下 [儲存] **** 來儲存這些設定。

#### <a name="3-recreate-the-connection"></a>3. 重新創建連接。

1. 瀏覽至您 VNet 的虛擬網路閘道。 (而非區域網路閘道。)
2. 在 [虛擬網路閘道] 的 [設定]**** 區段中，按一下 [連線]****。
3. 按一下 [+新增]**** 以開啟 [新增連線]**** 刀鋒視窗。
4. 重新建立您的連線。
5. 按一下 [確定]**** 來建立連線。
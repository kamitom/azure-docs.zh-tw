---
title: 在 Azure Stack 中部署 OpenShift
description: 在 Azure Stack 中部署 OpenShift。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldwongms
manager: joraio
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: d6c73b8cd33aa85793a2ce839410065e03b97be7
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "74035537"
---
# <a name="deploy-openshift-container-platform-or-okd-in-azure-stack"></a>在 Azure Stack 中部署 OpenShift 容器平台或 OKD

在 Azure Stack 中可以部署 OpenShift。 Azure 與 Azure Stack 之間有一些主要差異，因此部署會有些許不同，而且功能也有些許不同。

目前，Azure 雲端提供者不適用於 Azure Stack。 基於這個理由，您無法使用磁碟連結在 Azure Stack 中永續儲存。 相反，您可以配置其他存儲選項，如 NFS、iSCSI、GlusterFS 等。作為替代方案，您可以啟用 CNS 並使用 GlusterFS 進行持久存儲。 如果已啟用 CN，將會部署額外三個節點，而且有額外的儲存體可供 GlusterFS 使用。

您可以使用數種方法其中之一，在 Azure Stack 中部署 OpenShift 容器平台或 OKS：

- 您可以手動部署必要的 Azure 基礎結構元件，然後依照 [OpenShift 容器平台文件](https://docs.openshift.com/container-platform)或 [OKD 文件](https://docs.okd.io)進行。
- 您也可以使用能夠簡化 OpenShift 容器平台叢集部署的現有 [Resource Manager 範本](https://github.com/Microsoft/openshift-container-platform/) \(英文\)。
- 您也可以使用能夠簡化 OKD 叢集部署的現有 [Resource Manager 範本](https://github.com/Microsoft/openshift-origin)。

如果使用 Resource Manager 範本，請選取適當的分支 (azurestack-release-3.x)。 因為 Azure 與 Azure Stack 之間的 API 版本不同，所以 Azure 範本無法運作。 RHEL 映像參考目前已硬式編碼為 azuredeploy.json 檔案中的變數，而且必須變更以符合您的映像。

```json
"imageReference": {
    "publisher": "Redhat",
    "offer": "RHEL-OCP",
    "sku": "7-4",
    "version": "latest"
}
```

對於所有選項，都需要有 Red Hat 訂用帳戶。 在部署期間，Red Hat Enterprise Linux 執行個體會向 Red Hat 訂用帳戶註冊，並且附加至包含 OpenShift 容器平台之權利的集區識別碼。
請確定您具備有效的 Red Hat Subscription Manager (RHSM) 使用者名稱、密碼和集區識別碼。 或者，您可以使用啟用金鑰、組織識別碼和集區識別碼。  您可以登入 https://access.redhat.com 來驗證這項資訊。

## <a name="azure-stack-prerequisites"></a>Azure Stack 先決條件

RHEL 映像 (OpenShift 容器平台) 或 CentOS 映像 (OKD) 必須新增至 Azure Stack 環境，才能部署 OpenShift 叢集。 請連絡 Azure Stack 系統管理員來新增這些映像。 在此可以找到相關指示：

- https://docs.microsoft.com/azure/azure-stack/azure-stack-add-vm-image
- https://docs.microsoft.com/azure/azure-stack/azure-stack-marketplace-azure-items
- https://docs.microsoft.com/azure/azure-stack/azure-stack-redhat-create-upload-vhd

## <a name="deploy-by-using-the-openshift-container-platform-or-okd-resource-manager-template"></a>使用 OpenShift 容器平台或 OKD Resource Manager 範本部署

若要使用 Resource Manager 範本部署，請使用參數檔案來提供輸入參數。 若要進一步自訂部署，請分支處理 GitHub 存放庫並變更適當的項目。

某些常見的自訂選項包括但不限於：

- 防禦 VM 大小 (azuredeploy.json 中的變數)
- 命名慣例 (azuredeploy.json 中的變數)
- OpenShift 叢集詳細規格，已透過主機檔案 (deployOpenShift.sh) 修改
- RHEL 映像參考 (azuredeploy.json 中的變數)

如需使用 Azure CLI 進行部署的步驟，請依照 [OpenShift 容器平台](./openshift-container-platform-3x.md)一節或 [OKD](./openshift-okd.md) 一節中的適當小節進行。

## <a name="next-steps"></a>後續步驟

- [部署後工作](./openshift-container-platform-3x-post-deployment.md)
- [針對 Azure 中的 OpenShift 部署進行疑難排解](./openshift-container-platform-3x-troubleshooting.md)
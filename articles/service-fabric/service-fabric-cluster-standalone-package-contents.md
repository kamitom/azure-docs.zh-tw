---
title: 適用于 Windows 伺服器的 Azure 服務結構獨立包
description: 適用於 Windows Server 的 Azure Service Fabric 獨立封裝描述和內容。
author: maburlik
ms.topic: conceptual
ms.date: 8/10/2017
ms.author: maburlik
ms.openlocfilehash: 1bb73fa69717f067139067f127a0d50af4878d6c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "75451837"
---
# <a name="contents-of-service-fabric-standalone-package-for-windows-server"></a>適用於 Windows Server 的 Service Fabric 獨立封裝內容
在[下載](https://go.microsoft.com/fwlink/?LinkId=730690)的 Service Fabric 獨立封裝中，您會找到下列檔案︰

| [檔案名稱]**** | **簡短描述** |
| --- | --- |
| CreateServiceFabricCluster.ps1 |使用 ClusterConfig.json 中的設定建立叢集的 PowerShell 指令碼。 |
| RemoveServiceFabricCluster.ps1 |使用 ClusterConfig.json 中的設定移除叢集的 PowerShell 指令碼。 |
| AddNode.ps1 |用於將節點加入至目前電腦上現有部署叢集的 PowerShell 指令碼。 |
| RemoveNode.ps1 |用於將節點從目前電腦中的現有部署叢集移除的 PowerShell 指令碼。 |
| CleanFabric.ps1 |從目前電腦中清除獨立 Service Fabric 安裝的 PowerShell 指令碼。 先前的 MSI 安裝應該以本身相關聯的解除安裝程式來移除。 |
| TestConfiguration.ps1 |分析 Cluster.json 中指定之基礎結構的 PowerShell 指令碼。 |
| DownloadServiceFabricRuntimePackage.ps1 |用於下載最新的頻外執行階段封裝的 PowerShell 指令碼，適用於部署未與網際網路連線的電腦。 |
| DeploymentComponentsAutoextractor.exe |包含獨立封裝指令碼使用之部署元件的自我解壓縮封存。 |
| EULA_ENU.txt |使用 Microsoft Azure Service Fabric 獨立 Windows Server 封裝的授權條款。 您可以立即[下載一份 EULA](https://go.microsoft.com/fwlink/?LinkID=733084)。 |
| Readme.txt |指向版本資訊和基本安裝指示的連結。 這是您在此文件中找到的一部分指示。 |
| ThirdPartyNotice.rtf |套件中協力廠商軟體的注意事項。 |
| Tools\Microsoft.Azure.ServiceFabric.WindowsServer.SupportPackage.zip |依需求執行的 StandaloneLogCollector.exe，可收集追蹤記錄並上傳至 Microsoft 以供支援用途。 |
| Tools\ServiceFabricUpdateService.zip |一種用於為沒有網際網路存取權的叢集，啟用自動程式碼升級的工具。 在 [這裡](service-fabric-cluster-upgrade-windows-server.md)|

**範本** 

| [檔案名稱]**** | **簡短描述** |
| --- | --- |
| ClusterConfig.Unsecure.DevCluster.json |叢集組態範例檔案，其中包含適用於不安全、三個節點、單一電腦 (或虛擬機器) 部署叢集的設定，包括叢集中每個節點的相關資訊。 |
| ClusterConfig.Unsecure.MultiMachine.json |叢集組態範例檔案，其中包含適用於不安全、多個電腦 (或虛擬機器) 叢集的設定，包括叢集中每個電腦的相關資訊。 |
| ClusterConfig.Windows.DevCluster.json |叢集組態範例檔案，其中包含適用於安全、三個節點、單一電腦 (或虛擬機器) 部署叢集的所有設定，包括叢集中每個節點的相關資訊。 使用 [Windows 身分識別](https://msdn.microsoft.com/library/ff649396.aspx)保護叢集的安全。 |
| ClusterConfig.Windows.MultiMachine.json |叢集組態範例檔案，其中包含適用於使用 Windows 安全性的安全、多個電腦 (或虛擬機器) 叢集的所有設定，包括安全叢集中每個電腦的相關資訊。 使用 [Windows 身分識別](https://msdn.microsoft.com/library/ff649396.aspx)保護叢集的安全。 |
| ClusterConfig.x509.DevCluster.json |叢集組態範例檔案，其中包含適用於安全、三個節點、單一電腦 (或虛擬機器) 部署叢集的所有設定，包括叢集中每個節點的相關資訊。 使用 x509 憑證保護叢集的安全。 |
| ClusterConfig.x509.MultiMachine.json |叢集組態範例檔案，其中包含適用於安全、多個電腦 (或虛擬機器) 叢集的所有設定，包括安全叢集中每個節點的相關資訊。 使用 x509 憑證保護叢集的安全。 |
| ClusterConfig.gMSA.Windows.MultiMachine.json |叢集組態範例檔案，其中包含適用於安全、多個電腦 (或虛擬機器) 叢集的所有設定，包括安全叢集中每個節點的相關資訊。 使用[群組受控服務帳戶](https://technet.microsoft.com/library/jj128431(v=ws.11).aspx)保護叢集的安全。 |

## <a name="cluster-configuration-samples"></a>叢集組態範例
最新版本的群集配置範本可以在 GitHub 頁面：[獨立群集配置示例](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)中找到。

## <a name="independent-runtime-package"></a>獨立的執行階段套件
叢集部署期間會自動從[下載連結 - Service Fabric 執行階段 - Windows Server](https://go.microsoft.com/fwlink/?linkid=839354)下載最新的執行階段套件。

## <a name="related"></a>相關參考
* [建立獨立 Azure Service Fabric 叢集](service-fabric-cluster-creation-for-windows-server.md)
* [Service Fabric 叢集安全性案例](service-fabric-windows-cluster-windows-security.md)

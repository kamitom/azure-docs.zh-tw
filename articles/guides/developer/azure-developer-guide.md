---
title: Azure 上的開發人員開始使用指南 | Microsoft Docs
description: 本文為希望開始使用 Microsoft Azure 平臺以滿足其開發需求的開發人員提供了重要資訊。
services: ''
cloud: ''
documentationcenter: ''
author: ggailey777
manager: erikre
ms.assetid: ''
ms.service: azure
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/18/2019
ms.author: glenga
ms.openlocfilehash: d74fd2e3f6b5cc090c6313aba67a1e139c713b85
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79245001"
---
# <a name="get-started-guide-for-azure-developers"></a>Azure 開發人員開始使用指南

## <a name="what-is-azure"></a>何謂 Azure？

Azure 是一個完整的雲平臺，可以託管現有應用程式並簡化新的應用程式開發。 Azure 甚至可以增強本地應用程式。 Azure 集成了開發、測試、部署和管理應用程式所需的雲服務，同時利用雲計算的效率。

在 Azure 中裝載應用程式，即可從小規模著手，並隨著客戶需求的成長，輕鬆地調整應用程式。 Azure 也會提供高可用性應用程式所需的可靠性，甚至包括不同地區之間的容錯移轉。 [Azure 入口網站](https://portal.azure.com)可讓您輕鬆地管理您的所有 Azure 服務。 您也可以使用服務特定 API 和範本，以程式設計方式管理服務。

本指南是適用於應用程式開發人員的 Azure 平台簡介。 它會提供下列作業所需的指引和指示：開始在 Azure 中建置新應用程式，或將現有應用程式移轉至 Azure。

## <a name="where-do-i-start"></a>我該從哪裡開始？

使用 Azure 提供的所有服務，找出支援解決方案體系結構所需的服務可能是一項令人生畏的任務。 本節著重於開發人員常用的 Azure 服務。 如需所有 Azure 服務的清單，請參閱 [Azure 文件](../../index.yml)。

首先，您必須決定如何在 Azure 中裝載應用程式。 您需要以虛擬機器 (VM) 形式管理整個基礎結構嗎？ 您可以使用 Azure 所提供的平台管理功能嗎？ 您只需要使用無伺服器架構來裝載程式碼執行嗎？

您的應用程式需要 Azure 提供數個選項的雲端儲存體。 您可以利用 Azure 的企業驗證。 另外還有工具可進行雲端式開發和監視，而且大部分的裝載服務提供 DevOps 整合。

現在，請查看我們建議您針對應用程式所調查的某些特定服務。

### <a name="application-hosting"></a>應用程式裝載

Azure 提供數個雲端式運算供應項目來執行您的應用程式，讓您不需要擔心基礎結構詳細資料。 隨著您應用程式使用的成長，您可以輕鬆地向上延展或向外延展資源。

Azure 提供可支援應用程式開發和裝載需求的服務。 Azure 提供基礎結構即服務 (IaaS)，讓您完全控制應用程式裝載。 Azure 的平臺即服務 （PaaS） 產品提供為應用供電所需的完全託管服務。 Azure 中甚至存在真正的無伺服器託管，您需要在其中只需編寫代碼。

![Azure 應用程式裝載選項](./media/azure-developer-guide/azure-developer-hosting-options.png)


#### <a name="azure-app-service"></a>Azure App Service

當您想要發佈 Web 專案的最快速路徑時，請考慮使用 Azure App Service。 App Service 可讓您更輕鬆地擴充 Web 應用程式來支援行動用戶端，以及發佈方便使用的 REST API。 此平台提供驗證的方式是使用社交提供者、流量自動調整，在生產環境中測試，以及持續的容器部署。

您可以建立 Web 應用程式、行動應用程式後端和 API 應用程式。

因為所有三種應用程式類型都共用 App Service 執行階段，所以您可以架設網站、支援行動用戶端，並在 Azure 中公開 API，而全部都是來自相同的專案或解決方案。 若要深入了解 App Service，請參閱[什麼是 Azure Web Apps](../../app-service/overview.md)。

請注意，已使用 DevOps 設計 App Service。 它支援用於發佈和持續集成部署的各種工具。 這些工具組括 GitHub 網路掛、Jenkins、Azure DevOps、團隊城等。

您可以使用[線上移轉工具](https://appmigration.microsoft.com/)，將現有應用程式移轉至 App Service。

> **何時使用**：將現有 Web 應用程式遷移到 Azure 時以及需要 Web 應用的完全託管託管平臺時使用應用服務。 當您需要支援行動用戶端或是使用應用程式公開 REST API 時，也可以使用 App Service。
>
> **開始使用**：App Service 可讓您輕鬆地建立和部署第一個 [Web 應用程式](../../app-service/app-service-web-get-started-dotnet.md)、[行動應用程式](../../app-service-mobile/app-service-mobile-ios-get-started.md)或 [API 應用程式](../../app-service/app-service-web-tutorial-rest-api.md)。
>
> **立即試用**：App Service 可讓您佈建短期應用程式來嘗試平台，而不需要註冊 Azure 帳戶。 請嘗試平台，並[建立 Azure App Service 應用程式](https://tryappservice.azure.com/)。

#### <a name="azure-virtual-machines"></a>Azure 虛擬機器

身為基礎結構即服務 (IaaS) 提供者，Azure 可讓您將應用程式部署或移轉至 Windows 或 Linux VM。 Azure Virtual Machines 與 Azure Virtual Network 搭配使用，可支援將 Windows 或 Linux VM 部署至 Azure。 使用 VM，您可以完整控制電腦的設定。 使用 VM 時，您負責所有伺服器軟體安裝、設定、維護和作業系統修補程式。

基於您對 VM 的控制等級，您可以在 Azure 上執行不符合 PaaS 模型的各種不同伺服器工作負載。 這些工作負載包含資料庫伺服器、Windows Server Active Directory 和 Microsoft SharePoint。 如需詳細資訊，請參閱 [Linux](/azure/virtual-machines/linux/) 或 [Windows](/azure/virtual-machines/windows/) 的 Virtual Machines 文件。

> **使用時機**：當您想要完整控制應用程式基礎結構時，或將內部部署應用程式工作負載移轉至 Azure 而不需要進行變更時，請使用 Virtual Machines。
>
> **開始使用**：從 Azure 入口網站建立 [Linux VM](../../virtual-machines/virtual-machines-linux-quick-create-portal.md) 或 [Windows VM](../../virtual-machines/virtual-machines-windows-hero-tutorial.md)。

#### <a name="azure-functions-serverless"></a>Azure Functions (無伺服器)

與其擔心構建和管理整個應用程式或基礎結構來運行代碼，不如說，如果可以編寫代碼並讓它運行以回應事件或按計劃運行，該怎麼辦？  [Azure Functions](../../azure-functions/functions-overview.md) 是「無伺服器」樣式供應項目，可讓您只撰寫所需的程式碼。 使用函數，您可以使用 HTTP 要求、Webhook、雲服務事件或計畫觸發代碼執行。 您也可以使用選擇的開發語言編寫程式碼，例如 C\#、F\#、Node.js、Python 或 PHP。 使用耗用型計費時，您只需要支付程式碼執行時間的費用，而且 Azure 會視需要調整。

> **使用時機**：當您的程式碼是透過其他 Azure 服務、Web 事件或依排程觸發時，請使用 Azure Functions。 當您不需要整個託管專案的額外負荷時，或只想要支付程式碼執行時間的費用時，也可以使用 Functions。 若要深入了解，請參閱 [Azure Functions 概觀](../../azure-functions/functions-overview.md)。
>
> **開始使用**：遵循 Functions 快速入門教學課程，從入口網站[建立第一個函式](../../azure-functions/functions-create-first-azure-function.md)。
>
> **立即試用**：Azure Functions 可讓您執行程式碼，而不需要註冊 Azure 帳戶。 請立即試用，並[建立您的第一個 Azure 函式](https://tryappservice.azure.com/)。

#### <a name="azure-service-fabric"></a>Azure Service Fabric

Azure 服務結構是一個分散式系統平臺。 此平臺可輕鬆構建、打包、部署和管理可擴展且可靠的微服務。 它還提供全面的應用程式管理功能，例如：

* 佈建
* 正在部署
* 監視
* 升級/修補
* 刪除中

在共用電腦集區上執行的應用程式可以從小規模著手，然後視需要調整為成千上萬部電腦。

Service Fabric 支援具有 Open Web Interface for .NET (OWIN) 和 ASP.NET Core 的 WebAPI。 它提供在 Linux 上建置服務的 .NET Core 和 Java SDK。 若要深入了解 Service Fabric，請參閱 [Service Fabric 文件](https://docs.microsoft.com/azure/service-fabric/)。

> **使用時機**：當您建立應用程式或重新撰寫現有的應用程式以使用微服務架構時，Service Fabric 是不錯的選擇。 當您需要更充分地控制或直接存取基礎結構時，請使用 Service Fabric。
>
> **開始使用** [：建立第一個 Azure Service Fabric 應用程式](../../service-fabric/service-fabric-create-your-first-application-in-visual-studio.md)。

### <a name="enhance-your-applications-with-azure-services"></a>使用 Azure 服務增強應用程式

除了應用程式託管，Azure 還提供可以增強功能的服務產品。 Azure 還可以改進應用程式在雲和本地中的開發和維護。

#### <a name="hosted-storage-and-data-access"></a>託管儲存體和資料存取

大多數應用程式必須存儲資料，因此，但是您決定在 Azure 中託管應用程式，請考慮以下一個或多個存儲和資料服務。

- **Azure Cosmos DB：** 一個全域分佈的多模型資料庫服務。 此資料庫使您能夠使用全面的 SLA 對任意數量的地理區域進行彈性擴展輸送量和存儲。

  > **使用時機：** 當您的應用程式需要文件、資料表或圖表資料庫時，請併入具有多個定義完善、一致模型的 MongoDB 資料庫。
  >
  > **入門**：[建置 Azure Cosmos DB Web 應用程式](../../cosmos-db/create-sql-api-dotnet.md)。 如果您是 MongoDB 開發人員，請參閱[使用 Azure Cosmos DB 建置 MongoDB Web 應用程式](../../cosmos-db/create-mongodb-dotnet.md)。

- **Azure 儲存體**：提供 Blob、佇列、檔案和其他類型之非關聯式資料的持久性高可用性儲存體。 儲存體提供 VM 的儲存體基礎。

  > **使用時機：** 當您的應用程式儲存非關聯式資料 (例如，鍵值組 (資料表)、Blob、檔案共用或訊息 (佇列)) 時。
  >
  > **開始使用**：選擇下列其中一種類型的儲存體：[Blob](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)、[資料表](../../cosmos-db/table-storage-how-to-use-dotnet.md)、[佇列](../../storage/queues/storage-dotnet-how-to-use-queues.md)或[檔案](../../storage/files/storage-dotnet-how-to-use-files.md)。

- **Azure SQL Database**：Azure 版本的 Microsoft SQL Server 引擎，可將關聯式表格資料儲存在雲端中。 SQL Database 提供可預測的效能、無停機時間的延展性、商務持續性和資料保護功能。

  > **使用時機：** 當您的應用程式需要具有參考完整性、交易支援和 TSQL 查詢支援的資料儲存體時。
  >
  > **開始使用**[：使用 Azure 入口網站在幾分鐘內建立 SQL Database](../../sql-database/sql-database-get-started.md)。


您可以使用 [Azure Data Factory](../../data-factory/introduction.md)，將現有內部部署資料移至 Azure。 如果尚未準備好將資料移動到雲中，Azure 應用服務中的[混合連接](../../app-service/app-service-hybrid-connections.md)允許您將應用服務託管應用連接到本地資源。 您也可以從內部部署應用程式連接至 Azure 資料和儲存體服務。

#### <a name="docker-support"></a>Docker 支援

Docker 容器是一種 OS 虛擬化，可讓您透過更有效率且可預測的方式部署應用程式。 容器化應用程式在生產環境中的運作方式與在開發和測試系統上相同。 您可以使用標準 Docker 工具來管理容器。 您可以在 Azure 上使用現有技能和熱門開放原始碼工具來部署和管理容器應用程式。

Azure 提供數種方式，以在應用程式中使用容器。

- **Azure Docker VM 延伸模組**：可讓您使用 Docker 工具，將 VM 設定為 Docker 主機。

  > **使用時機**：當您想要在 VM 上產生應用程式的一致容器部署時，或當您想要使用 [Docker Compose](https://docs.docker.com/compose/overview/) 時。
  >
  > **開始使用**：[使用 Docker VM 延伸模組在 Azure 中建立 Docker 環境](../../virtual-machines/virtual-machines-linux-dockerextension.md)。

- **Azure Kubernetes 服務**：允許您創建、配置和管理預配置為運行容器化應用程式的虛擬機器群集。 要瞭解有關 Azure 庫伯奈斯服務的更多詳細資訊，請參閱[Azure 庫伯奈斯服務簡介](../../aks/intro-kubernetes.md)。

  > **使用時機**：當您需要建置提供其他排程和管理工具的生產就緒、可調整環境時，或當您部署 Docker Swarm 叢集時。
  >
  > **開始**：[部署庫伯內斯服務群集](../../aks/tutorial-kubernetes-deploy-cluster.md)。

- **Docker Machine**：可讓您使用 docker-machine 命令安裝和管理虛擬主機上的 Docker 引擎。

  >**使用時機**：當您需要建立單一 Docker 主機來快速設定應用程式的原型時。

- **App Service 的自訂 Docker 映像**：在 Linux 上部署 Web 應用程式時，可讓您使用容器登錄或客戶容器中的 Docker 容器。

  > **使用時機**：在 Linux 上將 Web 應用程式部署至 Docker 映像時。
  >
  > **開始使用**：[針對 Linux 上的 App Service 使用自訂 Docker 映像](../../app-service/containers/quickstart-docker-go.md)。

### <a name="authentication"></a>驗證

重要的不只是知道誰正在使用您的應用程式，還可以防止未經授權的資源存取。 Azure 提供數種方式，以驗證應用程式用戶端。

- **Azure Active Directory (Azure AD)**：Microsoft 多租用戶雲端式身分識別和存取管理服務。 您可以透過與 Azure AD 整合，將單一登入 (SSO) 新增至應用程式。 您可以直接使用 Azure AD Graph API 或使用 Microsoft Graph API，來存取目錄內容。 您可以使用原生 HTTP/REST 端點和多平台 Azure AD 驗證程式庫，以與 OAuth2.0 授權架構和 Open ID Connect 的 Azure AD 支援整合。

  > **使用時機**：當您想要提供 SSO 體驗時，請使用 Graph 資料，或驗證網域使用者。
  >
  > **開始使用**：若要深入了解，請參閱 [Azure Active Directory 開發人員指南](../../active-directory/develop/v2-overview.md)。

- **App Service 驗證**：當您選擇 App Service 裝載應用程式時，也會取得 Azure AD 的內建驗證支援，以及社交身分識別提供者，包括 Facebook、Google、Microsoft 和 Twitter。

  > **使用時機**：當您想要使用 Azure AD 和 (或) 社交身分識別提供者在 App Service 應用程式中啟用驗證時。
  >
  > **開始使用**：若要深入了解 App Service 中的驗證，請參閱 [Azure App Service 中的驗證和授權](../../app-service/overview-authentication-authorization.md)。

若要深入了解 Azure 中的安全性最佳做法，請參閱 [Azure 安全性最佳做法與模式](../../security/fundamentals/best-practices-and-patterns.md)。

### <a name="monitoring"></a>監視

在 Azure 中啟動和運行應用程式後，需要監視性能、監視問題以及查看客戶如何使用應用。 Azure 提供數個監視選項。

-   **應用程式見解**：一種由 Azure 託管的可擴展分析服務，與 Visual Studio 集成以監視即時 Web 應用程式。 它為您提供了持續提高應用性能和可用性所需的資料。 無論您是否在 Azure 上託管應用程式，都會發生此改進。

    >**開始使用**：遵循 [Application Insights 教學課程](../../azure-monitor/app/app-insights-overview.md)。

-   **Azure 監視器**：一種服務，可説明您視覺化、查詢、路由、存檔，並處理使用 Azure 基礎結構和資源生成的指標和日誌。 監視器是監視 Azure 資源的單一來源，並提供您在 Azure 門戶中看到的資料檢視。

    >**開始使用**：[開始使用 Azure 監視器](../../monitoring-and-diagnostics/monitoring-get-started.md)。

### <a name="devops-integration"></a>DevOps 整合

不論使用持續整合佈建 VM 還是發佈 Web 應用程式，Azure 都會與大部分的熱門 DevOps 工具整合。 您可以使用已有的工具，並借助支援以下工具來最大化現有體驗：

* Jenkins
* GitHub
* Puppet
* Chef
* 團隊城
* Ansible
* Azure DevOps

> **開始使用**：若要查看 App Service 應用程式的 DevOps 選項，請參閱[持續部署至 Azure App Service](../../app-service/deploy-continuous-deployment.md)。
>
> **立即試用：** [試用數個 DevOps 整合](https://azure.microsoft.com/try/devops/)。


## <a name="azure-regions"></a>Azure 區域

Azure 是在全球各地許多地區正式推出的全域雲端平台。 在 Azure 中預配服務、應用程式或 VM 時，系統會要求您選擇區域。 此區域表示應用程式運行或資料存儲的特定資料中心。 這些區域對應到在 [Azure 區域](https://azure.microsoft.com/regions/)頁面上發佈的特定位置。

### <a name="choose-the-best-region-for-your-application-and-data"></a>選擇您應用程式和資料的最佳區域

使用 Azure 的其中一個優點是您可以將應用程式部署至全球各地的各種資料中心。 您選擇的區域可能會影響應用程式的效能。 例如，最好選擇較接近大多數客戶的區域，以減少網路要求延遲。 您可能還希望選擇您所在的地區以滿足在某些國家/地區分發應用的法律要求。 最好一律將應用程式資料儲存在相同的資料中心內，或儲存在最接近裝載應用程式之資料中心的資料中心內。

### <a name="multi-region-apps"></a>多區域應用程式

雖然不可能，但是整個資料中心不可能因天然災害或網際網路失敗等事件而離線。 最好將重要商務應用程式裝載在多個資料中心，以提供最大的可用性。 使用多個區域也可以減少全域使用者的延遲，並在更新應用程式時提供彈性的其他機會。

某些服務 (例如 Virtual Machine 和 App Services) 使用 [Azure 流量管理員](../../traffic-manager/traffic-manager-overview.md)來啟用具有區域間之容錯移轉的多區域支援，以支援高可用性企業應用程式。 如需範例，請參閱 [Azure 參考架構︰在多個區域中執行 Web 應用程式](https://docs.microsoft.com/azure/architecture/reference-architectures/app-service-web-app/multi-region)。

>**使用時機**：當您有受益於容錯移轉和複寫的企業和高可用性應用程式時。

## <a name="how-do-i-manage-my-applications-and-projects"></a>如何管理我的應用程式和專案？

Azure 提供一組豐富的體驗，讓您建立和管理 Azure 資源、應用程式和專案，方法是透過程式設計以及在 [Azure 入口網站](https://portal.azure.com/)中進行。

### <a name="command-line-interfaces-and-powershell"></a>命令列介面和 PowerShell

Azure 提供了兩種從命令列管理應用程式和服務的方法。 您可以使用 Bash、終端、命令提示符或您選擇的命令列工具等工具。 通常，可以從命令列執行與 Azure 門戶相同的任務，例如創建和配置虛擬機器、虛擬網路、Web 應用和其他服務。

-   [Azure 命令列介面 (CLI)](../../xplat-cli-install.md)：可讓您連接到 Azure 訂用帳戶，並從命令列對 Azure 資源設計各種工作的程式。

-   [Azure PowerShell](../../powershell-install-configure.md)：提供一組含 Cmdlet 的模組，可讓您使用 Windows PowerShell 來管理 Azure 資源。

### <a name="azure-portal"></a>Azure 入口網站

[Azure 門戶](https://portal.azure.com)是基於 Web 的應用程式。 可以使用 Azure 門戶創建、管理和刪除 Azure 資源和服務。 其中包括：

* 可配置儀表板
* Azure 資源管理工具
* 訪問訂閱設置和計費資訊。 如需詳細資訊，請參閱 [Azure 入口網站概觀](../../azure-portal-overview.md)。

### <a name="rest-apis"></a>REST API

Azure 是以支援 Azure 入口網站 UI 的一組 REST API 所建置。 也支援其中的大部分 REST API，讓您以程式設計方式從任何啟用網際網路的裝置佈建和管理 Azure 資源和應用程式。 如需一組完整 REST API 文件，請參閱 [Azure REST SDK 參考](https://docs.microsoft.com/rest/api/)。

### <a name="apis"></a>API

除了 REST API，許多 Azure 服務還允許您使用特定于平臺的 Azure SDK（包括用於以下開發平臺的 SDK）以程式設計方式管理來自應用程式的資源：

-   [.NET](https://go.microsoft.com/fwlink/?linkid=834925)
-   [Node.js](https://docs.microsoft.com/azure/javascript/)
-   [JAVA](https://docs.microsoft.com/java/azure)
-   [Php](https://github.com/Azure/azure-sdk-for-php/blob/master/README.md)
-   [Python](/azure/python/)
-   [紅寶石](https://github.com/Azure/azure-sdk-for-ruby/blob/master/README.md)
-   [移至](https://docs.microsoft.com/azure/go)

[Mobile Apps](../../app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library.md) 和 [Azure 媒體服務](../../media-services/previous/media-services-dotnet-how-to-use.md) 這類服務提供用戶端 SDK，讓您可以從 Web 和行動用戶端應用程式存取服務。

### <a name="azure-resource-manager"></a>Azure Resource Manager

在 Azure 上運行應用可能涉及使用多個 Azure 服務。 這些服務遵循相同的生命週期，可以被視為一個邏輯單元。 例如，Web 應用程式可能使用 Web Apps、SQL Database、「儲存體」、「Azure Redis 快取」及「Azure 內容傳遞網路」服務。 [Azure Resource Manager](../../azure-resource-manager/management/overview.md) 可讓您將應用程式中的資源當成群組使用。 您可以透過單一的協調作業來部署、更新或刪除所有資源。

除了對相關資源進行邏輯分組和管理外，Azure 資源管理器還包括部署功能，允許您自訂相關資源的部署和配置。 例如，可以使用資源管理器部署和配置應用程式。 此應用程式可以由多個虛擬機器、負載等化器和 Azure SQL 資料庫作為單個單元組成。

透過使用本身為 JSON 格式化文件的 Azure Resource Manager 範本，即可開發這些部署。 範本可讓您定義部署，以及使用宣告式範本而非指令碼來管理應用程式。 您的範本可以用於測試、預備和生產這類不同環境。 例如，可以使用範本向 GitHub 存儲庫添加按鈕，該存儲庫中的代碼只需按一下一下即可將存儲庫中的代碼部署到一組 Azure 服務。

> **使用時機**：當您想要可使用 REST API、Azure CLI 和 Azure PowerShell 以程式設計方式管理之應用程式的範本部署，請使用 Resource Manager 範本。
>
> **開始使用**：若要開始使用範本，請參閱[製作 Azure Resource Manager 範本](../../resource-group-authoring-templates.md)。

## <a name="understanding-accounts-subscriptions-and-billing"></a>了解帳戶、訂用帳戶和計費

身為開發人員，我們要深入了解程式碼，並嘗試盡快開始讓我們的應用程式執行。 當然，我們想要鼓勵您盡可能輕鬆地開始使用 Azure。 為了協助您更輕鬆地進行，Azure 提供[免費試用](https://azure.microsoft.com/free/)。 某些服務甚至具有「免費試用」功能，例如 [Azure App Service](https://tryappservice.azure.com/)，而且您甚至不需要建立帳戶。 雖然深入瞭解對應用程式進行編碼並將應用程式部署到 Azure 很有趣，但瞭解 Azure 的工作原理也很重要。 具體來說，您應該從使用者帳戶、訂閱和計費的角度瞭解其工作原理。

### <a name="what-is-an-azure-account"></a>什麼是 Azure 帳戶？

要創建或使用 Azure 訂閱，必須具有 Azure 帳戶。 Azure 帳戶只是 Azure AD 或 Azure AD 信任的目錄中（如工作或學校組織）中的標識。 如果您不屬於這類組織，則一律會使用 Azure AD 所信任的 Microsoft 帳戶來建立訂用帳戶。 若要深入了解整合內部部署 Windows Server Active Directory 與 Azure AD，請參閱[整合您的內部部署身分識別與 Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md)。

每個 Azure 訂用帳戶都會與 Azure AD 執行個體有信任關係。 這表示它信任該目錄來驗證使用者、服務和裝置。 多個訂用帳戶可以信任相同的目錄，但是一個訂用帳戶只能信任一個目錄。 若要深入了解，請參閱 [Azure 訂用帳戶與 Azure Active Directory 建立關聯的方式](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)。

除了定義單個 Azure 帳戶標識（也稱為*使用者*）外，還可以在 Azure AD 中定義*組*。 建立使用者群組是使用角色型存取控制 (RBAC) 管理訂用帳戶中資源存取權的好方法。 若要了解如何建立群組，請參閱[在 Azure Active Directory Preview 中建立群組](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。 您也可以[使用 PowerShell](../../active-directory/users-groups-roles/groups-settings-v2-cmdlets.md) 建立和管理群組。

### <a name="manage-your-subscriptions"></a>管理訂用帳戶

訂用帳戶是連結至 Azure 帳戶之 Azure 服務的邏輯分組。 單一 Azure 帳戶可以包含多個訂用帳戶。 Azure 服務是根據訂用帳戶計費。 如需依類型的可用訂用帳戶供應項目清單，請參閱 [Microsoft Azure 供應項目詳細資料](https://azure.microsoft.com/support/legal/offer-details/)。 Azure 訂閱具有對訂閱具有完全控制的帳戶管理員。 他們還有一個服務管理員，他控制著訂閱中的所有服務。 如需有關傳統訂用帳戶管理員的資訊，請參閱[新增或變更 Azure 訂用帳戶管理員](../../cost-management-billing/manage/add-change-subscription-administrator.md)。 可以使用[基於角色的存取控制 （RBAC）](../../role-based-access-control/overview.md)授予單個帳戶對 Azure 資源的詳細控制。

#### <a name="resource-groups"></a>資源群組

當您佈建新的 Azure 服務時，即可在指定的訂用帳戶中這麼做。 個別 Azure 服務 (也稱為資源) 是在資源群組的內容中建立。 資源群組可讓您更輕鬆地部署和管理應用程式資源。 資源群組應該包含您想要當成一個單位使用之應用程式的所有資源。 您可以在資源群組之間移動資源，甚至移至不同的訂用帳戶。 若要了解如何移動資源，請參閱[將資源移動到新的資源群組或訂用帳戶](../../azure-resource-manager/management/move-resource-group-and-subscription.md)。

Azure 資源總管是很棒的工具，可用來以視覺化方式檢視您已經在訂用帳戶中建立的資源。 若要深入了解，請參閱[使用 Azure 資源總管來檢視及修改資源](../../resource-manager-resource-explorer.md)。

#### <a name="grant-access-to-resources"></a>授與對資源的存取

當您允許訪問 Azure 資源時，最佳做法始終是為使用者提供執行給定任務所需的最低許可權。

- **角色型存取控制 (RBAC)**：在 Azure 中，您可以授與指定範圍的使用者帳戶 (主體) 存取權：訂用帳戶、資源群組或個別資源。 RBAC 允許您將資源部署到資源組，並向特定使用者或組授予許可權。 它還允許您僅將存取權限限制為屬於目標資源組的資源。 您也可以授與對單一資源的存取，例如虛擬機器或虛擬網路。 若要授與存取權，請將角色指派給使用者、群組或服務主體。 有許多預先定義的角色，而且您也可以定義自己的自訂角色。 要瞭解更多資訊，請參閱[什麼是基於角色的存取控制 （RBAC）？？](../../role-based-access-control/overview.md)

  > **使用時機**：當您需要使用者和群組的更細緻存取管理時，或當您需要將使用者設定為訂用帳戶的擁有者時。
  >
  > **開始使用**：若要深入了解，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../../role-based-access-control/role-assignments-portal.md)。

- **服務主體物件**：除了提供對使用者主體和組的訪問外，還可以授予對服務主體的相同存取權限。

  > **使用時機**：當您以程式設計方式管理 Azure 資源或授與應用程式存取權時。 如需詳細資訊，請參閱[建立 Active Directory 應用程式和服務主體](../../active-directory/develop/howto-create-service-principal-portal.md)。

#### <a name="tags"></a>Tags

Azure Resource Manager 可讓您將自訂標記指派給個別資源。 當您需要組織資源以進行計費或監視時，標記 (即鍵值組) 可能特別有用。 標記提供一種方法來追蹤多個資源群組的資源。 您可以通過以下方式分配標記：

* 在門戶中
* 在 Azure 資源管理器範本中
* 使用 REST API
* 使用 Azure CLI
* 使用 PowerShell

您可以將多個標記指派給每個資源。 若要深入了解，請參閱[使用標記來組織 Azure 資源](../../resource-group-using-tags.md)。

### <a name="billing"></a>計費

從內部部署運算移至雲端託管服務時，追蹤和預估服務使用和相關成本十分重要。 估計每月運行的新資源成本非常重要。 您還可以根據當前支出預測單月計費的外觀。

#### <a name="get-resource-usage-data"></a>取得資源使用量資料

Azure 提供一組計費 REST API，可存取 Azure 訂用帳戶的資源使用和中繼資料資訊。 這些計費 API 可讓您更妥善地預測和管理 Azure 成本。 您可以以小時增量跟蹤和分析支出，並創建支出警報。 您還可以根據當前使用趨勢預測未來的計費。

>**開始使用**：若要深入了解如何使用計費 API，請參閱 [Azure 計費使用和 RateCard API 概觀](../../cost-management-billing/manage/usage-rate-card-overview.md)。

#### <a name="predict-future-costs"></a>預測未來成本

儘管提前估算成本具有挑戰性，但 Azure 擁有可説明您的工具。 它有一個[定價計算機](https://azure.microsoft.com/pricing/calculator/)，以説明估計部署的資源的成本。 您還可以使用門戶中的計費資源和計費 REST API 根據當前消耗量估計未來成本。

>**開始使用**：請參閱 [Azure 計費使用和 RateCard API 概觀](../../cost-management-billing/manage/usage-rate-card-overview.md)。

---
title: 在本地部署遠端監視解決方案 - 視覺化工作室代碼 - Azure |微軟文檔
description: 此操作指南會示範如何使用 Visual Studio Code，將遠端監視解決方案加速器部署到本機電腦，以進行測試和開發。
author: avneet723
manager: hegate
ms.author: avneets
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/17/2019
ms.topic: conceptual
ms.openlocfilehash: 8f1d20e9a6a78d99a23fe4b98aeb4f3eb8359da7
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "73890948"
---
# <a name="deploy-the-remote-monitoring-solution-accelerator-locally---visual-studio-code"></a>本機部署遠端監視解決方案加速器 - Visual Studio Code

[!INCLUDE [iot-accelerators-selector-local](../../includes/iot-accelerators-selector-local.md)]

本文會示範如何將遠端監視解決方案加速器部署到本機電腦，以進行測試和開發。 您可了解如何在 Visual Studio Code 中執行微服務。 本地微服務部署使用以下雲服務：IoT 中心、Cosmos DB、Azure 流分析和 Azure 時間序列見解。

## <a name="prerequisites"></a>Prerequisites

若要部署遠端監視解決方案加速器所使用的 Azure 服務，您需要一個有效的 Azure 訂用帳戶。

如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 有關詳細資訊，請參閱[Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

### <a name="machine-setup"></a>電腦設定

若要完成本機部署，您必須在本機開發機器上安裝下列工具：

* [Git](https://git-scm.com/)
* [.NET Core](https://dotnet.microsoft.com/download)
* [Docker](https://www.docker.com)
* [Nginx](https://nginx.org/en/download.html)
* [視覺工作室代碼](https://code.visualstudio.com/)
* [VS Code 的 C# 擴充功能](https://code.visualstudio.com/docs/languages/csharp)
* [Node.js v8](https://nodejs.org/) - 此軟體是指令碼用來建立 Azure 資源之 PCS CLI 的先決條件。 不要使用 Node.js v10

> [!NOTE]
> Visual Studio Code 適用於 Windows、Mac 和 Ubuntu。

[!INCLUDE [iot-accelerators-local-setup](../../includes/iot-accelerators-local-setup.md)]

## <a name="run-the-microservices"></a>執行微服務

在本節中，您會執行遠端監視微服務。 原生執行 Web UI、Docker 中的裝置模擬服務，以及 Visual Studio Code 中的微服務。

### <a name="build-the-code"></a>建置程式碼

在命令提示字元中瀏覽至 azure-iot-pcs-remote-monitoring-dotnet\services，並執行下列命令來建置程式碼。

```cmd
dotnet restore
dotnet build -c Release
```

### <a name="deploy-all-other-microservices-on-local-machine"></a>在本機電腦上部署所有其他微服務

以下步驟演示如何在 Visual Studio 代碼中運行遠端監視微服務：

1. 啟動 Visual Studio Code。
1. 在 VS 代碼中，打開**azure-iot-pcs 遠端監視點網**資料夾。
1. 在**azure-iot-pcs 遠端監視點網**資料夾中創建一個名為 **.vscode**的新資料夾。
1. 將檔**啟動.json**和**tasks.json**從服務\腳本\local_啟動_idesettings_vscode 複製到您剛剛創建的 **.vscode**資料夾。
1. 在 VS 代碼中打開**調試面板**並運行 **"運行所有微服務**"配置。 此組態會在 Docker 中執行裝置模擬微服務，並且在偵錯工具中執行其他微服務。

運行**運行中的所有微索器**的輸出如下所示：

[![部署本地-微服務](./media/deploy-locally-vscode/auth-debug-results-inline.png)](./media/deploy-locally-vscode/auth-debug-results-expanded.png#lightbox)

### <a name="run-the-web-ui"></a>執行 Web UI

在此步驟中，您會啟動 Web UI。 瀏覽至您本機複本中的 **azure-iot-pcs-remote-monitoring-dotnet\webui** 資料夾，然後執行下列命令：

```cmd
npm install
npm start
```

啟動完成後，瀏覽器將顯示頁面**\/HTTP： /本地主機：3000/儀表板**。 此頁面上的錯誤一如預期。 若要檢視沒有錯誤的應用程式，請完成下列步驟。

### <a name="configure-and-run-nginx"></a>設定和執行 NGINX

設定反向 Proxy 伺服器，以連結 Web 應用程式與在本機電腦上執行的微服務：

* 將 **webui\scripts\localhost** 資料夾中的 **nginx.conf** 檔案複製到 **nginx\conf** 安裝目錄。
* 執行 **nginx**。

如需執行 **nginx** 的詳細資訊，請參閱[適用於 Windows 的 nginx](https://nginx.org/en/docs/windows.html) \(英文\)。

### <a name="connect-to-the-dashboard"></a>連線至儀表板

要訪問遠端監視解決方案儀表板，請導航到瀏覽器中的\/HTTP： /localhost：9000。

## <a name="clean-up"></a>清除

為避免不必要的費用，當您完成測試後，請從您的 Azure 訂用帳戶中移除雲端服務。 若要移除服務，請瀏覽到 [Azure 入口網站](https://ms.portal.azure.com)，然後刪除 **start.cmd** 指令碼建立的資源群組。

您也可以將您從 GitHub 複製原始程式碼時，所建立的遠端監視存放庫本機副本刪除。

## <a name="next-steps"></a>後續步驟

既然您已部署遠端監視解決方案，下一步便是[探索解決方案儀表板的功能](quickstart-remote-monitoring-deploy.md)。

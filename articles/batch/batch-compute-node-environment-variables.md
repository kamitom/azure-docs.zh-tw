---
title: 任務運行時環境變數 - Azure 批次處理 |微軟文檔
description: 任務運行時環境變數指導和 Azure 批次處理分析的參考。
services: batch
author: LauraBrenner
manager: evansma
ms.assetid: ''
ms.service: batch
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 09/12/2019
ms.author: labrenne
ms.openlocfilehash: ebaa06acf309a0f941b8b4efd76fa4e9e7460810
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80053939"
---
# <a name="azure-batch-runtime-environment-variables"></a>Azure 批次處理運行時環境變數

[Azure Batch 服務](https://azure.microsoft.com/services/batch/)會在計算節點上設定下列環境變數。 您可以在工作命令列中，以及由該命令列執行的程式及指令碼中，參照這些環境變數。

有關將環境變數與 Batch 一起使用的詳細資訊，請參閱[任務的環境設置](https://docs.microsoft.com/azure/batch/batch-api-basics#environment-settings-for-tasks)。

## <a name="environment-variable-visibility"></a>環境變數可見性

這些環境變數僅適用於**工作使用者**的內容中，也就是工作執行所在節點上的使用者帳戶。 如果您透過遠端桌面通訊協定 (RDP) 或安全殼層 (SSH) [遠端連線](https://azure.microsoft.com/documentation/articles/batch-api-basics/#connecting-to-compute-nodes)到計算節點，並列出環境變數，您將*不會*看見這些環境變數。 這是因為用於遠端連線的使用者帳戶與工作所用的帳戶不同。

要獲取環境變數的當前值，在 Windows`cmd.exe`計算節點或`/bin/sh`Linux 節點上啟動：

`cmd /c set <ENV_VARIABLE_NAME>`

`/bin/sh printenv <ENV_VARIABLE_NAME>`

## <a name="command-line-expansion-of-environment-variables"></a>環境變數的命令列擴充功能

工作在計算節點上執行的命令列不會在殼層下執行。 因此，這些命令列無法以原生方式利用如環境變數擴充功能等殼層功能 (這包括 `PATH`)。 若要利用這類功能，您必須在命令列中**叫用殼層**。 例如，在 Windows 計算節點上啟動 `cmd.exe`，或在 Linux 節點上啟動 `/bin/sh`：

`cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

`/bin/sh -c MyTaskApplication $MY_ENV_VAR`

## <a name="environment-variables"></a>環境變數

| 變數名稱                     | 描述                                                              | 可用性 | 範例 |
|-----------------------------------|--------------------------------------------------------------------------|--------------|---------|
| AZ_BATCH_ACCOUNT_NAME           | 工作所屬之批次帳戶的名稱。                  | 所有工作。   | mybatchaccount |
| AZ_BATCH_ACCOUNT_URL            | Batch 帳戶的 URL。 | 所有工作。 | `https://myaccount.westus.batch.azure.com` |
| AZ_BATCH_APP_PACKAGE            | 所有應用程式套件環境變數的前置詞。 例如，如果應用程式"FOO"版本"1"安裝在池上，則環境變數AZ_BATCH_APP_PACKAGE_FOO_1（在 Linux 上）或AZ_BATCH_APP_PACKAGE_FOO#1（在 Windows 上）。 AZ_BATCH_APP_PACKAGE_FOO_1指向包下載的位置（資料夾）。 使用應用包的預設版本時，請使用沒有版本號的AZ_BATCH_APP_PACKAGE環境變數。 如果在 Linux 中，應用程式包名稱為"Agent-linux-x64"，並且版本為"1.1.46.0"，則環境名稱實際上是：AZ_BATCH_APP_PACKAGE_agent_linux_x64_1_1_46_0，使用底線和小寫。 詳細資訊請看[這裡](https://docs.microsoft.com/azure/batch/batch-application-packages#execute-the-installed-applications)。 | 與應用程式套件相關聯的任何工作。 如果節點本身有應用程式套件，也適用於所有的工作。 | AZ_BATCH_APP_PACKAGE_FOO_1（Linux）或AZ_BATCH_APP_PACKAGE_FOO#1（視窗） |
| AZ_BATCH_AUTHENTICATION_TOKEN   | 驗證權杖，可授與一組有限 Batch 服務作業的存取權。 只有在設定[新增工作](/rest/api/batchservice/task/add#request-body)時設定 [authenticationTokenSettings](/rest/api/batchservice/task/add#authenticationtokensettings)，此環境變數才存在。 權杖值會在 Batch API 中作為認證來建立 Batch 用戶端，例如在 [BatchClient.Open() .NET API](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.batchclient.open#Microsoft_Azure_Batch_BatchClient_Open_Microsoft_Azure_Batch_Auth_BatchTokenCredentials_)。 | 所有工作。 | OAuth2 存取權杖 |
| AZ_BATCH_CERTIFICATES_DIR       | 系統為 Linux 計算節點儲存憑證所在[工作工作目錄][files_dirs]內的目錄。 此環境變數不適用於 Windows 計算節點。                                                  | 所有工作。   |  /mnt/batch/tasks/workitems/batchjob001/job-1/task001/certs |
| AZ_BATCH_HOST_LIST              | 配置給[多重執行個體工作][multi_instance]的節點清單，清單格式為 `nodeIP,nodeIP`。 | 多重執行個體的主要工作和子工作。 | `10.0.0.4,10.0.0.5` |
| AZ_BATCH_IS_CURRENT_NODE_MASTER | 指定目前節點是否為[多重執行個體工作][multi_instance]的主要節點。 可能的值是 `true` 和 `false`。| 多重執行個體的主要工作和子工作。 | `true` |
| AZ_BATCH_JOB_ID                 | 工作所屬之作業的 ID。 | 啟動工作之外的所有工作。 | batchjob001 |
| AZ_BATCH_JOB_PREP_DIR           | 節點上作業準備[工作目錄][files_dirs]的完整路徑。 | 啟動工作與作業準備工作之外的所有工作。 僅適用於透過作業準備工作設定作業時。 | C:\user\tasks\workitems\jobprepreleasesamplejob\job-1\jobpreparation |
| AZ_BATCH_JOB_PREP_WORKING_DIR   | 節點上的作業準備[任務工作目錄][files_dirs]的完整路徑。 | 啟動工作與作業準備工作之外的所有工作。 僅適用於透過作業準備工作設定作業時。 | C:\user\tasks\workitems\jobprepreleasesamplejob\job-1\jobpreparation\wd |
| AZ_BATCH_MASTER_NODE            | [多重執行個體工作][multi_instance]的主要工作執行所在的計算節點其 IP 位址與連接埠。 | 多重執行個體的主要工作和子工作。 | `10.0.0.4:6000` |
| AZ_BATCH_NODE_ID                | 將工作指派至該節點的識別碼。 | 所有工作。 | tvm-1219235766_3-20160919t172711z |
| AZ_BATCH_NODE_IS_DEDICATED      | 若為 `true`，目前的節點就是專用節點。 若為 `false`，它就是[低優先順序節點](batch-low-pri-vms.md)。 | 所有工作。 | `true` |
| AZ_BATCH_NODE_LIST              | 配置給[多重執行個體工作][multi_instance]的節點清單，清單格式為 `nodeIP;nodeIP`。 | 多重執行個體的主要工作和子工作。 | `10.0.0.4;10.0.0.5` |
| AZ_BATCH_NODE_MOUNTS_DIR        | 節點級[檔案系統裝載](virtual-file-mount.md)位置的完整路徑，其中所有裝載目錄都駐留。 Windows 檔共用使用磁碟機號，因此對於 Windows，裝載磁碟機是設備和磁碟機的一部分。  |  由於使用者知道裝載的目錄的裝載許可權，所有任務（包括啟動任務）都有權訪問使用者。 | 例如，在 Ubuntu 中，位置是：`/mnt/batch/tasks/fsmounts` |
| AZ_BATCH_NODE_ROOT_DIR          | 節點上所有 [Batch 目錄][files_dirs]其根目錄的完整路徑。 | 所有工作。 | C:\user\tasks |
| AZ_BATCH_NODE_SHARED_DIR        | 節點上[共用目錄][files_dirs]的完整路徑。 在節點上執行的所有工作皆具備此目錄的讀取/寫入存取權。 在其他節點上執行的工作沒有遠端存取此目錄 (它不是「共用的」網路目錄) 的權限。 | 所有工作。 | C:\user\tasks\shared |
| AZ_BATCH_NODE_STARTUP_DIR       | 節點上[啟動工作目錄][files_dirs]的完整路徑。 | 所有工作。 | C:\user\tasks\startup |
| AZ_BATCH_POOL_ID                | 執行工作之集區的 ID。 | 所有工作。 | batchpool001 |
| AZ_BATCH_TASK_DIR               | 節點上[工作目錄][files_dirs]的完整路徑。 此目錄包含工作的 `stdout.txt` 與 `stderr.txt`，以及 AZ_BATCH_TASK_WORKING_DIR。 | 所有工作。 | C:\user\tasks\workitems\batchjob001\job-1\task001 |
| AZ_BATCH_TASK_ID                | 目前工作的 ID。 | 啟動工作之外的所有工作。 | task001 |
| AZ_BATCH_TASK_SHARED_DIR | [多重執行個體工作][multi_instance]的主要工作與每個子工作相同的目錄路徑。 此路徑存在於多重執行個體工作執行所在的每個節點上，且可由在該節點上執行的工作命令 ([協調命令][coord_cmd]與[應用程式命令][app_cmd]) 以讀取/寫入的方式存取。 在其他節點上執行的子任務或主要任務沒有對此目錄的遠端存取（它不是"共用"網路目錄）。 | 多重執行個體的主要工作和子工作。 | C:\user\tasks\workitems\multiinstancesamplejob\job-1\multiinstancesampletask |
| AZ_BATCH_TASK_WORKING_DIR       | 節點上[工作工作目錄][files_dirs]的完整路徑。 目前執行中工作具有此目錄的讀取/寫入存取權。 | 所有工作。 | C:\user\tasks\workitems\batchjob001\job-1\task001\wd |
| CCP_NODES                       | 各節點配置給[多重執行個體工作][multi_instance]的節點清單與核心數目。 列出節點與核心的格式為：`numNodes<space>node1IP<space>node1Cores<space>`<br/>`node2IP<space>node2Cores<space> ...`，其中節點數目後面會加上一或多個節點 IP 位址，及每個節點的核心數目。 |  多重執行個體的主要工作和子工作。 |`2 10.0.0.4 1 10.0.0.5 1` |

[files_dirs]: https://azure.microsoft.com/documentation/articles/batch-api-basics/#files-and-directories
[multi_instance]: https://azure.microsoft.com/documentation/articles/batch-mpi/
[coord_cmd]: https://azure.microsoft.com/documentation/articles/batch-mpi/#coordination-command
[app_cmd]: https://azure.microsoft.com/documentation/articles/batch-mpi/#application-command

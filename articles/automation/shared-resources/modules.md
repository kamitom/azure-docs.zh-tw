---
title: 在 Azure 自動化中管理模組
description: 本文介紹如何在 Azure 自動化中管理模組。
services: automation
ms.service: automation
author: mgoedtel
ms.author: magoedte
ms.date: 01/31/2020
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 605d1bc72406a9aeecc9273f9bd2d7fd2b30ab11
ms.sourcegitcommit: bc738d2986f9d9601921baf9dded778853489b16
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/02/2020
ms.locfileid: "80618698"
---
# <a name="manage-modules-in-azure-automation"></a>在 Azure 自動化中管理模組

您可以將 PowerShell 模組導入 Azure 自動化,使其 cmdlet 在 Runbook 及其 DSC 資源中可用 DSC 配置。 在幕後,Azure 自動化存儲這些模組。 在 Runbook 作業和 DSC 編譯作業執行時間,自動化將它們載入到 Azure 自動化沙箱中,其中執行簿執行和 DSC 配置編譯。 模組中的任何 DSC 資源也會自動放置於 Automation DSC 提取伺服器。 計算機在應用 DSC 配置時可以拔取它們。

Azure 自動化中使用的模組可以是已建立的自定義模組、PowerShell 庫中的模組或 Azure 的 AzureRM 和 Az 模組。 創建自動化帳戶時,默認情況下將導入某些模組。

## <a name="default-modules"></a>預設模組

下表列出了默認情況下在創建自動化帳戶時導入的模組。 自動化可以導入這些模組的較新版本。 但是,即使您刪除了較新版本,也不能從自動化帳戶中刪除原始版本。

|模組名稱|版本|
|---|---|
| 審計政策 | 1.1.0.0 |
| Azure | 1.0.3 |
| Azure.Storage | 1.0.3 |
| AzureRM.Automation | 1.0.3 |
| AzureRM.Compute | 1.2.1 |
| AzureRM.Profile | 1.0.3 |
| AzureRM.Resources | 1.0.3 |
| AzureRM.Sql | 1.0.3 |
| AzureRM.Storage | 1.0.3 |
| ComputerManagementDsc | 5.0.0.0 |
| GP註冊政策分析器 | 0.2 |
| Microsoft.PowerShell.Core | 0 |
| Microsoft.PowerShell.Diagnostics |  |
| Microsoft.PowerShell.Management |  |
| Microsoft.PowerShell.Security |  |
| Microsoft.PowerShell.Utility |  |
| Microsoft.WSMan.Management |  |
| 協調器.資產管理.Cmdlet | 1 |
| 私營部門服務資源 | 2.9.0.0 |
| 安全政策 | 2.1.0.0 |
| 國家配置綜合資源 | 1 |
| xDSCDomainjoin | 1.1 |
| xPowerShell 執行政策 | 1.1.0.0 |
| x遠端桌面管理員 | 1.1.0.0 |

## <a name="internal-cmdlets"></a>內部公分

下表列出了導入到每個自動化帳戶的內部`Orchestrator.AssetManagement.Cmdlets`模組中的cmdlet。 這些 cmdlet 可在 Runbook 和 DSC 設定中訪問,並允許您與自動化帳戶中的資產進行交互。 此外,內部 cmdlet 允許您從加密的變數、憑據和加密連接中檢索機密。 Azure PowerShell cmdlet 無法檢索這些機密。 這些 cmdlet 不需要在使用它們時隱式連接到 Azure,就像使用「運行作為帳戶」對 Azure 進行身份驗證時一樣。

>[!NOTE]
>這些內部 cmdlet 在 Windows 混合 Runbook 輔助角色上可用,但在 Linux 混合 Runbook 輔助功能程式上不可用。 對於直接在電腦上運行的 Runbook 或針對環境中的資源,請使用相應的[AzureRM.自動化](https://docs.microsoft.com/powershell/module/AzureRM.Automation/?view=azurermps-6.13.0)或[Az 模組](../az-modules.md)cmdlet。 

|名稱|描述|
|---|---|
|Get-AutomationCertificate|`Get-AutomationCertificate [-Name] <string> [<CommonParameters>]`|
|Get-AutomationConnection|`Get-AutomationConnection [-Name] <string> [-DoNotDecrypt] [<CommonParameters>]` |
|Get-AutomationPSCredential|`Get-AutomationPSCredential [-Name] <string> [<CommonParameters>]` |
|Get-AutomationVariable|`Get-AutomationVariable [-Name] <string> [-DoNotDecrypt] [<CommonParameters>]`|
|Set-AutomationVariable|`Set-AutomationVariable [-Name] <string> -Value <Object> [<CommonParameters>]` |
|啟動-自動化執行簿|`Start-AutomationRunbook [-Name] <string> [-Parameters <IDictionary>] [-RunOn <string>] [-JobId <guid>] [<CommonParameters>]`|
|等待自動化作業|`Wait-AutomationJob -Id <guid[]> [-TimeoutInMinutes <int>] [-DelayInSeconds <int>] [-OutputJobsTransitionedToRunning] [<CommonParameters>]`|

## <a name="importing-modules"></a>匯入模組

您可以通過多種方式將模組導入自動化帳戶。 以下各節顯示了導入模組的不同方法。

> [!NOTE]
> Azure 自動化中使用的模組中檔的最大路徑大小為 140 個字元。 自動化無法將路徑大小超過 140 個字元的檔案匯入到 具有的 PowerShell 工作階段中`Import-Module`。

### <a name="import-modules-in-azure-portal"></a>Azure 門戶中匯入模組

要在 Azure 門戶中導入模組,請參閱:

1. 瀏覽至您的自動化帳戶。
2. 在**分享資源**下選擇**模組**。
3. 按下 **「添加模組**」。 
4. 選擇包含模組的 **.zip**檔。
5. 按下 **「確定**」開始導入過程。

### <a name="import-modules-using-powershell"></a>使用 PowerShell 匯入模組

您可以使用[「新建 AzureRm 自動化模組](/powershell/module/azurerm.automation/new-azurermautomationmodule)cmdlet」將模組導入自動化帳戶。 cmdlet 取得模組 .zip 包的 URL。

```azurepowershell-interactive
New-AzureRmAutomationModule -Name <ModuleName> -ContentLinkUri <ModuleUri> -ResourceGroupName <ResourceGroupName> -AutomationAccountName <AutomationAccountName>
```

您還可以使用相同的 cmdlet 直接從 PowerShell 庫導入模組。 一定要抓住`ModuleName`並`ModuleVersion`從[PowerShell畫廊](https://www.powershellgallery.com)。

```azurepowershell-interactive
$moduleName = <ModuleName>
$moduleVersion = <ModuleVersion>
New-AzureRmAutomationModule -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -Name $moduleName -ContentLinkUri "https://www.powershellgallery.com/api/v2/package/$moduleName/$moduleVersion"
```
### <a name="import-modules-from-powershell-gallery"></a>從 PowerShell 函式庫匯入模組

您可以直接從庫或從自動化帳戶導入[PowerShell 庫](https://www.powershellgallery.com)模組。

要直接從 PowerShell 函式庫匯入模組:

1. 跳到https://www.powershellgallery.com並搜索要導入的模組。
2. 按下 **「在****」安裝選項**「下的**Azure 自動化**選項卡上部署到 Azure 自動化。 此動作會開啟 Azure 入口網站。 
3. 在「導入」頁上,選擇您的自動化帳戶,然後單擊 **「確定**」。

![PowerShell 函式庫匯入模組](../media/modules/powershell-gallery.png)

要直接從您的自動化帳戶導入 PowerShell 函式庫模組,請:

1. 在**分享資源**下選擇**模組**。 
2. 在"模組"頁上,按一下 **「流覽庫**」,然後搜索庫中的模組。 
3. 選擇要導入的模組,然後單擊「**導入**」。 
4. 在「導入」頁上,按一下「**確定」** 以啟動導入過程。

![從 Azure 門戶導入電源外殼庫](../media/modules/gallery-azure-portal.png)

## <a name="deleting-modules"></a>移除模組

如果模組出現問題或需要回滾到模組的早期版本,則可以將其從自動化帳戶中刪除。 無法刪除創建自動化帳戶時導入的[預設模組](#default-modules)的原始版本。 如果要刪除的模組是[預設模組](#default-modules)之一的較新版本,則它回滾到與自動化帳戶一起安裝的版本。 否則,您從自動化帳戶中刪除的任何模組都將被刪除。

### <a name="delete-modules-in-azure-portal"></a>移除 Azure 門戶中的模組

要刪除 Azure 門戶中的模組,請執行以下工作:

1. 導覽您的自動化帳戶,並在**分享資源**下選擇**模組**。 
2. 選擇要刪除的模組。 
3. 在 **「模組」** 頁上,選擇 **「刪除**」。 如果此模組是[預設模組](#default-modules)之一,它將回滾到創建自動化帳戶時存在的版本。

### <a name="delete-modules-using-powershell"></a>使用 PowerShell 移除模組

要透過 PowerShell 刪除模組,請執行以下指令:

```azurepowershell-interactive
Remove-AzureRmAutomationModule -Name <moduleName> -AutomationAccountName <automationAccountName> -ResourceGroupName <resourceGroupName>
```
## <a name="adding-a-connection-type-to-your-module"></a>新增連線類型

您可以透過模組加入選擇的資料檔,提供在自動化帳戶中使用的自訂[連接類型](../automation-connections.md)。 此檔指定要與自動化帳戶中的模組 cmdlet 一起使用的 Azure 自動化連接類型。 為此,您必須首先瞭解如何編寫 PowerShell 模組。 請參考[如何編寫 PowerShell 文稿模組](/powershell/scripting/developer/module/how-to-write-a-powershell-script-module)。

![在 Azure 門戶中使用自訂連線](../media/modules/connection-create-new.png)

指定連接類型屬性的檔案名為**&lt;&gt;ModuleName -自動化.json,** 位於壓縮 **.zip**檔案的模組資料夾中。 此檔包含連接到模組表示的系統或服務所需的連接欄位。 配置允許在 Azure 自動化中創建連接類型。 使用此檔案,您可以設定欄位名稱、類型以及欄位是加密還是模組的連接類型的可選。 下面的範例是 **.json**檔案格式的樣本,用於定義使用者名稱和密碼屬性:

```json
{
   "ConnectionFields": [
   {
      "IsEncrypted":  false,
      "IsOptional":  true,
      "Name":  "Username",
      "TypeName":  "System.String"
   },
   {
      "IsEncrypted":  true,
      "IsOptional":  false,
      "Name":  "Password",
      "TypeName":  "System.String"
   }
   ],
   "ConnectionTypeName":  "MyModuleConnection",
   "IntegrationModuleName":  "MyModule"
}
```

## <a name="best-practices-for-authoring-modules"></a>編寫模組的最佳做法

我們建議您在編寫 PowerShell 模組以在 Azure 自動化中使用時遵循本節中的注意事項。

### <a name="version-folder"></a>版本資料夾

請勿在模組的 **.zip**包中包含版本資料夾。  此問題不太適合 Runbook,但確實會導致狀態配置服務出現問題。 當模組分發到由 DSC 管理的節點時,Azure 自動化會自動創建版本資料夾。 如果存在版本資料夾,則最終有兩個實例。 下面是 DSC 模組的範例資料夾結構:

```powershell
myModule
  - DSCResources
    - myResourceFolder
      myResourceModule.psm1
      myResourceSchema.mof
  myModuleManifest.psd1
```

### <a name="help-information"></a>說明資訊

包括模組中每個 cmdlet 的概要、說明和説明 URI。 在 PowerShell 中,您`Get-Help`可以使用 cmdlet 為 cmdlet 定義説明資訊。 下面的範例展示如何在 **.psm1**模組檔案中定義概要並説明 URI:

  ```powershell
  <#
       .SYNOPSIS
        Gets a Contoso User account
  #>
  function Get-ContosoUser {
  [CmdletBinding](DefaultParameterSetName='UseConnectionObject', `
  HelpUri='https://www.contoso.com/docs/information')]
  [OutputType([String])]
  param(
     [Parameter(ParameterSetName='UserAccount', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [string]
     $UserName,

     [Parameter(ParameterSetName='UserAccount', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [string]
     $Password,

     [Parameter(ParameterSetName='ConnectionObject', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [Hashtable]
     $Connection
  )

  switch ($PSCmdlet.ParameterSetName) {
     "UserAccount" {
        $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $UserName, $Password
        Connect-Contoso -Credential $cred
     }
     "ConnectionObject" {
        Connect-Contoso -Connection $Connection
    }
  }
  }
  ```

  提供此資訊透過 PowerShell`Get-Help`主控台中的 cmdlet 顯示説明文字。 此文本也顯示在 Azure 門戶中。

  ![整合模組說明](../media/modules/module-activity-description.png)

### <a name="connection-type"></a>連線類型

若是模組連線到外部服務,請定義[連線類型](#adding-a-connection-type-to-your-module)。 模組中的每個 cmdlet 都應接受連接物件(該連接類型的實例)作為參數。 使用者每次調用 cmdlet 時都會將連接資產的參數映射到 cmdlet 的相應參數。 基於上面的 Runbook 範例,它使用`ContosoConnection`調用的 Contoso 連接資產的範例來存取 Contoso 資源並從外部服務返回數據。

  在下面的範例中,欄位映射到`UserName``Password``PSCredential`物件的和屬性,然後傳遞到 cmdlet。

  ```powershell
  $contosoConnection = Get-AutomationConnection -Name 'ContosoConnection'

  $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $contosoConnection.UserName, $contosoConnection.Password
  Connect-Contoso -Credential $cred
  }
  ```

  處理此行為的一種更簡單、更好的方法是將連接物件直接傳遞到 cmdlet:

  ```powershell
  $contosoConnection = Get-AutomationConnection -Name 'ContosoConnection'

  Connect-Contoso -Connection $contosoConnection
  }
  ```

  您可以通過允許 cmdlet 直接接受連接物件作為參數,而不是僅為參數接受連接欄位,從而為 cmdlet 啟用類似的行為。 通常您會想讓每個 Cmdlet 都設定參數，讓未使用 Azure 自動化的使用者可以直接呼叫 Cmdlet，而不必建構雜湊表來作為連線物件。 參數集`UserAccount`, 用於傳遞連接欄位屬性。 `ConnectionObject`允許您直接傳遞連接。

### <a name="output-type"></a>輸出類型

定義模組中所有 cmdlet 的輸出類型。 為 Cmdlet 定義輸出類型，可讓設計階段 IntelliSense 協助您判斷 Cmdlet 的輸出屬性，以供在撰寫期間使用。 在自動化 Runbook 圖形創作期間,這種做法尤其有用,為此,設計時間知識是輕鬆體驗模組的關鍵。

添加`[OutputType([<MyOutputType>])]``MyOutputType`位置 是有效的類型。 要瞭解有關詳細資訊`OutputType`,請參閱[有關函數輸出類型屬性](/powershell/module/microsoft.powershell.core/about/about_functions_outputtypeattribute)。 以下代碼是新增到`OutputType`cmdlet 的範例:

  ```powershell
  function Get-ContosoUser {
  [OutputType([String])]
  param(
     [string]
     $Parameter1
  )
  # <script location here>
  }
  ```

  ![圖形化 Runbook 輸出類型](../media/modules/runbook-graphical-module-output-type.png)

  此行為類似於 Cmdlet 在 PowerShell ISE 中的輸出的「自動提示」功能，但不需要加以執行。

  ![POSH IntelliSense](../media/modules/automation-posh-ise-intellisense.png)

### <a name="cmdlet-state"></a>Cmdlet 狀態

使模組中的所有 cmdlet 都無狀態。 多個 Runbook 作業可以在`AppDomain`同一 進程和沙箱中同時運行。 如果這些級別上共用任何狀態,則作業可能會相互影響。 此行為可能導致間歇性和難以診斷的問題。 下面是不該做什麼的範例:

  ```powershell
  $globalNum = 0
  function Set-GlobalNum {
     param(
         [int] $num
     )

     $globalNum = $num
  }
  function Get-GlobalNumTimesTwo {
     $output = $globalNum * 2

     $output
  }
  ```

### <a name="module-dependency"></a>模組相依項

確保模組完全包含在可 xcopy 的封裝中。 執行 Runbook 時,Azure 自動化模組將分發到自動化沙箱。 模組必須獨立於運行它們的主機工作。 

您應該能夠壓縮和移動模組包,並在導入到另一個主機的 PowerShell 環境中時使其正常工作。 為此,請確保模組不依賴於模組資料夾外的任何文件,這些檔在將模組導入 Azure 自動化時壓縮。 

您的模組不應依賴於主機上的任何唯一註冊表設置。 例如,安裝產品時所做的設置。 

確保模組中的所有檔具有少於 140 個字元的路徑。 任何超過 140 個字元的路徑都會導致導入 Runbook 出現問題。 如果不遵循此最佳實踐,該模組在 Azure 自動化中不可用。  

### <a name="references-to-azurerm-and-az"></a>對 AzureRM 與 Az 的參考

如果在模組中引用[Azure PowerShell Az 模組](/powershell/azure/new-azureps-module-az?view=azps-1.1.0),請確保不引用 AzureRM。 不能將 Az 模組與 AzureRM 模組結合使用。 在 Runbook 中支援 Az,但默認情況下不導入。 要瞭解 Az 模組和注意事項,請參閱[Azure 自動化 中的 Az 模組支援](../az-modules.md)。

## <a name="next-steps"></a>後續步驟

* 要瞭解有關建立 PowerShell 模組的更多資訊,請參閱[編寫 Windows PowerShell 模組](/powershell/scripting/developer/windows-powershell)。

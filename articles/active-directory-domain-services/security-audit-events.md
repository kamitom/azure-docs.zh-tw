---
title: 為 Azure AD 域服務啟用安全審核 |微軟文件
description: 瞭解如何啟用安全稽核以集中紀錄 Azure AD 網域服務中的分析和警報事件
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 662362c3-1a5e-4e94-ae09-8e4254443697
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 02/10/2020
ms.author: iainfou
ms.openlocfilehash: ce910b553e14d09eefa35efc5f2973337dfa1309
ms.sourcegitcommit: 62c5557ff3b2247dafc8bb482256fef58ab41c17
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2020
ms.locfileid: "80654667"
---
# <a name="enable-security-audits-for-azure-active-directory-domain-services"></a>為 Azure 活動目錄域服務啟用安全稽核

Azure 活動目錄域服務 (Azure AD DS) 安全審核允許 Azure 將安全事件流式傳輸到目標資源。 這些資源包括 Azure 存儲、Azure 日誌分析工作區或 Azure 事件中心。 啟用安全審核事件後,Azure ADDS 會將所選類別的所有審核事件發送到目標資源。

您可以使用 Azure 事件中心將事件存檔到 Azure 儲存中,並將事件流式傳輸到安全資訊和事件管理 (SIEM) 軟體(或等效軟體),或者執行自己的分析和使用 Azure 門戶中的 Azure 日誌分析工作區。

> [!IMPORTANT]
> Azure AD DS 安全審核僅適用於基於 Azure 資源管理員的實例。 有關如何遷移的資訊,請參閱[將 Azure AD DS 從傳統網路模型遷移到資源管理員][migrate-azure-adds]。

## <a name="security-audit-destinations"></a>安全稽核目標

可以將 Azure 儲存、Azure 事件中心或 Azure 日誌分析工作區用作 Azure AD DS 安全審核的目標資源。 這些目的地可以組合。 例如,可以使用 Azure 儲存存檔安全審核事件,但使用 Azure 日誌分析工作區來分析和報告資訊。

下表概述了每種目標資源類型的方案。

> [!IMPORTANT]
> 在啟用 Azure AD DS 安全審核之前,需要創建目標資源。 可以使用 Azure 門戶、Azure PowerShell 或 Azure CLI 創建這些資源。

| 目標資源 | 狀況 |
|:---|:---|
|Azure 儲存體| 當您的主要需要是存儲安全審核事件以用於存檔時,應使用此目標。 其他目標可用於存檔目的,但這些目標提供了超出存檔主要需求的功能。 <br /><br />在啟用 Azure AD DS 安全稽核事件之前,請先[建立 Azure 儲存帳戶](../storage/common/storage-account-create.md)。|
|Azure 事件中心| 當您的主要需求是與其他軟體(如數據分析軟體或安全資訊&事件管理 (SIEM) 軟體共用安全審核事件時,應使用此目標。<br /><br />在開啟 Azure AD DS 安全稽核事件之前,[請使用 Azure 門戶建立事件中心](https://docs.microsoft.com/azure/event-hubs/event-hubs-create)|
|Azure 紀錄分析工作區| 當您的主要需要是直接分析和查看 Azure 門戶的安全審核時,應使用此目標。<br /><br />在啟用 Azure AD DS 安全稽核事件之前,[請在 Azure 門戶中建立日誌分析工作區。](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)|

## <a name="enable-security-audit-events-using-the-azure-portal"></a>使用 Azure 門戶啟用安全稽核事件

要使用 Azure 門戶啟用 Azure AD DS 安全審核事件,請使用以下步驟完成。

> [!IMPORTANT]
> Azure AD DS 安全審核不具有追溯性。 無法從過去檢索或重播事件。 Azure AD DS 只能發送啟用安全審核後發生的事件。

1. 在 https://portal.azure.com 登入 Azure 入口網站。
1. 在 Azure 門戶的頂部,搜尋並選擇**Azure AD 網域服務**。 選擇託管域,如*aaddscontoso.com*。
1. 在 Azure AD DS 視窗中, 選擇左邊的**診斷設定**。
1. 默認情況下不配置任何診斷。 要開始,請選擇 **「添加診斷設置**」。

    ![為 Azure AD 網域服務新增診斷設定](./media/security-audit-events/add-diagnostic-settings.png)

1. 輸入診斷設定的名稱,如*adds 稽核*。

    選中所需的安全審核目標複選框。 您可以從 Azure 儲存帳戶、Azure 事件中心或日誌分析工作區中進行選擇。 這些目標資源必須已存在於 Azure 訂閱中。 無法在此嚮導中創建目標資源。

    ![開啟所需的稽核事件的目標與類型以捕捉](./media/security-audit-events/diagnostic-settings-page.png)

    * **Azure 儲存**
        * 選擇 **「存檔到儲存帳戶**」,然後選擇「**設定**」 。
        * 選擇要用於存取安全稽核事件的**訂閱**與**儲存帳戶**。
        * 準備就緒后,選擇 **"確定**"。
    * **Azure 事件中心**
        * 選擇 **「流到事件中心**」,然後選擇 **「設定**」 。。
        * 選擇**訂閱**與**事件中心命名空間**。 如果需要,還可以選擇**事件中心名稱**,然後選擇**事件中心策略名稱**。
        * 準備就緒后,選擇 **"確定**"。
    * **Azure 紀錄分析工作區**
        * 選擇 **「傳送到紀錄分析**」,然後選擇要用於儲存安全稽核事件的**訂閱**與**紀錄分析工作區**。

1. 選擇要為特定目標資源包括的日誌類別。 如果將審核事件發送到 Azure 儲存帳戶,還可以配置定義保留數據的天數的保留策略。 默認設置*0*保留所有數據,並且不會在一段時間後旋轉事件。

    您可以在單個配置中為每個目標資源選擇不同的日誌類別。 例如,此功能允許您選擇要為日誌分析保留的日誌類別以及要存檔的日誌類別。

1. 完成後,選擇 **「保存」** 以提交更改。 目標資源在保存配置后不久開始接收 Azure AD DS 安全審核事件。

## <a name="enable-security-audit-events-using-azure-powershell"></a>使用 Azure PowerShell 啟用安全稽核事件

要使用 Azure PowerShell 啟用 Azure AD DS 安全審核事件,請使用以下步驟完成。 如果需要,請先[安裝 Azure PowerShell 模組並連接到 Azure 訂閱](/powershell/azure/install-az-ps)。

> [!IMPORTANT]
> Azure AD DS 安全審核不具有追溯性。 無法從過去檢索或重播事件。 Azure AD DS 只能發送啟用安全審核後發生的事件。

1. 使用[連接-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount) cmdlet 對 Azure 訂閱進行身份驗證。 出現提示后,輸入您的帳戶憑據。

    ```azurepowershell
    Connect-AzAccount
    ```

1. 為安全審核事件創建目標資源。

    * **Azure 儲存** - [使用 Azure PowerShell 建立儲存帳戶](../storage/common/storage-account-create.md?tabs=azure-powershell)
    * **Azure 事件中心** - [使用 Azure PowerShell 建立事件中心](../event-hubs/event-hubs-quickstart-powershell.md)。 您可能還需要使用[New-AzEventHub 授權規則](/powershell/module/az.eventhub/new-azeventhubauthorizationrule)cmdlet 來建立授權規則,將 Azure AD DS 權限授權的事件中心*命名空間*。 授權規則必須包括 **「管理**」、**偵聽**和**發送**許可權。

        > [!IMPORTANT]
        > 請確保在事件中心命名空間而不是事件中心本身上設置授權規則。

    * **Azure 紀錄分析工作區** - [使用 Azure PowerShell 建立紀錄分析工作區](../azure-monitor/learn/quick-create-workspace-posh.md)。

1. 使用[獲取-AzResource](/powershell/module/Az.Resources/Get-AzResource) cmdlet 獲取 Azure AD DS 託管域的資源 ID。 建立名為$aadds的變數 *。要儲存*該值的資源代碼:

    ```azurepowershell
    $aadds = Get-AzResource -name aaddsDomainName
    ```

1. 使用[「設定-Az診斷設定](/powershell/module/Az.Monitor/Set-AzDiagnosticSetting)」cmdlet 設定 Azure 診斷設定,以便使用 Azure AD 域服務安全稽核事件的目標資源。 在以下示例中,變數 *$aadds。資源 Id*從上一步使用。

    * **Azure 儲存**- 儲存*帳號 Id*為儲存帳戶名稱:

        ```powershell
        Set-AzDiagnosticSetting `
            -ResourceId $aadds.ResourceId `
            -StorageAccountId storageAccountId `
            -Enabled $true
        ```

    * **Azure 事件中心**- 將*事件 HubName*取代為事件中心的名稱,並將*事件HubRuleId*取代為授權規則 ID:

        ```powershell
        Set-AzDiagnosticSetting -ResourceId $aadds.ResourceId `
            -EventHubName eventHubName `
            -EventHubAuthorizationRuleId eventHubRuleId `
            -Enabled $true
        ```

    * **Azure 紀錄分析工作區**-*將 工作區 Id*取代為紀錄分析工作區的 ID:

        ```powershell
        Set-AzureRmDiagnosticSetting -ResourceId $aadds.ResourceId `
            -WorkspaceID workspaceId `
            -Enabled $true
        ```

## <a name="query-and-view-security-audit-events-using-azure-monitor"></a>使用 Azure 監視器查詢及檢視安全稽核事件

以日誌分析工作區,您可以使用 Azure 監視器和 Kusto 查詢語言查看和分析安全審核事件。 此查詢語言專為唯讀使用而設計,具有易於閱讀的語法的電源分析功能。 有關開始使用 Kusto 查詢語言的詳細資訊,請參閱以下文章:

* [Azure 監視器文件](https://docs.microsoft.com/azure/azure-monitor/)
* [開始使用 Azure 監視器 Log Analytics](../azure-monitor/log-query/get-started-portal.md)
* [開始使用 Azure 監視器中的查詢](../azure-monitor/log-query/get-started-queries.md)
* [建立和共用 Log Analytics 資料的儀表板](../azure-monitor/learn/tutorial-logs-dashboards.md)

以下範例查詢可用於開始分析 Azure AD DS 的安全審核事件。

### <a name="sample-query-1"></a>範例查詢 1

檢視過去七天的所有帳戶鎖定事件:

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= ago(7d)
| where OperationName has "4740"
```

### <a name="sample-query-2"></a>範例查詢 2

查看 2020 年 2 月 3 日上午 9 點之間的所有帳戶鎖定事件 (*4740)。* 和 2020 年 2 月 10 日午夜,按日期和時間排序:

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= datetime(2020-02-03 09:00) and TimeGenerated <= datetime(2020-02-10)
| where OperationName has "4740"
| sort by TimeGenerated asc
```

### <a name="sample-query-3"></a>範例查詢 3

檢視七天前(即日起)名為使用者的帳戶登錄事件:

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-4"></a>範例查詢 4

檢視帳號登入事件七天前從現在開始,帳戶命名使用者試圖使用錯誤密碼登入 (*0xC0000006a*):

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| where "0xc000006a" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-5"></a>範例查詢 5

檢視帳號登入事件七天前從現在開始,帳戶命名使用者試圖在帳戶鎖定時登入 (*0xC0000234*):

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| where "0xc0000234" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-6"></a>範例查詢 6

檢視七天前的帳戶登入事件數,查看所有鎖定使用者發生的所有登入嘗試:

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "0xc0000234" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| summarize count()
```

## <a name="audit-event-categories"></a>稽核事件類別

Azure AD DS 安全審核與傳統 AD DS 域控制器的傳統審核一致。 在混合環境中,您可以重用現有的審核模式,以便在分析事件時使用相同的邏輯。 根據需要排除故障或分析的方案,需要針對不同的審核事件類別。

以下稽核事件類別可用:

| 稽核類別名稱 | 描述 |
|:---|:---|
| 帳號登入|審核嘗試對域控制器或本地安全帳戶管理員 (SAM) 上的帳戶數據進行身份驗證。</p>登錄和註銷策略設置和事件跟蹤訪問特定計算機的嘗試。 此類別中的設置和事件側重於所使用的帳戶資料庫。 此類別包括以下子類別:<ul><li>[稽核認證驗證](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-credential-validation)</li><li>[Kerbero 驗證的新功能](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-kerberos-authentication-service)</li><li>[稽核 Kerberos 服務票證作業](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-kerberos-service-ticket-operations)</li><li>[稽核其他登入/登出事件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-logonlogoff-events)</li></ul>|
| 帳戶管理|審核對使用者和計算機帳戶和組的更改。 此類別包括以下子類別:<ul><li>[稽核應用程式群組管理](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-application-group-management)</li><li>[稽核電腦帳戶管理](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-computer-account-management)</li><li>[稽核通訊群組管理](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-distribution-group-management)</li><li>[稽核其他帳戶管理](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-account-management-events)</li><li>[稽核安全性群組管理](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-security-group-management)</li><li>[稽核使用者帳戶管理](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-user-account-management)</li></ul>|
| 詳細資訊追蹤|審核該電腦上的單個應用程式和用戶的活動,並了解計算機的使用方式。 此類別包括以下子類別:<ul><li>[稽核 DPAPI 活動](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-dpapi-activity)</li><li>[稽核PNP活動](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-pnp-activity)</li><li>[稽核程序建立](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-process-creation)</li><li>[稽核程序終止](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-process-termination)</li><li>[稽核 RPC 事件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-rpc-events)</li></ul>|
| 目錄服務存取|審核嘗試存取和修改活動目錄域服務 (AD DS) 中的物件。 這些審核事件僅記錄在域控制器上。 此類別包括以下子類別:<ul><li>[稽核詳細目錄服務複寫](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-detailed-directory-service-replication)</li><li>[稽核目錄服務存取](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-directory-service-access)</li><li>[稽核目錄服務變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-directory-service-changes)</li><li>[稽核目錄服務複寫](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-directory-service-replication)</li></ul>|
| 登入-登出|審核嘗試以交互方式或通過網路登錄到計算機。 這些事件可用於跟蹤使用者活動和識別對網路資源的潛在攻擊。 此類別包括以下子類別:<ul><li>[稽核帳戶鎖定](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-account-lockout)</li><li>[稽核使用者/裝置宣告](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-user-device-claims)</li><li>[稽核 IPsec 延伸模式](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-ipsec-extended-mode)</li><li>[稽核群組成員資格](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-group-membership)</li><li>[稽核 IPsec 主要模式](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-ipsec-main-mode)</li><li>[稽核 IPsec 快速模式](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-ipsec-quick-mode)</li><li>[稽核登出](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-logoff)</li><li>[稽核登入](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-logon)</li><li>[稽核網路原則伺服器](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-network-policy-server)</li><li>[稽核其他登入/登出事件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-logonlogoff-events)</li><li>[稽核特殊登入](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-special-logon)</li></ul>|
|物件存取| 審核嘗試存取網路或電腦上的特定物件或物件類型。 此類別包括以下子類別:<ul><li>[稽核已產生的應用程式](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-application-generated)</li><li>[稽核憑證服務](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-certification-services)</li><li>[稽核詳細檔案共用](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-detailed-file-share)</li><li>[稽核檔案共用](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-file-share)</li><li>[審計檔案系統](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-file-system)</li><li>[稽核篩選平台連線](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-filtering-platform-connection)</li><li>[稽核篩選平台封包丟棄](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-filtering-platform-packet-drop)</li><li>[稽核柄操作](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-handle-manipulation)</li><li>[稽核物件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-kernel-object)</li><li>[稽核其他物件存取事件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-object-access-events)</li><li>[審計登記處](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-registry)</li><li>[稽核可移動記憶體](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-removable-storage)</li><li>[稽核 SAM](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-sam)</li><li>[稽核中央存取原則階段](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-central-access-policy-staging)</li></ul>|
|原則變更|審核對本地系統或網路上的重要安全策略的更改。 策略通常由管理員建立,以幫助保護網路資源。 監視更改或嘗試更改這些策略可能是網路安全管理的一個重要方面。 此類別包括以下子類別:<ul><li>[稽核稽核原則變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-audit-policy-change)</li><li>[稽核驗證原則變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-authentication-policy-change)</li><li>[稽核授權原則變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-authorization-policy-change)</li><li>[稽核篩選平台原則變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-filtering-platform-policy-change)</li><li>[稽核 MPSSVC 規則層級原則變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-mpssvc-rule-level-policy-change)</li><li>[稽核其他原則變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-policy-change-events)</li></ul>|
|權限使用| 審核對一個或多個系統使用某些許可權。 此類別包括以下子類別:<ul><li>[稽核非機密特殊權限使用](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-non-sensitive-privilege-use)</li><li>[稽核機密特殊權限使用](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-sensitive-privilege-use)</li><li>[稽核其他特殊權限使用事件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-privilege-use-events)</li></ul>|
|系統| 審核對未包含在其他類別中且具有潛在安全隱患的計算機的系統級更改。 此類別包括以下子類別:<ul><li>[稽核 IPSec 驅動程式](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-ipsec-driver)</li><li>[稽核其他系統事件](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-other-system-events)</li><li>[稽核安全性狀態變更](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-security-state-change)</li><li>[稽核安全性系統延伸](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-security-system-extension)</li><li>[稽核系統整合性](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-system-integrity)</li></ul>|

## <a name="event-ids-per-category"></a>每個類別的事件 ID

 當特定操作觸發可審核事件時,Azure AD DS 安全審核將記錄以下事件 ID:

| 事件類別名稱 | 事件識別碼 |
|:---|:---|
|帳號登入安全性|4767, 4774, 4775, 4776, 4777|
|帳戶管理安全性|4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4730, 4731, 4732, 4733, 4734, 4735, 4737, 4738, 4740, 4741, 4742, 4743, 4754, 4755, 4756, 4757, 4758, 4764, 4765, 4766, 4780, 4781, 4782, 4793, 4798, 4799, 5376, 5377|
|詳細資訊追蹤安全性|None|
|DS 存取安全性|5136, 5137, 5138, 5139, 5141|
|登入-登出安全性|4624, 4625, 4634, 4647, 4648, 4672, 4675, 4964|
|物件存取安全性|None|
|原則變更安全性|4670, 4703, 4704, 4705, 4706, 4707, 4713, 4715, 4716, 4717, 4718, 4719, 4739, 4864, 4865, 4866, 4867, 4904, 4906, 4911, 4912|
|特權 使用安全性|4985|
|系統安全性|4612, 4621|

## <a name="next-steps"></a>後續步驟

有關 Kusto 的具體資訊,請參閱以下文章:

* 函式庫的查詢語言的[概述](/azure/kusto/query/)。
* [Kusto 教程](/azure/kusto/query/tutorial),讓您熟悉查詢基礎知識。
* [幫助您](/azure/kusto/query/samples)學習查看數據的新方法的範例查詢。
* Kusto[最佳實務](/azure/kusto/query/best-practices),以最佳化您的查詢以取得成功。

<!-- LINKS - Internal -->
[migrate-azure-adds]: migrate-from-classic-vnet.md

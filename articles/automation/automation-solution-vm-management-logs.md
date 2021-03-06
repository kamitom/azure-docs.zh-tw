---
title: 如何從啟動/停止 VM 解決方案查詢紀錄
description: 本文介紹如何查詢 Azure 監視器中的開始/停止 VM 解決方案生成的日誌數據。
services: automation
ms.subservice: process-automation
ms.date: 04/01/2020
ms.topic: conceptual
ms.openlocfilehash: 00f6a5f88a66f0a19943ff3a2c722ae1a9938e9e
ms.sourcegitcommit: 980c3d827cc0f25b94b1eb93fd3d9041f3593036
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/02/2020
ms.locfileid: "80550392"
---
# <a name="how-to-query-logs-from-startstop-vms-solution"></a>如何從啟動/停止 VM 解決方案查詢紀錄

Azure 自動化將兩種類型的記錄轉發到連結的日誌分析工作區:作業日誌和作業流。 此資料可用於 Azure 監視器中的[查詢](../azure-monitor/log-query/log-query-overview.md)。

## <a name="job-logs"></a>作業記錄

|屬性 | 描述|
|----------|----------|
|呼叫者 |  起始作業的人員。 可能的值為電子郵件地址或排程作業的系統。|
|類別 | 資料類型的分類。 對自動化來說，該值是 JobLogs。|
|CorrelationId | Runbook 作業之相互關聯識別碼的 GUID。|
|JobId | Runbook 作業之識別碼的 GUID。|
|operationName | 指定在 Azure 中執行的作業類型。 對自動化來說，該值是 Job。|
|resourceId | 指定 Azure 中的資源類型。 對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
|ResourceGroup | 指定 Runbook 作業的資源群組名稱。|
|ResourceProvider | 指定 Azure 服務，以提供您可部署及管理的資源。 對自動化來說，此值是 Azure 自動化。|
|ResourceType | 指定 Azure 中的資源類型。 對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
|resultType | Runbook 作業的狀態。 可能的值包括：<br>- Started (已啟動)<br>- Stopped (已停止)<br>- Suspended (暫止)<br>- Failed (失敗)<br>- Succeeded|
|resultDescription | 說明 Runbook 作業的結果狀態。 可能的值包括：<br>- Job is started (工作已啟動)<br>- Job Failed (工作失敗)<br>- Job Completed|
|RunbookName | 指定 Runbook 的名稱。|
|SourceSystem | 指定所提交資料的來源系統。 對自動化來說，該值是 OpsManager|
|StreamType | 指定事件的類型。 可能的值包括：<br>- Verbose<br>- Output<br>- Error (錯誤)<br>- Warning (警告)|
|SubscriptionId | 指定作業的訂用帳戶 ID。
|Time | Runbook 作業的執行日期和時間。|

## <a name="job-streams"></a>作業串流

|屬性 | 描述|
|----------|----------|
|呼叫者 |  起始作業的人員。 可能的值為電子郵件地址或排程作業的系統。|
|類別 | 資料類型的分類。 對自動化來說，該值是 JobStreams。|
|JobId | Runbook 作業之識別碼的 GUID。|
|operationName | 指定在 Azure 中執行的作業類型。 對自動化來說，該值是 Job。|
|ResourceGroup | 指定 Runbook 作業的資源群組名稱。|
|resourceId | 指定 Azure 中的資源識別碼。 對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
|ResourceProvider | 指定 Azure 服務，以提供您可部署及管理的資源。 對自動化來說，此值是 Azure 自動化。|
|ResourceType | 指定 Azure 中的資源類型。 對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
|resultType | Runbook 作業在產生事件時的結果。 可能的值為：<br>- InProgres|
|resultDescription | 包含來自 Runbook 的輸出串流。|
|RunbookName | Runbook 的名稱。|
|SourceSystem | 指定所提交資料的來源系統。 對自動化來說，該值是 OpsManager。|
|StreamType | 作業串流的類型。 可能的值包括：<br>- Progress (進度)<br>- Output<br>- Warning (警告)<br>- Error (錯誤)<br>- Debug (偵錯)<br>- Verbose|
|Time | Runbook 作業的執行日期和時間。|

當您執行的記錄搜尋傳回 **JobLogs** 或 **JobStreams** 的類別記錄時，您可以選取 **JobLogs** 或 **JobStreams** 檢視，其中會顯示一組彙總搜尋所傳回更新的圖格。

## <a name="sample-log-searches"></a>記錄搜尋範例

下表提供此方案所收集之作業記錄的記錄檔搜尋範例。

|查詢 | 描述|
|----------|----------|
|尋找 ScheduledStartStop_Parent Runbook 已順利完成的作業 | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "ScheduledStartStop_Parent" ) <br>&#124;  where ( ResultType == "Completed" )  <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|尋找未成功完成的 runbook ScheduledStartStop_Parent的作業 | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "ScheduledStartStop_Parent" ) <br>&#124;  where ( ResultType == "Failed" )  <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|尋找 SequencedStartStop_Parent Runbook 已順利完成的作業 | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "SequencedStartStop_Parent" ) <br>&#124;  where ( ResultType == "Completed" ) <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|尋找未成功完成的 runbook SequencedStartStop_Parent的作業 | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "SequencedStartStop_Parent" ) <br>&#124;  where ( ResultType == "Failed" ) <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|

## <a name="next-steps"></a>後續步驟

在非工作時間啟動/停止 VM 解決方案不包括預定義的警報集。 使用 Azure 監視器查看[創建日誌警報](../azure-monitor/platform/alerts-log.md),瞭解如何建立作業失敗警報以支援 DevOps 或操作過程和過程。
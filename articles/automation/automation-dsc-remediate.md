---
title: 修復不符合的 Azure 自動化狀態設定伺服器
description: 如何依需設定重新套用於設定狀態漂移的伺服器
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.topic: conceptual
ms.date: 07/17/2019
manager: nirb
ms.openlocfilehash: f4ca76f4be9d00e185f8774fc33296d1af1aeece
ms.sourcegitcommit: 3c318f6c2a46e0d062a725d88cc8eb2d3fa2f96a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/02/2020
ms.locfileid: "80585507"
---
# <a name="remediate-non-compliant-dsc-servers"></a>補救不相容的 DSC 伺服器

當伺服器註冊到 Azure 自動化狀態配置時,「配置模式」設置為僅應用、應用和監視或應用和自動更正。
如果未將模式設置為"自動更正",則出於任何原因偏離相容狀態的伺服器將保持不合規狀態,直到手動更正它們。

Azure 計算提供了一個名為 Run 命令的功能,允許客戶在虛擬機內執行文本。
本文檔在手動更正配置漂移時提供了此功能的示例腳本。

## <a name="correct-drift-of-windows-virtual-machines-using-powershell"></a>使用 PowerShell 正確漂移 Windows 虛擬機器

有關使用 Windows 虛擬機器上的「執行命令」功能的分步說明,請參閱[Windows VM 中的文件頁「執行 PowerShell 文稿」,其中包含執行命令](/azure/virtual-machines/windows/run-command)。

要強制 Azure 自動化狀態配置節點下載並應用它,`Update-DscConfiguration`請使用 cmdlet。
有關詳細資訊,請參閱 cmdlet 文件[更新-Dsc 設定](/powershell/module/psdesiredstateconfiguration/update-dscconfiguration)。

```powershell
Update-DscConfiguration -Wait -Verbose
```

## <a name="correct-drift-of-linux-virtual-machines"></a>正確漂移 Linux 虛擬機器

類似的功能目前不適用於 Linux 伺服器。
唯一的選擇是重複註冊過程。
對於 Azure 節點,可以從門戶或使用 Az 自動化 cmdlet 進行漂移校正。
有關此過程的詳細資訊記錄在[Azure 自動化狀態配置進行管理的頁載入電腦](/azure/automation/automation-dsc-onboarding#onboard-a-vm-using-azure-portal)中。
對於混合節點,可以使用隨附的 Python 文稿完成漂移校正。
有關 Linux 回購,請參閱[PowerShell DSC 中](https://github.com/Microsoft/PowerShell-DSC-for-Linux#performing-dsc-operations-from-the-linux-computer)的文檔。

## <a name="next-steps"></a>後續步驟

- 如需 PowerShell Cmdlet 參考，請參閱 [Azure 自動化狀態設定 Cmdlet](/powershell/module/azurerm.automation/#automation)
- 若要查看在持續部署管線中使用 Azure 自動化狀態設定的範例，請參閱[使用 Azure 自動化狀態設定和 Chocolatey 的持續部署](automation-dsc-cd-chocolatey.md)

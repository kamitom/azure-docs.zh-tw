---
title: 茲抄工作顯示 |微軟文檔
description: 本文為 azcopy 作業顯示命令提供了參考資訊。
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 10/16/2019
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 7b5f566757dd77a61f252b123d0c9c1b74303fbe
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "74034127"
---
# <a name="azcopy-jobs-show"></a>azcopy 作業顯示

顯示給定作業 ID 的詳細資訊。

## <a name="synopsis"></a>概要

如果僅提供沒有標誌的作業 ID，則返回作業的進度摘要。

運行此命令時顯示的位元組計數和完成百分比僅反映作業中完成的檔。 它們不反映部分已完成的檔。

如果設置了`with-status`標誌，則將顯示具有給定值的作業中的傳輸清單。

```azcopy
azcopy jobs show [jobID] [flags]
```

## <a name="related-conceptual-articles"></a>相關概念文章

- [開始使用 AzCopy](storage-use-azcopy-v10.md)
- [使用 AzCopy 和 Blob 存儲傳輸資料](storage-use-azcopy-blobs.md)
- [使用 AzCopy 和檔案儲存體轉送資料](storage-use-azcopy-files.md)
- [對 AzCopy 進行設定、最佳化及疑難排解](storage-use-azcopy-configure.md)

## <a name="options"></a>選項。

|選項|描述|
|--|--|
|-h, --help|顯示顯示命令的説明內容。|
|--帶狀態字串|僅列出具有此狀態的作業的轉移，可用值：已啟動、成功、失敗|

## <a name="options-inherited-from-parent-commands"></a>從父命令繼承的選項

|選項|描述|
|---|---|
|--蓋-mbps uint32|以每秒百萬位元表示傳輸速率上限。 逐時輸送量可能與上限略有不同。 如果此選項設置為零，或者省略此選項，則輸送量不會上限。|
|--輸出類型字串|命令輸出的格式。 選項包括：文本，json。 預設值為"文本"。|

## <a name="see-also"></a>另請參閱

- [azcopy 作業](storage-ref-azcopy-jobs.md)

---
title: 如何替換磁帶基礎結構
description: 瞭解 Azure 備份如何提供類似磁帶的語義，使您能夠在 Azure 中備份和還原資料
ms.topic: conceptual
ms.date: 04/30/2017
ms.openlocfilehash: aeda1cefc84d425855c40b793f8334936541e63f
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/27/2020
ms.locfileid: "77425097"
---
# <a name="move-your-long-term-storage-from-tape-to-the-azure-cloud"></a>將您的長期儲存空間從磁帶移至 Azure 雲端

使用 Azure 備份和 System Center Data Protection Manager 的客戶可以進行下列操作：

* 依照最適合組織需求的排程備份資料。
* 將備份資料保留更長時間。
* 讓 Azure (而非磁帶) 成為長期保留需求的一部分。

此文章說明客戶如何啟用備份和保留原則。 使用磁帶處理長期保留需求的客戶，透過此功能的可用性現在擁有功能強大且可行的替代方案。 最新版的 Azure 備份已啟用此功能 (可從 [這裡](https://aka.ms/azurebackup_agent)取得)。 System Center DPM 客戶至少必須更新為 DPM 2012 R2 UR5，才能搭配使用 DPM 與 Azure 備份服務。

## <a name="what-is-the-backup-schedule"></a>什麼是備份排程？

備份排程會指出備份作業的頻率。 例如，下列畫面中的設定指出備份將於每日下午 6 點和午夜執行。

![每日排程](./media/backup-azure-backup-cloud-as-tape/dailybackupschedule.png)

客戶也可以排程每週備份。 例如，下列畫面中的設定指出備份將於每隔兩週的星期日和星期三上午 9:30 和 1:00 執行。

![每週排程](./media/backup-azure-backup-cloud-as-tape/weeklybackupschedule.png)

## <a name="what-is-the-retention-policy"></a>保留原則為何？

保留原則會指定備份必須儲存的期間。 除了僅針對所有備份點指定「一般原則」之外，客戶可以指定在進行備份時根據不同的保留原則。 例如，每日取得的備份點 (可充當作業復原點) 可保留 90 天。 在每一季結尾所取得的備份點 (供稽核之用) 可能需要保留較長的時間。

![保留原則](./media/backup-azure-backup-cloud-as-tape/retentionpolicy.png)

此原則中指定的「保留點」總數是 90 (每日保留點) + 40 (每季一點，連續 10 年) = 130。

## <a name="example--putting-both-together"></a>範例 – 將兩者綜合比較

![範例畫面](./media/backup-azure-backup-cloud-as-tape/samplescreen.png)

1. **每日的保留原則**：每日所進行的備份會儲存 7 天。
2. **每週保留政策**：週六午夜和下午 6 點的備份將保留四周。
3. **每月保留政策**：每月最後一個星期六午夜和下午 6 點的備份將保留 12 個月。
4. **年度保留政策**：每年 3 月最後一個星期六午夜進行的備份將保留 10 年。

上圖中計算「保留點」總數 (客戶可從該保留點還原資料) 的方式如下：

* 7 天中每天 2 個保留點 = 14 個復原點
* 4 週中每週 2 個保留點 = 8 個復原點
* 12 個月中每月 2 個保留點 = 24 個復原點
* 10 年中每年 1 個保留點 = 10 個復原點

因此復原點的總數是 56。

> [!NOTE]
> 使用 Azure 備份，每個受保護的實例最多可以創建 9999 個復原點。 受保護的執行個體係指會備份至 Azure 的電腦、伺服器 (實體或虛擬) 或工作負載。
>

## <a name="advanced-configuration"></a>進階組態

按一下上方畫面中的 [修改] **** ，客戶在指定保留排程時擁有更進一步的彈性。

![修改](./media/backup-azure-backup-cloud-as-tape/modify.png)

## <a name="next-steps"></a>後續步驟

如需有關 Azure Backup 的詳細資訊，請參閱：

* [Azure 備份的簡介](backup-introduction-to-azure-backup.md)
* [試用 Azure 備份](backup-try-azure-backup-in-10-mins.md)

---
title: 支援的版本 - MySQL 的 Azure 資料庫
description: 瞭解 MySQL 服務的 Azure 資料庫中支援哪些版本的 MySQL 伺服器。
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 197b3100190711a51cfe125fe1214a59c18e1491
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79536967"
---
# <a name="supported-azure-database-for-mysql-server-versions"></a>支援的適用於 MySQL 的 Azure 資料庫伺服器版本

適用於 MySQL 的 Azure 資料庫是使用 InnoDB 引擎，從 [MySQL Community Edition](https://www.mysql.com/products/community/) 開發的。

MySQL 會使用 X.Y.Z 命名配置。 X 是主要版本，Y 是次要版本，而 Z 是 Bug 修正版本。 如需此配置的詳細資訊，請參閱 [MySQL 文件](https://dev.mysql.com/doc/refman/5.7/en/which-version.html)。

> [!NOTE]
> 在服務中，會使用閘道將連線重新導向到伺服器執行個體。 建立連線之後，MySQL 用戶端會顯示閘道中設定的 MySQL 的版本，而非 MySQL 伺服器執行個體上執行的實際版本。 若要判斷您的 MySQL 伺服器執行個體的版本，請在 MySQL 提示字元命令下使用 `SELECT VERSION();` 命令。

適用於 MySQL 的 Azure 資料庫目前支援下列版本：

## <a name="mysql-version-56"></a>MySQL 5.6 版

錯誤修復版本： 5.6.45

請參閱 MySQL[版本資訊](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-45.html)，瞭解有關此版本中的改進和修復的詳細資訊。

## <a name="mysql-version-57"></a>MySQL 5.7 版

錯誤修復版本： 5.7.27

請參閱 MySQL[版本資訊](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-27.html)，瞭解有關此版本中的改進和修復的詳細資訊。

## <a name="mysql-version-80"></a>MySQL 版本 8.0

錯誤修復版本： 8.0.15

請參閱 MySQL[版本資訊](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-15.html)，瞭解有關此版本中的改進和修復的詳細資訊。

## <a name="managing-updates-and-upgrades"></a>管理更新和升級
服務會自動管理對於錯誤 (bug) 修正版本更新的修補。 例如 5.7.20 至 5.7.21。  

目前不支援主要和次要版本升級。 例如，不支援從 MySQL 5.6 升級至 MySQL 5.7。 如果您想要從 5.6 升級至 5.7，請將資料庫[備份和還原](./concepts-migrate-dump-restore.md)至使用新引擎版本所建立的伺服器。

## <a name="next-steps"></a>後續步驟

如需以您**服務層級**為依據之特定資源配額和限制的相關資訊，請參閱[服務層級](./concepts-pricing-tiers.md)

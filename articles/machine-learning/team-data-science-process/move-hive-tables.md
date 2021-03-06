---
title: 建立 Hive 資料表，並從 Blob 儲存體載入資料 - Team Data Science Process
description: 使用 Hive 查詢建立 Hive 資料表，並從 Azure Blob 儲存體載入資料。 資料分割 Hive 資料表，以及使用最佳化單欄式資料列 (ORC) 格式來提升查詢效能。
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 625d9d5c5ecf095d4acbff625754b2065f184536
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "79251657"
---
# <a name="create-hive-tables-and-load-data-from-azure-blob-storage"></a>建立 Hive 資料表，並從 Azure Blob 儲存體載入資料

本文會顯示泛型 Hive 查詢，這類查詢可建立 Hive 資料表，並從 Azure Blob 儲存體載入資料。 同時也會提供一些關於資料分割 Hive 資料表，以及使用最佳化單欄式資料列 (ORC) 格式來提升查詢效能的指引。

## <a name="prerequisites"></a>Prerequisites
本文假設您已經：

* 創建了 Azure 存儲帳戶。 如果需要說明，請參閱[有關 Azure 存儲帳戶](../../storage/common/storage-introduction.md)。
* 佈建含有 HDInsight 服務的自訂 Hadoop 叢集。  如需指示，請參閱[在 HDInsight 中設定叢集](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。
* 啟用叢集的遠端存取、登入，然後開啟 Hadoop 命令列主控台。 如需指示，請參閱[管理 Apache Hadoop 叢集](../../hdinsight/hdinsight-administer-use-portal-linux.md)。

## <a name="upload-data-to-azure-blob-storage"></a>將資料上傳至 Azure Blob 儲存體
如果您遵循[設定適用於進階分析的 Azure 虛擬機器](../../machine-learning/data-science-virtual-machine/overview.md)中所提供的指示建立了 Azure 虛擬機器，應該已將這個指令碼檔案下載至虛擬機器上的 *C:\\Users\\\<使用者名稱\>\\Documents\\Data Science Scripts* 目錄中。 這些 Hive 查詢僅要求您在適當的欄位中提供資料架構和 Azure Blob 存儲配置，以便隨時提交。

我們假設 Hive 資料表的資料為 **未壓縮的** 表格格式，而且資料已上傳至 Hadoop 叢集所使用之儲存體帳戶的預設 (或其他) 容器。

如果您想要使用 **NYC 計程車車程資料**進行練習，您需要︰

* **下載** 24 個 [NYC 計程車車程資料](https://www.andresmh.com/nyctaxitrips) 檔案 (12 個車程檔案和 12 個費用檔案)，
* **解壓縮** 為 .csv 檔案，然後
* **將它們上載**到 Azure 存儲帳戶的預設（或適當的容器）;將它們上載到 Azure 存儲帳戶的預設（或適當的容器）;此類帳戶的選項顯示在["使用 Azure 存儲與 Azure HDInsight 群集"主題](../../hdinsight/hdinsight-hadoop-use-blob-storage.md)中。 請參閱此 [頁面](hive-walkthrough.md#upload)，以了解將 .csv 檔案上傳至儲存體帳戶上之預設容器的程序。

## <a name="how-to-submit-hive-queries"></a><a name="submit"></a>如何提交蜂巢查詢
您可以使用下列方法來提交 Hive 查詢：

* [透過 Hadoop 叢集前端節點中的 Hadoop 命令列提交 Hive 查詢](#headnode)
* [利用 Hive 編輯器提交 Hive 查詢](#hive-editor)
* [利用 Azure PowerShell 命令提交 Hive 查詢](#ps)

Hive 查詢類似 SQL。 如果您熟悉 SQL，您可能會發現 [Hive for SQL 使用者功能提要](https://hortonworks.com/wp-content/uploads/2013/05/hql_cheat_sheet.pdf) 很有用。

提交 Hive 查詢時，您也可以控制 Hive 查詢輸出的目的地，它是否會出現在螢幕上，或是輸出到前端節點上的本機檔案或 Azure Blob。

### <a name="submit-hive-queries-through-hadoop-command-line-in-headnode-of-hadoop-cluster"></a><a name="headnode"></a>透過 Hadoop 叢集前端節點中的 Hadoop 命令列提交 Hive 查詢
如果 Hive 查詢相當複雜，在 Hadoop 叢集的前端節點中直接提交 Hive 查詢，通常會導致整備速度比使用 Hive 編輯器或 Azure PowerShell 指令碼進行提交還快。

登入 Hadoop 叢集的前端節點、在前端節點的桌面上開啟 Hadoop 命令列，然後輸入命令 `cd %hive_home%\bin`。

您有三種方式可在 Hadoop 命令列中提交 Hive 查詢：

* 直接
* 使用".hql"檔
* 利用 Hive 命令主控台

#### <a name="submit-hive-queries-directly-in-hadoop-command-line"></a>在 Hadoop 命令列中直接提交 Hive 查詢。
您可以執行類似 `hive -e "<your hive query>;` 的命令，在 Hadoop 命令列中直接提交簡單的 Hive 查詢。 在下列範例中，紅色方塊框起來的是提交 Hive 查詢的命令，而綠色方塊框起來的則是 Hive 查詢的輸出。

![使用 Hive 查詢輸出，提交 Hive 查詢的命令](./media/move-hive-tables/run-hive-queries-1.png)

#### <a name="submit-hive-queries-in-hql-files"></a>在".hql"檔中提交 Hive 查詢
若 Hive 查詢更複雜且有多行，則在命令列或 Hive 命令主控台中編輯查詢並不實際。 另一種方法是在 Hadoop 群集的頭節點中使用文字編輯器，將 Hive 查詢保存在頭節點的本地目錄中的".hql"檔中。 然後，可以使用如下`-f`參數提交".hql"檔中的 Hive 查詢：

    hive -f "<path to the '.hql' file>"

![".hql"檔中的 Hive 查詢](./media/move-hive-tables/run-hive-queries-3.png)

**隱藏 Hive 查詢的進度狀態畫面顯示**

根據預設，在 Hadoop 命令列中提交 Hive 查詢之後，Map/Reduce 作業的進度會顯示於螢幕上。 若要隱藏 Map/Reduce 工作進度的畫面顯示，您可以在命令列中使用引數 `-S` (大寫的 "S")，如下所示：

    hive -S -f "<path to the '.hql' file>"
    hive -S -e "<Hive queries>"

#### <a name="submit-hive-queries-in-hive-command-console"></a>在 Hive 命令主控台中提交 Hive 查詢。
您也可以在 Hadoop 命令列中執行 `hive` 命令，先進入 Hive 命令主控台，然後在 Hive 命令主控台中提交 Hive 查詢。 範例如下。 在此範例中，這兩個紅色方塊反白顯示的命令分別是用來進入 Hive 命令主控台，以及在 Hive 命令主控台中提交 Hive 查詢。 綠色方塊反白顯示的是 Hive 查詢的輸出。

![開啟 Hive 命令主控台並輸入命令，檢視 Hive 查詢輸出](./media/move-hive-tables/run-hive-queries-2.png)

上述範例會在螢幕上直接輸出 Hive 查詢結果。 您也可以將輸出寫入前端節點上的本機檔案，或寫入 Azure Blob。 接著，您可以使用其他工具，進一步分析 Hive 查詢的輸出。

**將 Hive 查詢結果輸出到本機檔案。**
若要將 Hive 查詢結果輸出到前端節點上的本機目錄，您必須在 Hadoop 命令列中提交 Hive 查詢，如下所示：

    hive -e "<hive query>" > <local path in the head node>

在下列範例中，Hive 查詢的輸出會寫入 `hivequeryoutput.txt` 目錄中的 `C:\apps\temp` 檔案。

![Hive 查詢的輸出](./media/move-hive-tables/output-hive-results-1.png)

**將 Hive 查詢結果輸出到 Azure Blob**

您也可以將 Hive 查詢結果輸出到 Hadoop 叢集預設容器內的 Azure Blob。 此作業的 Hive 查詢如下所示：

    insert overwrite directory wasb:///<directory within the default container> <select clause from ...>

在下列範例中，Hive 查詢的輸出會寫入 Hadoop 叢集預設容器內的 Blob 目錄 `queryoutputdir` 。 在此處，您只需提供目錄名稱，而不需提供 Blob 名稱。 如果您同時提供目錄和 Blob 名稱 (例如 `wasb:///queryoutputdir/queryoutput.txt`)，則會擲回錯誤。

![Hive 查詢的輸出](./media/move-hive-tables/output-hive-results-2.png)

如果您使用 Azure 儲存體總管開啟 Hadoop 叢集的預設容器，則可以看到如下圖所示的 Hive 查詢輸出。 您可以套用篩選條件 (以紅色方塊反白顯示)，只擷取名稱中具有指定字母的 Blob。

![Azure 儲存體總管顯示的 Hive 查詢輸出](./media/move-hive-tables/output-hive-results-3.png)

### <a name="submit-hive-queries-with-the-hive-editor"></a><a name="hive-editor"></a>利用 Hive 編輯器提交 Hive 查詢
您還可以使用查詢主控台（Hive 編輯器）在 Web 瀏覽器中輸入表單*HTTPs\//\<的 URL：hadoop 群集名稱>.azurehdinsight.net/Home/HiveEditor。* 您必須登入才能看到此主控台，因此您在這裡需要 Hadoop 叢集認證。

### <a name="submit-hive-queries-with-azure-powershell-commands"></a><a name="ps"></a>利用 Azure PowerShell 命令提交 Hive 查詢
您也可以使用 PowerShell 提交 Hive 查詢。 如需指示，請參閱 [使用 PowerShell 提交 Hive 工作](../../hdinsight/hadoop/apache-hadoop-use-hive-powershell.md)。

## <a name="create-hive-database-and-tables"></a><a name="create-tables"></a>創建 Hive 資料庫和表
Hive 查詢在[GitHub 存儲庫](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_db_tbls_load_data_generic.hql)中共用，並可從那裡下載。

以下是建立 Hive 資料表的 Hive 查詢。

    create database if not exists <database name>;
    CREATE EXTERNAL TABLE if not exists <database name>.<table name>
    (
        field1 string,
        field2 int,
        field3 float,
        field4 double,
        ...,
        fieldN string
    )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' lines terminated by '<line separator>'
    STORED AS TEXTFILE LOCATION '<storage location>' TBLPROPERTIES("skip.header.line.count"="1");

以下是您需要插入的欄位和其他設定的說明：

* **資料庫名稱\>：要創建的資料庫\<** 的名稱。 如果只想使用預設資料庫，則可以省略查詢"*創建資料庫..."。*
* **表名稱\>：要在指定資料庫中創建的表\<** 的名稱。 如果要使用預設資料庫，可以通過*\<表名\>* 直接引用該表，而無需\<資料庫名稱\>。
* **欄位分隔符號\>：分隔要上載到 Hive 表的資料檔案中的欄位的分隔符號。 \< **
* **行分隔符號\>：分隔資料檔案中的行的分隔符號。 \< **
* **存儲位置\>：用於保存 Hive 表資料的 Azure 存儲\<** 位置。 如果您未指定 *LOCATION\< 儲存體位置\>*，資料庫和資料表預設會儲存在 Hive 叢集之預設容器的 *hive/warehouse/* 目錄中。 如果您想要指定儲存體位置，儲存體位置必須位於資料庫和資料表的預設容器內。 此位置必須稱為與群集的預設容器相對的位置，其格式為 *"wasb:///\<目錄 1>/"* 或*\<"wasb:///目錄 1>/\<目錄 2>/'* 等。執行查詢後，將在預設容器中創建相對目錄。
* **TBLPROPERTIES("skip.header.line.count"="1")**：如果資料檔有標頭行，您就必須在 create table 查詢的**結尾**處新增這個屬性**。 否則，載入的標頭行會做為資料表的記錄。 如果資料檔不含標頭行，則可在查詢中省略此設定。

## <a name="load-data-to-hive-tables"></a><a name="load-data"></a>將資料載入至 Hive 資料表
以下是將資料載入 Hive 資料表的 Hive 查詢。

    LOAD DATA INPATH '<path to blob data>' INTO TABLE <database name>.<table name>;

* blob**資料的\>路徑：如果要上載到 Hive 表的 Blob 檔位於 HDInsight Hadoop 群集的預設容器中，則 blob 資料的路徑應採用"此容器中的wasb://目錄>/blob 檔案名>"。。 \< ** * \< \> * * \< \< * Blob 檔案也可以位於 HDInsight Hadoop 叢集的其他容器中。 在這種情況下*\<，blob\>資料的路徑*應採用 *"wasb://\<容器名稱>\<存儲帳戶名稱>.blob.core.net/\<blob 檔案名>"* 的格式。

  > [!NOTE]
  > 上傳至 Hive 資料表的 Blob 資料必須位於 Hadoop 叢集儲存體帳戶的預設或其他容器中。 否則，「LOAD DATA」 ** 查詢會失敗並提報它無法存取資料。
  >
  >

## <a name="advanced-topics-partitioned-table-and-store-hive-data-in-orc-format"></a><a name="partition-orc"></a>進階主題：資料分割資料表並以 ORC 格式儲存 Hive 資料
如果資料量很大，對於只需掃描資料表中數個資料分割的查詢而言，分割資料表就很有助益。 例如，依日期分割網站的記錄資料就很合理。

除了資料分割 Hive 資料表之外，對於使用最佳化單欄式資料列 (ORC) 格式來儲存 Hive 資料也很有幫助。 如需 ORC 格式的詳細資訊，請參閱<a href="https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-ORCFiles" target="_blank">在 Hive 讀取、寫入及處理資料時使用 ORC 檔案提升效能</a>。

### <a name="partitioned-table"></a>資料分割資料表
以下是建立資料分割資料表和並將資料載入其中的 Hive 查詢。

    CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<table name>
    (field1 string,
    ...
    fieldN string
    )
    PARTITIONED BY (<partitionfieldname> vartype) ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>'
         lines terminated by '<line separator>' TBLPROPERTIES("skip.header.line.count"="1");
    LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<partitioned table name>
        PARTITION (<partitionfieldname>=<partitionfieldvalue>);

查詢分區表時，建議在`where`子句**的開頭**添加分區條件，從而提高搜索效率。

    select
        field1, field2, ..., fieldN
    from <database name>.<partitioned table name>
    where <partitionfieldname>=<partitionfieldvalue> and ...;

### <a name="store-hive-data-in-orc-format"></a><a name="orc"></a>使用 ORC 格式儲存 Hive 資料
您無法將資料從 Blob 儲存體直接載入以 ORC 格式儲存的 Hive 資料表。 以下是您為了將資料從 Azure Blob 載入到以 ORC 格式儲存的 Hive 資料表所需採取的步驟。

建立外部資料表 **STORED AS TEXTFILE** ，並將資料從 Blob 儲存體載入該資料表。

        CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<external textfile table name>
        (
            field1 string,
            field2 int,
            ...
            fieldN date
        )
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>'
            lines terminated by '<line separator>' STORED AS TEXTFILE
            LOCATION 'wasb:///<directory in Azure blob>' TBLPROPERTIES("skip.header.line.count"="1");

        LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<table name>;

建立和步驟 1 中建立的外部資料表具備相同結構描述及相同欄位分隔符號的內部資料表，並使用 ORC 格式儲存 Hive 資料。

        CREATE TABLE IF NOT EXISTS <database name>.<ORC table name>
        (
            field1 string,
            field2 int,
            ...
            fieldN date
        )
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' STORED AS ORC;

從步驟 1 中的外部資料表選取資料，並插入 ORC 資料表

        INSERT OVERWRITE TABLE <database name>.<ORC table name>
            SELECT * FROM <database name>.<external textfile table name>;

> [!NOTE]
> 如果 TEXTFILE 表*\<資料庫\>名稱\<。外部文字檔表名稱\>* 具有分區，在 STEP 3`SELECT * FROM <database name>.<external textfile table name>`中，命令選擇分區變數作為返回的資料集中的欄位。 將其插入到*\<資料庫\>名稱 中。\<ORC 表\>名稱*由於*\<資料庫名稱\>而\<失敗。ORC 表\>名稱*在表架構中沒有分區變數作為欄位。 在這種情況下，您需要特別選擇要插入到*\<資料庫名稱 的\>欄位。\<ORC 表\>名稱*如下：
>
>

        INSERT OVERWRITE TABLE <database name>.<ORC table name> PARTITION (<partition variable>=<partition value>)
           SELECT field1, field2, ..., fieldN
           FROM <database name>.<external textfile table name>
           WHERE <partition variable>=<partition value>;

在所有資料都*\<\>* 插入到*\<資料庫名稱\>後，使用以下查詢時，可以安全地刪除外部文字檔表\<名。ORC 表\>名稱*：

        DROP TABLE IF EXISTS <database name>.<external textfile table name>;

依照此程序執行之後，您應該會有含 ORC 格式之資料的資料表可供使用。  

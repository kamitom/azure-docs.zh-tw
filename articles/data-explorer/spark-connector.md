---
title: 使用 Apache Spark 的 Azure 資料資源管理器連接器在 Azure 資料資源管理器和 Spark 群集之間移動資料。
description: 本主題演示如何在 Azure 資料資源管理器和 Apache Spark 群集之間移動資料。
author: orspod
ms.author: orspodek
ms.reviewer: michazag
ms.service: data-explorer
ms.topic: conceptual
ms.date: 1/14/2020
ms.openlocfilehash: b358287664ac6d6a3b641e1ab63073810ceb4c40
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "78208577"
---
# <a name="azure-data-explorer-connector-for-apache-spark"></a>用於 Apache Spark 的 Azure 資料資源管理器連接器

[Apache Spark](https://spark.apache.org/)是用於大規模資料處理的統一分析引擎。 Azure 資料資源管理器是一種快速、完全託管的資料分析服務，用於對大量資料進行即時分析。 

Spark 的 Azure 資料資源管理器連接器是可在任何 Spark 群集上運行的[開源專案](https://github.com/Azure/azure-kusto-spark)。 它實現資料來源和資料來源接收器，用於跨 Azure 資料資源管理器和 Spark 群集移動資料。 使用 Azure 資料資源管理器和 Apache Spark，可以構建針對資料驅動方案的快速且可擴展的應用程式。 例如，機器學習 （ML）、提取轉換載入 （ETL） 和日誌分析。 使用連接器，Azure 資料資源管理器將成為標準 Spark 源和接收器操作（如寫入、讀取和寫入流）的有效資料存儲。

您可以在批次處理或流模式下寫入 Azure 資料資源管理器。 從 Azure 資料資源管理器讀取支援列修剪和謂詞向下，這將篩選 Azure 資料資源管理器中的資料，從而減少傳輸資料的數量。

本主題介紹如何安裝和配置 Azure 資料資源管理器 Spark 連接器，以及如何在 Azure 資料資源管理器和 Apache Spark 群集之間移動資料。

> [!NOTE]
> 儘管以下一些示例引用 Azure[資料塊](https://docs.azuredatabricks.net/)Spark 群集，但 Azure 資料資源管理器 Spark 連接器不直接依賴于 Databricks 或任何其他 Spark 分佈。

## <a name="prerequisites"></a>Prerequisites

* [創建 Azure 資料資源管理器群集和資料庫](/azure/data-explorer/create-cluster-database-portal) 
* 建立 Spark 叢集
* 安裝 Azure 資料資源管理器連接器庫：
    * 為[Spark 2.4、Scala 2.11](https://github.com/Azure/azure-kusto-spark/releases)預先構建的庫 
    * [馬文回購](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/spark-kusto-connector)
* [Maven 3.x](https://maven.apache.org/download.cgi)已安裝

> [!TIP]
> 2.3.x 版本也受支援，但可能需要對 pom.xml 依賴項進行一些更改。

## <a name="how-to-build-the-spark-connector"></a>如何構建火花連接器

> [!NOTE]
> 此為選用步驟。 如果使用預構建的庫，則轉到 Spark[群集設置](#spark-cluster-setup)。

### <a name="build-prerequisites"></a>生成先決條件

1. 安裝[依賴項](https://github.com/Azure/azure-kusto-spark#dependencies)中列出的庫，包括以下[Kusto JAVA SDK](/azure/kusto/api/java/kusto-java-client-library)庫：
    * [庫斯托資料用戶端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-data)
    * [庫斯特引入用戶端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-ingest)

1. 有關構建火花連接器，請參閱[此源](https://github.com/Azure/azure-kusto-spark)。

1. 對於使用 Maven 專案定義的 Scala/JAVA 應用程式，使用以下專案連結應用程式（最新版本可能有所不同）：
    
    ```Maven
       <dependency>
         <groupId>com.microsoft.azure</groupId>
         <artifactId>spark-kusto-connector</artifactId>
         <version>1.1.0</version>
       </dependency>
    ```

### <a name="build-commands"></a>建置命令

若要建置 jar 並執行所有測試：

```
mvn clean package
```

要生成 jar，請運行所有測試，並將 jar 安裝到本地 Maven 存儲庫：

```
mvn clean install
```

有關詳細資訊，請參閱[連接器使用方式](https://github.com/Azure/azure-kusto-spark#usage)。

## <a name="spark-cluster-setup"></a>火花群集設置

> [!NOTE]
> 在執行以下步驟時，建議使用最新的 Azure 資料資源管理器 Spark 連接器版本。

1. 使用 Spark 2.4.4 和 Scala 2.11 基於 Azure 資料磚塊群集配置以下 Spark 群集設置：

    ![資料磚群集設置](media/spark-connector/databricks-cluster.png)
    
1. 安裝 Maven 的最新火花庫-庫托連接器庫：
    
    ![導入庫](media/spark-connector/db-libraries-view.png)![選擇火花-庫斯托連接器](media/spark-connector/db-dependencies.png)

1. 驗證是否安裝了所有必需的庫：

    ![驗證已安裝的庫](media/spark-connector/db-libraries-view.png)

1. 對於使用 JAR 檔進行安裝，請驗證是否安裝了其他依賴項：

    ![新增相依性](media/spark-connector/db-not-maven.png)

## <a name="authentication"></a>驗證

Azure 資料資源管理器 Spark 連接器使您能夠使用以下方法之一使用 Azure 活動目錄 （Azure AD） 進行身份驗證：
* [Azure AD 應用程式](#azure-ad-application-authentication)
* [Azure AD 訪問權杖](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#direct-authentication-with-access-token)
* [設備身份驗證](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#device-authentication)（用於非生產方案）
* [Azure 金鑰保存庫](https://github.com/Azure/azure-kusto-spark/blob/dev/docs/Authentication.md#key-vault)用於訪問金鑰保存庫資源、安裝 Azure 金鑰保管包並提供應用程式憑據。

### <a name="azure-ad-application-authentication"></a>Azure AD 應用程式身份驗證

Azure AD 應用程式身份驗證是最簡單、最常見的身份驗證方法，建議用於 Azure 資料資源管理器 Spark 連接器。

|屬性  |描述  |
|---------|---------|
|**KUSTO_AAD_CLIENT_ID**     |   Azure AD 應用程式（用戶端）識別碼。      |
|**KUSTO_AAD_AUTHORITY_ID**     |  Azure AD 身份驗證許可權。 Azure AD 目錄（租戶）ID。        |
|**KUSTO_AAD_CLIENT_PASSWORD**    |    用戶端的 Azure AD 應用程式金鑰。     |

### <a name="azure-data-explorer-privileges"></a>Azure 資料資源管理器許可權

授予 Azure 資料資源管理器群集上的以下許可權：

* 對於讀取（資料來源），Azure AD 標識必須在目標資料庫上具有*檢視器*許可權，或在目標表上具有*管理員*許可權。
* 對於寫入（資料接收器），Azure AD 標識必須在目標資料庫上具有*引入者*許可權。 它還必須在目標資料庫上具有*使用者*許可權才能創建新表。 如果目標表已存在，則必須在目標表上配置*管理員*許可權。
 
有關 Azure 資料資源管理器主體角色的詳細資訊，請參閱[基於角色的授權](/azure/kusto/management/access-control/role-based-authorization)。 有關管理安全形色，請參閱[安全形色管理](/azure/kusto/management/security-roles)。

## <a name="spark-sink-writing-to-azure-data-explorer"></a>火花接收器：寫入 Azure 資料資源管理器

1. 設置接收器參數：

     ```scala
    val KustoSparkTestAppId = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppId")
    val KustoSparkTestAppKey = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppKey")
 
    val appId = KustoSparkTestAppId
    val appKey = KustoSparkTestAppKey
    val authorityId = "72f988bf-86f1-41af-91ab-2d7cd011db47" // Optional - defaults to microsoft.com
    val cluster = "Sparktest.eastus2"
    val database = "TestDb"
    val table = "StringAndIntTable"
    ```

1. 將 Spark 資料框架寫入 Azure 資料資源管理器群集為批次處理：

    ```scala
    import com.microsoft.kusto.spark.datasink.KustoSinkOptions
    import org.apache.spark.sql.{SaveMode, SparkSession}

    df.write
      .format("com.microsoft.kusto.spark.datasource")
      .option(KustoSinkOptions.KUSTO_CLUSTER, cluster)
      .option(KustoSinkOptions.KUSTO_DATABASE, database)
      .option(KustoSinkOptions.KUSTO_TABLE, "Demo3_spark")
      .option(KustoSinkOptions.KUSTO_AAD_CLIENT_ID, appId)
      .option(KustoSinkOptions.KUSTO_AAD_CLIENT_PASSWORD, appKey)
      .option(KustoSinkOptions.KUSTO_AAD_AUTHORITY_ID, authorityId)
      .option(KustoSinkOptions.KUSTO_TABLE_CREATE_OPTIONS, "CreateIfNotExist")
      .mode(SaveMode.Append)
      .save()  
    ```
    
   或者使用簡化的語法：
   
    ```scala
         import com.microsoft.kusto.spark.datasink.SparkIngestionProperties
         import com.microsoft.kusto.spark.sql.extension.SparkExtension._
         
         val sparkIngestionProperties = Some(new SparkIngestionProperties()) // Optional, use None if not needed
         df.write.kusto(cluster, database, table, conf, sparkIngestionProperties)
    ```
   
1. 寫入流資料：

    ```scala    
    import org.apache.spark.sql.streaming.Trigger
    import java.util.concurrent.TimeUnit
    import java.util.concurrent.TimeUnit
    import org.apache.spark.sql.streaming.Trigger

    // Set up a checkpoint and disable codeGen. 
    spark.conf.set("spark.sql.streaming.checkpointLocation", "/FileStore/temp/checkpoint")
        
    // Write to a Kusto table from a streaming source
    val kustoQ = df
          .writeStream
          .format("com.microsoft.kusto.spark.datasink.KustoSinkProvider")
          .options(conf) 
          .option(KustoSinkOptions.KUSTO_WRITE_ENABLE_ASYNC, "true") // Optional, better for streaming, harder to handle errors
          .trigger(Trigger.ProcessingTime(TimeUnit.SECONDS.toMillis(10))) // Sync this with the ingestionBatching policy of the database
          .start()
    ```

## <a name="spark-source-reading-from-azure-data-explorer"></a>火花源：從 Azure 資料資源管理器讀取

1. 讀取[少量資料](/azure/kusto/concepts/querylimits)時，定義資料查詢：

    ```scala
    import com.microsoft.kusto.spark.datasource.KustoSourceOptions
    import org.apache.spark.SparkConf
    import org.apache.spark.sql._
    import com.microsoft.azure.kusto.data.ClientRequestProperties

    val query = s"$table | where (ColB % 1000 == 0) | distinct ColA"
    val conf: Map[String, String] = Map(
          KustoSourceOptions.KUSTO_AAD_CLIENT_ID -> appId,
          KustoSourceOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey
        )

    val df = spark.read.format("com.microsoft.kusto.spark.datasource").
      options(conf).
      option(KustoSourceOptions.KUSTO_QUERY, query).
      option(KustoSourceOptions.KUSTO_DATABASE, database).
      option(KustoSourceOptions.KUSTO_CLUSTER, cluster).
      load()

    // Simplified syntax flavor
    import com.microsoft.kusto.spark.sql.extension.SparkExtension._
    
    val cpr: Option[ClientRequestProperties] = None // Optional
    val df2 = spark.read.kusto(cluster, database, query, conf, cpr)
    display(df2)
    ```

1. 可選：**如果**提供臨時 Blob 存儲（而不是 Azure 資料資源管理器），則創建 Blob 由調用方負責。 這包括預配存儲、旋轉訪問金鑰和刪除瞬態專案。 
    KustoBlobStorageUtils 模組包含協助程式功能，用於根據帳戶和容器座標和帳戶憑據刪除 blob，或具有寫入、讀取和清單許可權的完整 SAS URL。 當不再需要相應的 RDD 時，每個事務都會在單獨的目錄中存儲瞬態 Blob 專案。 此目錄作為 Spark 驅動程式節點上報告的讀取事務資訊日誌的一部分捕獲。

    ```scala
    // Use either container/account-key/account name, or container SaS
    val container = dbutils.secrets.get(scope = "KustoDemos", key = "blobContainer")
    val storageAccountKey = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountKey")
    val storageAccountName = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountName")
    // val storageSas = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageSasUrl")
    ```

    在上面的示例中，不會使用連接器介面訪問金鑰保存庫;因此，不會使用連接器介面訪問金鑰保存庫。使用資料磚機密的更簡單的方法。

1. 從 Azure 資料資源管理器讀取。

    * **如果**提供瞬態 Blob 存儲，請從 Azure 資料資源管理器中讀取，如下所示：

        ```scala
         val conf3 = Map(
              KustoSourceOptions.KUSTO_AAD_CLIENT_ID -> appId,
              KustoSourceOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey
              KustoSourceOptions.KUSTO_BLOB_STORAGE_SAS_URL -> storageSas)
        val df2 = spark.read.kusto(cluster, database, "ReallyBigTable", conf3)
        
        val dfFiltered = df2
          .where(df2.col("ColA").startsWith("row-2"))
          .filter("ColB > 12")
          .filter("ColB <= 21")
          .select("ColA")
        
        display(dfFiltered)
        ```

    * 如果**Azure 資料資源管理器**提供瞬態 Blob 存儲，請從 Azure 資料資源管理器中讀取，如下所示：
    
        ```scala
        val dfFiltered = df2
          .where(df2.col("ColA").startsWith("row-2"))
          .filter("ColB > 12")
          .filter("ColB <= 21")
          .select("ColA")
        
        display(dfFiltered)
        ```

## <a name="next-steps"></a>後續步驟

* 瞭解有關 Azure[資料資源管理器火花連接器](https://github.com/Azure/azure-kusto-spark/tree/master/docs)的更多
* [JAVA 和 Python 的示例代碼](https://github.com/Azure/azure-kusto-spark/tree/master/samples/src/main)

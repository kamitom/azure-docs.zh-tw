---
title: 使用 Azure 媒體服務實作容錯移轉串流 | Microsoft Docs
description: 本文演示如何使用 Azure 媒體服務實現容錯移轉流方案。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: ae1371a8f025fd5e5722d483323fbe937538eb15
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "78939214"
---
# <a name="implement-failover-streaming-with-media-services-v2"></a>使用媒體服務 v2 實現容錯移轉流

本逐步解說示範如何將內容 (Blob) 從一個資產複製到另一個資產，以便處理隨選資料流處理的備援。 如果您想要設定 Azure 內容傳遞網路，以便在某個資料中心發生中斷時在兩個資料中心之間進行容錯移轉，這個案例會很有用。 本逐步解說使用 Azure 媒體服務 SDK、Azure 媒體服務 REST API 和 Azure 儲存體 SDK 來示範下列工作：

1. 在「資料中心 A」中設定媒體服務帳戶。
2. 將夾層檔上傳到來源資產。
3. 將資產編碼為多位元速率 MP4 檔案。 
4. 建立唯讀的共用存取簽章定位器。 這是為了讓來源資產能夠存取與來源資產相關聯之儲存體帳戶中的容器。
5. 從上一個步驟中建立的唯讀共用存取簽章定位器取得來源資產的容器名稱。 必須有此名稱，才能在儲存體帳戶之間複製 Blob (本主題稍後說明)。
6. 為編碼工作所建立的資產建立原始定位器。 

接著，若要處理容錯移轉：

1. 在「資料中心 B」中設定媒體服務帳戶。
2. 在目標媒體服務帳戶中建立目標空資產。
3. 建立寫入共用存取簽章定位器。 這是為了讓目標空白資產能夠寫入到與目標資產相關聯之目標儲存體帳戶中的容器。
4. 使用「Azure 儲存體 SDK」在「資料中心 A」中的來源儲存體帳戶與「資料中心 B」中的目標儲存體帳戶之間複製 Blob (資產檔案)。 這些儲存體帳戶會與相關資產關聯。
5. 讓複製到目標 Blob 容器的 Blob (資產檔案) 與目標資產產生關聯。 
6. 為「資料中心 B」中的資產建立原始定位器，並指定為「資料中心 A」中的資產所產生的定位器識別碼。

這可提供串流 URL，其中 URL 的相對路徑相同 (只有基底 URL 不同)。 

然後，若要處理任何中斷情形，您可以在這些原始定位器之上建立內容傳遞網路。 

您必須考量下列事項：

* 目前的媒體服務 SDK 版本不支援以程式設計方式產生可讓資產和資產檔案產生關聯的 IAssetFile 資訊。 請改為使用 CreateFileInfos 媒體服務 REST API 來進行此操作。 
* 儲存體加密資產 (AssetCreationOptions.StorageEncrypted) 不支援複寫 (因為兩個媒體服務帳戶中的加密金鑰不同)。 
* 如果您想要利用動態封裝，請確定您想要從中串流內容的串流端點是處於 [執行中]**** 狀態。

## <a name="prerequisites"></a>Prerequisites

* 在新的或現有的 Azure 訂用帳戶中有兩個媒體服務帳戶。 請參閱 [如何建立媒體服務帳戶](media-services-portal-create-account.md)。
* 作業系統：Windows 7、Windows 2008 R2 或 Windows 8。
* .NET Framework 4.5 或 .NET Framework 4。
* Visual Studio 2010 SP1 或更新版本 (Professional, Premium、Ultimate 或 Express)。

## <a name="set-up-your-project"></a>設定專案

在本節中，您會建立 C# Console Application 專案。

1. 使用 Visual Studio 建立一個包含 C# Console Application 專案的新方案。 輸入名稱的**控制碼冗余OnDemand流**，然後按一下 **"確定**"。
2. 在與 **"處理冗余ForDemand流.csproj**專案檔案"檔相同的級別上創建**支援檔**資料夾。 在 **"支援檔"** 資料夾下，創建 **"輸出檔案和** **MP4Files"** 資料夾。 將 .mp4 檔案複製到 **MP4Files** 資料夾  （在此示例中，使用**ignite.mp4**檔。 
3. 使用**NuGet**添加對與媒體服務相關的 DLL 的引用。 在**視覺工作室主功能表**中，選擇**TOOLS** > **NuGet 包管理器** > **管理器主控台**。 在主控台視窗中，鍵入**安裝包視窗azure.媒體服務**，然後按 Enter。
4. 添加此專案所需的其他引用：系統.運行時.序列化和系統.Web。
5. 使用預設情況下添加到**Programs.cs**檔中**的語句替換為**以下語句：

```csharp
using System;
using System.Globalization;
using System.IO;
using System.Net;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Xml;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.Auth;
using System.Runtime.Serialization.Json;
```

## <a name="add-code-that-handles-redundancy-for-on-demand-streaming"></a>新增可為隨選資料流處理備援的程式碼

在本節中，您會建立處理備援的能力。

1. 將下列類別層級欄位加入至 Program 類別。

    ```csharp
    private static readonly string storageNameTarget = "amsstorageacct2";
    private static readonly string storageKeyTarget = "00000000000000000000000000000000000000000000000000000000000000000000000000000000000000==";
    private static readonly string storageNameSource = "amsstorageacct1";
    private static readonly string storageKeySource = "00000000000000000000000000000000000000000000000000000000000000000000000000000000000000==";

    private static readonly string sourceApiServer = "https://amsacct1.restv2.westus2-2.media.azure.net/api/";
    private static readonly string targetApiServer = "https://amsacct2.restv2.eastus.media.azure.net/api/";
    private static string tokenSPSource = null;
    private static string tokenSPTarget = null;

    private static CloudMediaContext contextSource = null;
    private static CloudMediaContext contextTarget = null;

    // Base support files path.  Update this field to point to the base path  
    // for the local support files folder that you create. 
    private static readonly string SupportFiles = Path.GetFullPath(@"../..\SupportFiles");

    // Paths to support files (within the above base path). 
    private static readonly string SingleInputMp4Path = Path.GetFullPath(SupportFiles + @"\MP4Files\ignite.mp4");
    private static readonly string OutputFilesFolder = Path.GetFullPath(SupportFiles + @"\OutputFiles");
    ```
2. 以下列其中一項取代預設的 Main 方法定義。 從 Main 呼叫的方法定義會定義如下。
        
    ```csharp
    static void Main(string[] args)
    {
        AzureAdTokenCredentials tokenCredentialsForAcct1 =
         new AzureAdTokenCredentials("microsoft.onmicrosoft.com",
             new AzureAdClientSymmetricKey("00000000-0000-0000-0000-000000000000", "00000000000000000000000000000000"),
             AzureEnvironments.AzureCloudEnvironment);

        AzureAdTokenCredentials tokenCredentialsForAcct2 =
        new AzureAdTokenCredentials("microsoft.onmicrosoft.com",
             new AzureAdClientSymmetricKey("00000000-0000-0000-0000-000000000000", "00000000000000000000000000000000"),
             AzureEnvironments.AzureCloudEnvironment);

        var tokenProviderSource = new AzureAdTokenProvider(tokenCredentialsForAcct1);
        var tokenProviderTarget = new AzureAdTokenProvider(tokenCredentialsForAcct2);
        try
        {
        tokenSPSource = tokenProviderSource.GetAccessToken().Item1.ToString();
        tokenSPTarget = tokenProviderTarget.GetAccessToken().Item1.ToString();

        contextSource = new CloudMediaContext(new Uri(sourceApiServer), tokenProviderSource);
        contextTarget = new CloudMediaContext(new Uri(targetApiServer), tokenProviderTarget);


        IAsset assetSingleFile = CreateAssetAndUploadSingleFile(contextSource,
                         AssetCreationOptions.None,
                         SingleInputMp4Path);

         IJob job = CreateEncodingJob(contextSource, assetSingleFile);

        if (job.State != JobState.Error)
        {
            IAsset sourceOutputAsset = job.OutputMediaAssets[0];
            // Get the locator for Smooth Streaming
            var sourceOriginLocator = GetStreamingOriginLocator(contextSource, sourceOutputAsset);

            Console.WriteLine("Locator Id: {0}", sourceOriginLocator.Id);

            // 1.Create a read-only SAS locator for the source asset to have read access to the container in the source Storage account (associated with the source Media Services account)
            var readSasLocator = GetSasReadLocator(contextSource, sourceOutputAsset);

            // 2.Get the container name of the source asset from the read-only SAS locator created in the previous step
            string containerName = (new Uri(readSasLocator.Path)).Segments[1];

            // 3.Create a target empty asset in the target Media Services account
            var targetAsset = CreateTargetEmptyAsset(contextTarget, containerName);

            // 4.Create a write SAS locator for the target empty asset to have write access to the container in the target Storage account (associated with the target Media Services account)
            ILocator writeSasLocator = CreateSasWriteLocator(contextTarget, targetAsset);

            // Get asset container name.
            string targetContainerName = (new Uri(writeSasLocator.Path)).Segments[1];
            Console.WriteLine(targetContainerName);

            // 5.Copy the blobs in the source container (source asset) to the target container (target empty asset)
            CopyBlobsFromDifferentStorage(containerName, targetContainerName, storageNameSource, storageKeySource, storageNameTarget, storageKeyTarget);

            // 6.Use the CreateFileInfos Media Services REST API to automatically generate all the IAssetFile’s for the target asset. 
            //      This API call is not supported in the current Media Services SDK for .NET. 
            CreateFileInfos(targetApiServer, tokenSPTarget, targetAsset.Id);
            // Check if the AssetFiles are now  associated with the asset.
            Console.WriteLine("Asset files associated with the {0} asset:", targetAsset.Name);
            foreach (var af in targetAsset.AssetFiles)
            {
            Console.WriteLine(af.Name);
            }

            // 7.Copy the Origin locator of the source asset to the target asset by using the same Id
            var replicatedLocatorPath = CreateTargetOriginLocatorWithRest(contextSource, contextTarget, sourceOriginLocator.Id, targetAsset.Id);

            // Create a full URL to the manifest file. Use this for playback
            // in streaming media clients. 
            string originalUrlForClientStreaming = sourceOriginLocator.Path + GetPrimaryFile(sourceOutputAsset).Name + "/manifest";

            Console.WriteLine("Original Locator Path: {0}\n", originalUrlForClientStreaming);

            string replicatedUrlForClientStreaming = replicatedLocatorPath + GetPrimaryFile(sourceOutputAsset).Name + "/manifest";

            Console.WriteLine("Replicated Locator Path: {0}", replicatedUrlForClientStreaming);

            readSasLocator.Delete();
            writeSasLocator.Delete();
        }

        }
        catch (Exception e)
        {
        Console.WriteLine("Exception:" + e.ToString());
        }
    }
    ```
3. 從 Main 呼叫下列方法定義。 有關每種方法的更多詳細資訊，請參閱注釋。

    >[!NOTE]
    >對於不同的媒體服務原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您總是使用相同的天數和存取權限，您應該使用相同的原則識別碼。 例如，為預定要長時間維持就地 (非上傳原則) 的定位器原則，使用相同的識別碼。 有關詳細資訊，請參閱[本主題](media-services-dotnet-manage-entities.md#limit-access-policies)。

    ```csharp
    public static IAsset CreateAssetAndUploadSingleFile(CloudMediaContext context,
                         AssetCreationOptions assetCreationOptions,
                         string singleFilePath)
    {
        var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();

        var asset = context.Assets.Create(assetName, assetCreationOptions);

        Console.WriteLine("Asset name: " + asset.Name);

        var fileName = Path.GetFileName(singleFilePath);

        var assetFile = asset.AssetFiles.Create(fileName);

        Console.WriteLine("Created assetFile {0}", assetFile.Name);

        Console.WriteLine("Upload {0}", assetFile.Name);

        assetFile.Upload(singleFilePath);
        Console.WriteLine("Done uploading of {0}", assetFile.Name);

        return asset;
    }

    public static IJob CreateEncodingJob(CloudMediaContext context, IAsset asset)
    {
        // Declare a new job.
        IJob job = context.Jobs.Create("My encoding job");

        // Get a media processor reference, and pass to it the name of the 
        // processor to use for the specific task.
        IMediaProcessor processor = GetLatestMediaProcessorByName(context,
                            "Media Encoder Standard");

        // Create a task with the encoding details, using a string preset.
        // In this case "Adaptive Streaming" preset is used.
        ITask task = job.Tasks.AddNew("My encoding task",
        processor,
        "Adaptive Streaming",
        TaskOptions.ProtectedConfiguration);

        // Specify the input asset to be encoded.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job. 
        // This output is specified as AssetCreationOptions.None, which 
        // means the output asset is in the clear (unencrypted). 
        var outputAssetName = "OutputAsset_" + Guid.NewGuid();
        task.OutputAssets.AddNew(outputAssetName,
        AssetCreationOptions.None);

        // Use the following event handler to check job progress.  
        job.StateChanged += new
            EventHandler<JobStateChangedEventArgs>(StateChanged);

        // Launch the job.
        job.Submit();

        // Optionally log job details. This displays basic job details
        // to the console and saves them to a JobDetails-{JobId}.txt file 
        // in your output folder.
        LogJobDetails(context, job.Id);

        // Check job execution and wait for job to finish. 
        Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
        progressJobTask.Wait();

        // Get an updated job reference.
        job = context.Jobs.Where(j => j.Id == job.Id).FirstOrDefault();
        // Since we the output asset contains a set of Smooth Streaming files,
        // set the .ism file to be the primary file
        if (job.State != JobState.Error)
        SetPrimaryFile(job.OutputMediaAssets[0]);

        return job;
    }

    public static ILocator GetStreamingOriginLocator(CloudMediaContext context, IAsset assetToStream)
    {
        // Get a reference to the streaming manifest file from the  
        // collection of files in the asset. 
        IAssetFile manifestFile = GetPrimaryFile(assetToStream);

        // Create a 30-day readonly access policy. 
        // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            

        IAccessPolicy policy = context.AccessPolicies.Create("Streaming policy",
        TimeSpan.FromDays(30),
        AccessPermissions.Read);

        // Create a locator to the streaming content on an origin. 
        ILocator originLocator = context.Locators.CreateLocator(LocatorType.OnDemandOrigin,
        assetToStream,
        policy,
        DateTime.UtcNow.AddMinutes(-5));

        // Return the locator. 
        return originLocator;
    }

    public static ILocator GetSasReadLocator(CloudMediaContext context, IAsset asset)
    {
        IAccessPolicy accessPolicy = context.AccessPolicies.Create("File Download Policy",
        TimeSpan.FromDays(30), AccessPermissions.Read);

        ILocator sasLocator = context.Locators.CreateLocator(LocatorType.Sas,
        asset, accessPolicy);

        return sasLocator;
    }

    public static ILocator CreateSasWriteLocator(CloudMediaContext context, IAsset asset)
    {

        IAccessPolicy writePolicy = context.AccessPolicies.Create("Write Policy",
        TimeSpan.FromDays(30), AccessPermissions.Write);

        ILocator sasLocator = context.Locators.CreateLocator(LocatorType.Sas,
        asset, writePolicy);

        return sasLocator;
    }

    public static IAsset CreateTargetEmptyAsset(CloudMediaContext context, string containerName)
    {
        // Create a new asset.
        IAsset assetToBeProcessed = context.Assets.Create(containerName,
        AssetCreationOptions.None);

        return assetToBeProcessed;
    }

    public static string CreateTargetOriginLocatorWithRest(CloudMediaContext contextSource, CloudMediaContext contextTarget,  string locatorIdToReplicate, string targetAssetId)
    {
        string locatorNewPath = "";

        if (!string.IsNullOrEmpty(tokenSPTarget))
        {
        var asset = contextTarget.Assets.Where(a => a.Id == targetAssetId).FirstOrDefault();

        // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            
        var accessPolicy = contextTarget.AccessPolicies.Create("RestTest", TimeSpan.FromDays(100),
                                    AccessPermissions.Read);
        if (asset != null)
        {
            string redirectedServiceUri = null;

            var xmlResponse = CreateLocator(targetApiServer, out redirectedServiceUri, tokenSPTarget,
                                asset.Id, accessPolicy.Id,
                                (int)LocatorType.OnDemandOrigin,
                                DateTime.UtcNow.AddMinutes(-10), locatorIdToReplicate);

            Console.WriteLine("Redirected to: " + redirectedServiceUri);
            if (xmlResponse != null)
            {
            Console.WriteLine(String.Format("Locator Id: {0}",
                            xmlResponse.GetElementsByTagName("Id")[0].InnerText));
            Console.WriteLine(String.Format("Locator Path: {0}",
                xmlResponse.GetElementsByTagName("Path")[0].InnerText));

            locatorNewPath = xmlResponse.GetElementsByTagName("Path")[0].InnerText;
            }
        }
        }

        return locatorNewPath;
    }

    public static void SetPrimaryFile(IAsset asset)
    {
        var ismAssetFiles = asset.AssetFiles.ToList().
            Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase));

        if (ismAssetFiles.Count() != 1)
        throw new ArgumentException("The asset should have only one, .ism file");

        ismAssetFiles.First().IsPrimary = true;
        ismAssetFiles.First().Update();
    }

    public static IAssetFile GetPrimaryFile(IAsset asset)
    {
        // Cast the reference to a true IAssetFile type. 
        IAssetFile theManifest = asset.AssetFiles.ToList().
            Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).
            FirstOrDefault();

        return theManifest;
    }

    public static void CopyBlobsFromDifferentStorage(string sourceContainerName, string targetContainerName,
                        string srcAccountName, string srcAccountKey,
                        string destAccountName, string destAccountKey)
    {
        var srcAccount = new CloudStorageAccount(new StorageCredentials(srcAccountName, srcAccountKey), true);
        var destAccount = new CloudStorageAccount(new StorageCredentials(destAccountName, destAccountKey), true);

        var cloudBlobClient = srcAccount.CreateCloudBlobClient();
        var targetBlobClient = destAccount.CreateCloudBlobClient();

        var sourceContainer = cloudBlobClient.GetContainerReference(sourceContainerName);
        var targetContainer = targetBlobClient.GetContainerReference(targetContainerName);
        targetContainer.CreateIfNotExists();

        string blobToken = sourceContainer.GetSharedAccessSignature(new SharedAccessBlobPolicy()
        {
        // Specify the expiration time for the signature.
        SharedAccessExpiryTime = DateTime.Now.AddDays(1),
        // Specify the permissions granted by the signature.
        Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read
        });

        foreach (var sourceBlob in sourceContainer.ListBlobs())
        {
        string fileName = (sourceBlob as ICloudBlob).Name;
        var sourceCloudBlob = sourceContainer.GetBlockBlobReference(fileName);
        sourceCloudBlob.FetchAttributes();

        if (sourceCloudBlob.Properties.Length > 0)
        {
            // In Azure Media Services, the files are stored as block blobs. 
            // Page blobs are not supported by Azure Media Services.  
            var destinationBlob = targetContainer.GetBlockBlobReference(fileName);
            destinationBlob.StartCopy(new Uri(sourceBlob.Uri.AbsoluteUri + blobToken));

            while (true)
            {
            // The StartCopyFromBlob is an async operation, 
            // so we want to check if the copy operation is completed before proceeding. 
            // To do that, we call FetchAttributes on the blob and check the CopyStatus. 
            destinationBlob.FetchAttributes();
            if (destinationBlob.CopyState.Status != CopyStatus.Pending)
            {
                break;
            }
            //It's still not completed. So wait for some time.
            System.Threading.Thread.Sleep(1000);
            }
        }

        Console.WriteLine(fileName);
        }

        Console.WriteLine("Done copying.");
    }

    private static IMediaProcessor GetLatestMediaProcessorByName(CloudMediaContext context, string mediaProcessorName)
    {

        var processor = context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
        ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

        if (processor == null)
        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

        return processor;
    }

    // This method is a handler for events that track job progress.   
    private static void StateChanged(object sender, JobStateChangedEventArgs e)
    {
        Console.WriteLine("Job state changed event:");
        Console.WriteLine("  Previous state: " + e.PreviousState);
        Console.WriteLine("  Current state: " + e.CurrentState);

        switch (e.CurrentState)
        {
        case JobState.Finished:
            Console.WriteLine();
            Console.WriteLine("********************");
            Console.WriteLine("Job is finished.");
            Console.WriteLine("Please wait while local tasks or downloads complete...");
            Console.WriteLine("********************");
            Console.WriteLine();
            Console.WriteLine();
            break;
        case JobState.Canceling:
        case JobState.Queued:
        case JobState.Scheduled:
        case JobState.Processing:
            Console.WriteLine("Please wait...\n");
            break;
        case JobState.Canceled:
        case JobState.Error:
            // Cast sender as a job.
            IJob job = (IJob)sender;
            // Display or log error details as needed.
            LogJobStop(null, job.Id);
            break;
        default:
            break;
        }
    }

    private static void LogJobStop(CloudMediaContext context, string jobId)
    {
        StringBuilder builder = new StringBuilder();
        IJob job = context.Jobs.Where(j => j.Id == jobId).FirstOrDefault();

        builder.AppendLine("\nThe job stopped due to cancellation or an error.");
        builder.AppendLine("***************************");
        builder.AppendLine("Job ID: " + job.Id);
        builder.AppendLine("Job Name: " + job.Name);
        builder.AppendLine("Job State: " + job.State.ToString());
        builder.AppendLine("Job started (server UTC time): " + job.StartTime.ToString());
        // Log job errors if they exist.  
        if (job.State == JobState.Error)
        {
        builder.Append("Error Details: \n");
        foreach (ITask task in job.Tasks)
        {
            foreach (ErrorDetail detail in task.ErrorDetails)
            {
            builder.AppendLine("  Task Id: " + task.Id);
            builder.AppendLine("    Error Code: " + detail.Code);
            builder.AppendLine("    Error Message: " + detail.Message + "\n");
            }
        }
        }
        builder.AppendLine("***************************\n");
        // Write the output to a local file and to the console. The template 
        // for an error output file is:  JobStop-{JobId}.txt
        string outputFile = OutputFilesFolder + @"\JobStop-" + JobIdAsFileName(job.Id) + ".txt";
        WriteToFile(outputFile, builder.ToString());
        Console.Write(builder.ToString());
    }

    private static void LogJobDetails(CloudMediaContext context, string jobId)
    {
        StringBuilder builder = new StringBuilder();
        IJob job = context.Jobs.Where(j => j.Id == jobId).FirstOrDefault();

        builder.AppendLine("\nJob ID: " + job.Id);
        builder.AppendLine("Job Name: " + job.Name);
        builder.AppendLine("Job submitted (client UTC time): " + DateTime.UtcNow.ToString());

        // Write the output to a local file and to the console. The template 
        // for an error output file is:  JobDetails-{JobId}.txt
        string outputFile = OutputFilesFolder + @"\JobDetails-" + JobIdAsFileName(job.Id) + ".txt";
        WriteToFile(outputFile, builder.ToString());
        Console.Write(builder.ToString());
    }

    // Replace ":" with "_" in Job id values so they can 
    // be used as log file names.  
    private static string JobIdAsFileName(string jobID)
    {
        return jobID.Replace(":", "_");
    }

    // Write method output to the output files folder.
    private static void WriteToFile(string outFilePath, string fileContent)
    {
        StreamWriter sr = File.CreateText(outFilePath);
        sr.WriteLine(fileContent);
        sr.Close();
    }

    //////////////////////////////////////////////////////
    /// The following methods use REST calls.
    //////////////////////////////////////////////////////
    public static XmlDocument CreateLocator(string mediaServicesApiServerUri,
                        out string redirectedMediaServicesApiServerUri,
                        string bearerToken, string assetId,
                        string accessPolicyId, int locatorType,
                        DateTime startTime, string locatorIdToReplicate = null,
                        bool autoRedirect = true)
    {
        if (string.IsNullOrEmpty(mediaServicesApiServerUri))
        {
        mediaServicesApiServerUri = "https://media.windows.net/api/";
        }
        if (!mediaServicesApiServerUri.EndsWith("/"))
        mediaServicesApiServerUri = mediaServicesApiServerUri + "/";

        if (string.IsNullOrEmpty(bearerToken)) throw new ArgumentNullException("BearerToken");
        if (string.IsNullOrEmpty(assetId)) throw new ArgumentNullException("assetId");
        if (string.IsNullOrEmpty(accessPolicyId)) throw new ArgumentNullException("accessPolicyId");

        redirectedMediaServicesApiServerUri = null;
        XmlDocument xmlResponse = null;

        StringBuilder sb = new StringBuilder();
        sb.Append("{ \"AssetId\" : \"" + assetId + "\"");
        sb.Append(", \"AccessPolicyId\" : \"" + accessPolicyId + "\"");
        sb.Append(", \"Type\" : \"" + locatorType + "\"");
        if (startTime != DateTime.MinValue)
        sb.Append(", \"StartTime\" : \"" + startTime.ToString("G", CultureInfo.CreateSpecificCulture("en-us")) + "\"");
        if (!string.IsNullOrEmpty(locatorIdToReplicate))
        sb.Append(", \"Id\" : \"" + locatorIdToReplicate + "\"");
        sb.Append("}");

        string requestbody = sb.ToString();

        try
        {
        var request = GenerateRequest("POST", mediaServicesApiServerUri, "Locators",
            null, bearerToken, requestbody);
        var response = (HttpWebResponse)request.GetResponse();

        switch (response.StatusCode)
        {
            case HttpStatusCode.MovedPermanently:
            //Recurse once with the mediaServicesApiServerUri redirect Location:
            if (autoRedirect)
            {
                redirectedMediaServicesApiServerUri = response.Headers["Location"];
                string secondRedirection = null;
                xmlResponse = CreateLocator(redirectedMediaServicesApiServerUri,
                            out secondRedirection, bearerToken,
                            assetId, accessPolicyId, locatorType,
                            startTime, locatorIdToReplicate, false);
            }
            else
            {
                Console.WriteLine("Redirection to {0} failed.",
                mediaServicesApiServerUri);
                return null;
            }
            break;
            case HttpStatusCode.Created:
            using (Stream responseStream = response.GetResponseStream())
            {
                using (StreamReader stream = new StreamReader(responseStream))
                {
                string responseString = stream.ReadToEnd();
                var reader = JsonReaderWriterFactory.
                    CreateJsonReader(Encoding.UTF8.GetBytes(responseString),
                    new XmlDictionaryReaderQuotas());

                xmlResponse = new XmlDocument();
                reader.Read();
                xmlResponse.LoadXml(reader.ReadInnerXml());
                }
            }
            break;

            default:
            Console.WriteLine(response.StatusDescription);
            break;
        }
        }
        catch (WebException ex)
        {
        Console.WriteLine(ex.Message);
        }

        return xmlResponse;
    }

    public static void CreateFileInfos(string mediaServicesApiServerUri,
                    string bearerToken,
                    string assetId
                    )
    {
        if (!mediaServicesApiServerUri.EndsWith("/"))
        mediaServicesApiServerUri = mediaServicesApiServerUri + "/";

        if (String.IsNullOrEmpty(bearerToken)) throw new ArgumentNullException("bearerToken");
        if (String.IsNullOrEmpty(assetId)) throw new ArgumentNullException("assetId");

        try
        {
        var request = GenerateRequest("GET", mediaServicesApiServerUri, "CreateFileInfos",
            String.Format(CultureInfo.InvariantCulture, "assetid='{0}'", assetId), bearerToken, null);

        using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
        {
            if (response.StatusCode == HttpStatusCode.MovedPermanently)
            {
            string redirectedMediaServicesApiUrl = response.Headers["Location"];

            CreateFileInfos(redirectedMediaServicesApiUrl, bearerToken, assetId);
            }
            else if ((response.StatusCode != HttpStatusCode.OK) &&
            (response.StatusCode != HttpStatusCode.Accepted) &&
            (response.StatusCode != HttpStatusCode.Created) &&
            (response.StatusCode != HttpStatusCode.NoContent))
            {
            throw new Exception("Invalid response received ");
            }
        }
        }
        catch (WebException ex)
        {
        Console.WriteLine(ex.Message);
        }
    }

    private static HttpWebRequest GenerateRequest(string verb,
                            string mediaServicesApiServerUri,
                            string resourcePath, string query,
                            string bearerToken, string requestbody)
    {
        var uriBuilder = new UriBuilder(mediaServicesApiServerUri);
        uriBuilder.Path += resourcePath;
        if (query != null)
        {
        uriBuilder.Query = query;
        }

        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(uriBuilder.Uri);
        request.AllowAutoRedirect = false; //We manage our own redirects.
        request.Method = verb;

        if (resourcePath == "$metadata")
        request.MediaType = "application/xml";
        else
        {
        request.ContentType = "application/json;odata=verbose";
        request.Accept = "application/json;odata=verbose";
        }

        request.Headers.Add("DataServiceVersion", "3.0");
        request.Headers.Add("MaxDataServiceVersion", "3.0");
        request.Headers.Add("x-ms-version", "2.19");
        request.Headers.Add(HttpRequestHeader.Authorization, "Bearer " + bearerToken);

        if (requestbody != null)
        {
        var requestBytes = Encoding.ASCII.GetBytes(requestbody);
        request.ContentLength = requestBytes.Length;

        var requestStream = request.GetRequestStream();
        requestStream.Write(requestBytes, 0, requestBytes.Length);
        requestStream.Close();
        }
        else
        {
        request.ContentLength = 0;
        }
        return request;
    }
    ```
    
## <a name="content-protection"></a>內容保護

本主題中的示例顯示清除流。 如果要執行受保護的流式處理，您需要設置其他一些操作，您需要使用相同的**資產交付策略**、相同的**內容金鑰授權策略**或外部金鑰伺服器 URL，並且需要使用相同的識別碼複製內容金鑰。

有關內容保護的詳細資訊，請參閱使用[AES-128 動態加密和金鑰傳遞服務](media-services-protect-with-aes128.md)。

## <a name="see-also"></a>另請參閱

[使用 Azure Webhook 監視媒體服務作業通知](media-services-dotnet-check-job-progress-with-webhooks.md)

## <a name="next-steps"></a>後續步驟

您現在可以使用流量管理員在兩個資料中心之間路由傳送要求，因此在任何中斷的情況下容錯移轉。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]


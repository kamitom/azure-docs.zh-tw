---
title: 如何將模型部署到 Azure 庫伯奈斯服務
titleSuffix: Azure Machine Learning
description: 瞭解如何使用 Azure 庫伯奈斯服務將 Azure 機器學習模型部署為 Web 服務。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 01/16/2020
ms.openlocfilehash: aec1b7f7bf60be34d21d52ca652a776cf3275fe8
ms.sourcegitcommit: 98e79b359c4c6df2d8f9a47e0dbe93f3158be629
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/07/2020
ms.locfileid: "80811760"
---
# <a name="deploy-a-model-to-an-azure-kubernetes-service-cluster"></a>將模型部署到 Azure 庫伯奈斯服務群集
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

瞭解如何使用 Azure 機器學習在 Azure 庫伯奈斯服務 (AKS) 上將模型部署為 Web 服務。 Azure 庫伯奈斯服務適用於大規模生產部署。 如果需要以下一個或多個功能,請使用 Azure 庫伯奈斯服務:

- __快速回應時間__。
- __自動縮放__已部署的服務。
- __硬體加速__選項,如 GPU 和現場可程式設計柵極陣列 (FPGA)。

> [!IMPORTANT]
> 群集縮放不通過 Azure 機器學習 SDK 提供。 有關縮放 AKS 群集中的節點的詳細資訊,請參閱縮放[AKS 群組集中的節點計數](../aks/scale-cluster.md)。

部署到 Azure 庫伯奈斯服務時,將部署到__連接到工作區__的 AKS 群集。 有兩種方法可以將 AKS 群集連線到工作區:

* 使用 Azure 機器學習 SDK、機器學習 CLI 或[Azure 機器學習工作室](https://ml.azure.com)創建 AKS 群集。 此過程會自動將群集連接到工作區。
* 將現有的 AKS 群集附加到 Azure 機器學習工作區。 可以使用 Azure 機器學習 SDK、機器學習 CLI 或 Azure 機器學習工作室附加群集。

> [!IMPORTANT]
> 創建或附件過程是一次性任務。 將 AKS 群集連接到工作區後,可以使用它進行部署。 如果不再需要 AKS 群集,可以分離或刪除它。 分離或刪除後,您將無法再部署到群集。

## <a name="prerequisites"></a>Prerequisites

- Azure Machine Learning 工作區。 有關詳細資訊,請參閱創建[Azure 機器學習工作區](how-to-manage-workspace.md)。

- 在工作區中註冊的機器學習模型。 如果沒有註冊的模型,請參閱[如何部署模型以及部署模型的位置](how-to-deploy-and-where.md)。

- [機器學習服務的 Azure CLI](reference-azure-machine-learning-cli.md)擴充[、Azure 機器學習 Python SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)或 Azure[機器學習視覺化工作室代碼延伸](tutorial-setup-vscode-extension.md)。

- 本文中的__Python__代碼段假定設定了以下變數:

    * `ws`- 設置為工作區。
    * `model`- 設置為已註冊型號。
    * `inference_config`- 設置為模型的推理配置。

    有關設定這些變數的詳細資訊,請參閱[部署模型的方式和位置](how-to-deploy-and-where.md)。

- 本文中的__CLI__代碼段假定您`inferenceconfig.json`已建立 文檔。 有關建立此文件的詳細資訊,請參考[部署模型的方式和位置](how-to-deploy-and-where.md)。

## <a name="create-a-new-aks-cluster"></a>建立新的 AKS 叢集

**時間估計**:約20分鐘。

創建或附加 AKS 群集是工作區的一次性過程。 您可以重複使用此叢集進行多個部署。 如果刪除群集或包含群集的資源組,則必須在下次需要部署時創建新群集。 可以將多個 AKS 群集附加到工作區。

> [!TIP]
> 如果要使用 Azure 虛擬網路保護 AKS 群集,必須首先創建虛擬網路。 有關詳細資訊,請參閱[Azure 虛擬網路的安全實驗和推理](how-to-enable-virtual-network.md#aksvnet)。

如果要建立用於__開發__、__驗證__與__測試__的 AKS 群集,而不是生產,則可以指定__群組__來__開發測試__。

> [!WARNING]
> 如果設置`cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST`,則創建的群集不適合生產級別的流量,並且可能會增加推理時間。 開發/測試群集也不保證容錯。 我們建議至少為開發/測試群集使用 2 個虛擬 CPU。

以下範例展示如何使用 SDK 與 CLI 建立新的 AKS 叢集:

**使用 SDK**

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this).
# For example, to create a dev/test cluster, use:
# prov_config = AksCompute.provisioning_configuration(cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
prov_config = AksCompute.provisioning_configuration()

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

> [!IMPORTANT]
> 對於[`provisioning_configuration()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py),如果`agent_count`為和選擇`vm_size`的`cluster_purpose`自定義值`DEV_TEST`,則需要確保`agent_count`乘以`vm_size`大於等於等於 12 個虛擬 CPU。 例如,如果使用具有`vm_size`4 個虛擬 CPU 的「Standard_D3_v2」,則應選擇`agent_count`3 個或更多。
>
> Azure 機器學習 SDK 不提供支援,以擴展 AKS 群集。 要縮放群集中的節點,請使用 Azure 機器學習工作室中 AKS 群集的 UI。 您只能更改節點計數,而不能更改群集的 VM 大小。

有關此範例中使用的類、方法和參數的詳細資訊,請參閱以下參考文檔:

* [阿克斯計算.群集用途](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py)
* [阿克斯計算.provisioning_configuration](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [計算目標.建立](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#create-workspace--name--provisioning-configuration-)
* [計算目標.wait_for_completion](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#wait-for-completion-show-output-false-)

**使用 CLI**

```azurecli
az ml computetarget create aks -n myaks
```

有關詳細資訊,請參閱 az [ml 計算目標創建 aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-aks)引用。

## <a name="attach-an-existing-aks-cluster"></a>附加現有的 AKS 群組

**時間估計:** 約5分鐘。

如果 Azure 訂閱中已有 AKS 群集,並且版本 1.17 或更低,則可以使用它部署映射。

> [!TIP]
> 現有的 AKS 群集可以位於 Azure 機器學習工作區以外的 Azure 區域中。
>
> 如果要使用 Azure 虛擬網路保護 AKS 群集,必須首先創建虛擬網路。 有關詳細資訊,請參閱[Azure 虛擬網路的安全實驗和推理](how-to-enable-virtual-network.md#aksvnet)。

將 AKS 群集附加到工作區時,`cluster_purpose`可以通過設置 參數來定義如何使用群集。

如果不設置`cluster_purpose`參數或`cluster_purpose = AksCompute.ClusterPurpose.FAST_PROD`設置 ,則群集必須至少有 12 個可用的虛擬 CPU。

如果設置`cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST`,則群集不需要具有 12 個虛擬 CPU。 我們建議至少使用 2 個虛擬 CPU 進行開發/測試。 但是,為開發/測試配置的群集不適合生產級別的流量,可能會增加推理時間。 開發/測試群集也不保證容錯。

> [!WARNING]
> 請勿從工作區創建多個同時到同一 AKS 群集的附件。 例如,使用兩個不同的名稱將一個 AKS 群集附加到工作區。 每個新附件將斷開以前的現有附件。
>
> 如果要重新附加 AKS 群集(例如更改 TLS 或其他群集配置設置),必須先使用[AksCompute.detach()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#detach--)刪除現有附件。

有關使用 Azure CLI 或門戶建立 AKS 群集的詳細資訊,請參閱以下文章:

* [建立 AKS 叢集 (CLI)](https://docs.microsoft.com/cli/azure/aks?toc=%2Fazure%2Faks%2FTOC.json&bc=%2Fazure%2Fbread%2Ftoc.json&view=azure-cli-latest#az-aks-create)
* [建立 AKS 叢集(門戶)](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal?view=azure-cli-latest)

以下範例展示如何將現有的 AKS 叢集附加到工作區:

**使用 SDK**

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Set the resource group that contains the AKS cluster and the cluster name
resource_group = 'myresourcegroup'
cluster_name = 'myexistingcluster'

# Attach the cluster to your workgroup. If the cluster has less than 12 virtual CPUs, use the following instead:
# attach_config = AksCompute.attach_configuration(resource_group = resource_group,
#                                         cluster_name = cluster_name,
#                                         cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
aks_target = ComputeTarget.attach(ws, 'myaks', attach_config)
```

有關此範例中使用的類、方法和參數的詳細資訊,請參閱以下參考文檔:

* [阿克斯計算.attach_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [阿克斯計算.群集用途](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?view=azure-ml-py)
* [阿克斯計算.附加](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computetarget?view=azure-ml-py#attach-workspace--name--attach-configuration-)

**使用 CLI**

要使用 CLI 連接現有群集,需要獲取現有群集的資源 ID。 要獲取此值,請使用以下命令。 替換為`myexistingcluster`AKS 群集的名稱。 取代為`myresourcegroup`包含群組的資源群組:

```azurecli
az aks show -n myexistingcluster -g myresourcegroup --query id
```

此命令會傳回類似下列文字的值：

```text
/subscriptions/{GUID}/resourcegroups/{myresourcegroup}/providers/Microsoft.ContainerService/managedClusters/{myexistingcluster}
```

要將現有群集附加到工作區,請使用以下命令。 替換為`aksresourceid`前一個命令返回的值。 替換為`myresourcegroup`包含工作區的資源組。 替換為`myworkspace`工作區名稱。

```azurecli
az ml computetarget attach aks -n myaks -i aksresourceid -g myresourcegroup -w myworkspace
```

有關詳細資訊,請參閱 az [ml 計算目標附加 aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-attach-aks)引用。

## <a name="deploy-to-aks"></a>部署到 AKS

要將模型部署到 Azure 函式庫─伯奈斯服務,請建立描述所需計算資源的__部署設定__。 例如,內核和記憶體的數量。 您還需要一個__推理配置__,它描述了託管模型和 Web 服務所需的環境。 有關建立推理設定的詳細資訊,請參閱[部署模型的方式和位置](how-to-deploy-and-where.md)。

### <a name="using-the-sdk"></a>使用 SDK

```python
from azureml.core.webservice import AksWebservice, Webservice
from azureml.core.model import Model

aks_target = AksCompute(ws,"myaks")
# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.
deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config, aks_target)
service.wait_for_deployment(show_output = True)
print(service.state)
print(service.get_logs())
```

有關此範例中使用的類、方法和參數的詳細資訊,請參閱以下參考文檔:

* [阿克斯計算](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.akscompute?view=azure-ml-py)
* [阿克斯Web服務.deploy_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?view=azure-ml-py)
* [模型.部署](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Web 服務.wait_for_deployment](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py#wait-for-deployment-show-output-false-)

### <a name="using-the-cli"></a>使用 CLI

要使用 CLI 進行部署,請使用以下命令。 替換為`myaks`AKS 計算目標的名稱。 替換為`mymodel:1`已註冊模型的名稱和版本。 取代為`myservice`名稱以提供此服務:

```azurecli-interactive
az ml model deploy -ct myaks -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

有關詳細資訊,請參閱 az [ml 模型部署](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy)引用。

### <a name="using-vs-code"></a>使用 VS Code

有關使用 VS 代碼的資訊,請參閱[通過 VS 代碼擴展部署到 AKS。](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model)

> [!IMPORTANT]
> 通過 VS 代碼進行部署需要提前創建 AKS 群集或連接到工作區。

## <a name="deploy-models-to-aks-using-controlled-rollout-preview"></a>使用受控部署(預覽)將模型部署到 AKS

使用端點以受控方式分析和推廣模型版本。 您可以在單個終結點後面部署最多六個版本。 終結點提供以下功能:

* 設定__設定到每個終結點的評分流量的百分比__。 例如,將 20% 的流量路由到端點「測試」,將 80% 的流量路由到「生產」。

    > [!NOTE]
    > 如果不考慮流量的 100%,則任何剩餘百分比將路由到__默認__終結點版本。 例如,如果將終結點版本「測試」設定為獲取 10% 的流量,將「prod」設定為 30%,則剩餘的 60% 將發送到預設終結點版本。
    >
    > 建立的第一個終結點版本將自動配置為預設值。 您可以在創建或更新終結點版本`is_default=True`時設置此設定來更改此設置。
     
* 將終結點版本標記為__控制項__或__處理__。 例如,當前生產終結點版本可能是控件,而潛在的新模型將作為處理版本部署。 評估處理版本的性能後,如果性能優於當前控制,則可能會將其提升為新的生產/控制。

    > [!NOTE]
    > 您只能有__一個__控制項。 你可以進行多種治療。

您可以啟用應用見解以查看終結點和已部署版本的操作指標。

### <a name="create-an-endpoint"></a>建立端點
準備好部署模型後,創建評分終結點並部署第一個版本。 下面的範例展示如何使用 SDK 部署和創建終結點。 第一個部署將定義為預設版本,這意味著所有版本中未指定的流量百分位數將轉到預設版本。  

> [!TIP]
> 在下面的範例中,配置將初始終結點版本設置以處理 20% 的流量。 由於這是第一個終結點,因此也是預設版本。 由於我們沒有其他版本的其他 80% 的流量,因此它也路由到預設值。 在部署佔用一定流量百分比的其他版本之前,此版本實際上接收100%的流量。

```python
import azureml.core,
from azureml.core.webservice import AksEndpoint
from azureml.core.compute import AksCompute
from azureml.core.compute import ComputeTarget
# select a created compute
compute = ComputeTarget(ws, 'myaks')
namespace_name= endpointnamespace
# define the endpoint and version name
endpoint_name = "mynewendpoint"
version_name= "versiona"
# create the deployment config and define the scoring traffic percentile for the first deployment
endpoint_deployment_config = AksEndpoint.deploy_configuration(cpu_cores = 0.1, memory_gb = 0.2,
                                                              enable_app_insights = True,
                                                              tags = {'sckitlearn':'demo'},
                                                              description = "testing versions",
                                                              version_name = version_name,
                                                              traffic_percentile = 20)
 # deploy the model and endpoint
 endpoint = Model.deploy(ws, endpoint_name, [model], inference_config, endpoint_deployment_config, compute)
 # Wait for he process to complete
 endpoint.wait_for_deployment(True)
 ```

### <a name="update-and-add-versions-to-an-endpoint"></a>更新版本並將新增到終結點

向終結點添加另一個版本,並配置要向該版本添加的評分流量百分位數。 有兩種類型的版本,一種控制項和一個處理版本。 可以有多個處理版本來幫助與單個控制版本進行比較。

> [!TIP]
> 第二個版本由以下代碼段創建,接受 10% 的流量。 第一個版本配置為 20%,因此只有 30% 的流量配置為特定版本。 剩餘的 70% 將發送到第一個終結點版本,因為它也是預設版本。

 ```python
from azureml.core.webservice import AksEndpoint

# add another model deployment to the same endpoint as above
version_name_add = "versionb"
endpoint.create_version(version_name = version_name_add,
                        inference_config=inference_config,
                        models=[model],
                        tags = {'modelVersion':'b'},
                        description = "my second version",
                        traffic_percentile = 10)
endpoint.wait_for_deployment(True)
```

更新現有版本或在終結點中刪除它們。 您可以更改版本的預設類型、控制項類型和流量百分位數。 在下面的示例中,第二個版本的流量增加到 40%,現在默認為預設版本。

> [!TIP]
> 在以下代碼段之後,第二個版本現在預設為預設版本。 現在配置為 40%,而原始版本仍配置為 20%。 這意味著 40% 的流量不由版本配置來考慮。 剩餘流量將路由到第二個版本,因為它現在預設。 它有效地接收了 80% 的流量。

 ```python
from azureml.core.webservice import AksEndpoint

# update the version's scoring traffic percentage and if it is a default or control type
endpoint.update_version(version_name=endpoint.versions["versionb"].name,
                        description="my second version update",
                        traffic_percentile=40,
                        is_default=True,
                        is_control_version_type=True)
# Wait for the process to complete before deleting
endpoint.wait_for_deployment(true)
# delete a version in an endpoint
endpoint.delete_version(version_name="versionb")

```


## <a name="web-service-authentication"></a>Web 服務驗證

部署到 Azure 庫伯奈斯服務時,默認情況下將啟用__基於密鑰__的身份驗證。 您還可以啟用__基於權杖的__認證認證。 基於權杖的身份驗證要求用戶端使用 Azure 活動目錄帳戶請求身份驗證權杖,該權杖用於向已部署的服務發出請求。

要__關閉__身份驗證,請設置`auth_enabled=False`參數,在創建部署配置時。 以下範例使用 SDK 關閉身份驗證:

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, auth_enabled=False)
```

有關從用戶端應用程式進行身份驗證的資訊,請參閱[使用為 Web 服務部署的 Azure 機器學習模型](how-to-consume-web-service.md)。

### <a name="authentication-with-keys"></a>使用金鑰進行認證

開啟金鑰認證,可以使用方法`get_keys`的認證金鑰與輔助身份認證金鑰:

```python
primary, secondary = service.get_keys()
print(primary)
```

> [!IMPORTANT]
> 如果需要重新產生金鑰,請使用[`service.regen_key`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py)

### <a name="authentication-with-tokens"></a>使用權杖進行認證

要啟用權杖身份驗證,`token_auth_enabled=True`在建立或更新部署時設定參數。 以下範例使用 SDK 啟用權杖身份驗證:

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, token_auth_enabled=True)
```

如果啟用了權杖身份驗證,`get_token`則可以使用 方法檢索 JWT 權杖和該權杖的過期時間:

```python
token, refresh_by = service.get_token()
print(token)
```

> [!IMPORTANT]
> 您需要在權杖`refresh_by`之後請求新權杖。
>
> Microsoft 強烈建議您在同一區域創建 Azure 機器學習工作區,與 Azure Kubernetes 服務群集相同。 要使用權杖進行身份驗證,Web 服務將調用創建 Azure 機器學習工作區的區域。 如果工作區的區域不可用,則即使群集與工作區位於不同的區域,也將無法為 Web 服務獲取權杖。 這有效地導致基於令牌的身份驗證不可用,直到工作區的區域再次可用。 此外,群集的區域和工作區區域之間的距離越大,獲取令牌所花的時間就越長。

## <a name="update-the-web-service"></a>更新 Web 服務

[!INCLUDE [aml-update-web-service](../../includes/machine-learning-update-web-service.md)]

## <a name="next-steps"></a>後續步驟

* [在虛擬網路中進行安全實驗和推理](how-to-enable-virtual-network.md)
* [如何使用自訂 Docker 映像部署模型](how-to-deploy-custom-docker-image.md)
* [部署故障排除](how-to-troubleshoot-deployment.md)
* [使用 TLS 透過 Azure 機器學習保護 Web 服務](how-to-secure-web-service.md)
* [取用部署為 Web 服務的 ML 模型](how-to-consume-web-service.md)
* [使用應用程式見解監視 Azure 機器學習模型](how-to-enable-app-insights.md)
* [在生產環境中收集模型資料](how-to-enable-data-collection.md)

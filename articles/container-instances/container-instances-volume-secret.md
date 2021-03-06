---
title: 將機密卷載入容器群組
description: 了解如何掛接秘密磁碟區，以儲存供您的容器執行個體存取的機密資訊
ms.topic: article
ms.date: 04/03/2020
ms.openlocfilehash: 756828e71174246450245938595c8872afc62961
ms.sourcegitcommit: 62c5557ff3b2247dafc8bb482256fef58ab41c17
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2020
ms.locfileid: "80657158"
---
# <a name="mount-a-secret-volume-in-azure-container-instances"></a>在 Azure 容器執行個體中掛接秘密磁碟區

使用「秘密」** 磁碟區，將敏感性資訊提供給容器群組中的容器。 「秘密」** 磁碟區會將您的祕密儲存在磁碟區內的檔案中，容器群組中的容器即可存取這些檔案。 藉由在「秘密」** 磁碟區中儲存密碼，您可以避免將 SSH 金鑰或資料庫認證等敏感性資料新增至您的應用程式程式碼。

* 在容器群組中使用機密部署後,機密卷是*唯讀的*。
* 所有秘密磁碟區都由 RAM 型檔案系統 [tmpfs][tmpfs] 支援；其內容永遠不會寫入靜態儲存區。

> [!NOTE]
> 「祕密」** 磁碟需目前僅限於 Linux 容器。 了解如何在[設定環境變數](container-instances-environment-variables.md)中，為 Windows 和 Linux 容器傳遞安全的環境變數。 當我們正在努力將所有功能帶到 Windows 容器時,您可以在[概述](container-instances-overview.md#linux-and-windows-containers)中找到當前平台差異。

## <a name="mount-secret-volume---azure-cli"></a>掛接秘密磁碟區 - Azure CLI

若要使用 Azure CLI 部署具有一或多個祕密的容器，請在 [az container create][az-container-create] 命令中包和 `--secrets` 和 `--secrets-mount-path` 參數。 這個範例在 中載入包含兩個包含機密的檔案("mysecret1"和"mysecret2")*的機密*卷`/mnt/secrets`, 在 :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name secret-volume-demo \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --secrets mysecret1="My first secret FOO" mysecret2="My second secret BAR" \
    --secrets-mount-path /mnt/secrets
```

下列 [az container exec][az-container-exec] 輸出顯示如何在執行中的容器中開啟殼層、列出祕密磁碟區內的檔案，然後顯示其內容：

```azurecli
az container exec \
  --resource-group myResourceGroup \
  --name secret-volume-demo --exec-command "/bin/sh"
```

```output
/usr/src/app # ls /mnt/secrets
mysecret1
mysecret2
/usr/src/app # cat /mnt/secrets/mysecret1
My first secret FOO
/usr/src/app # cat /mnt/secrets/mysecret2
My second secret BAR
/usr/src/app # exit
Bye.
```

## <a name="mount-secret-volume---yaml"></a>掛接秘密磁碟區 - YAML

您也可以使用 Azure CLI 和 [YAML 範本](container-instances-multi-container-yaml.md)部署容器群組。 部署由多個容器所組成的容器群組時，偏好經由 YAML 範本進行部署。

當您透過 YAML 範本進行部署時，範本中的祕密值必須為 **Base64 編碼**。 不過，祕密值會出現於容器中檔案內的純文字。

下列 YAML 範本可定義含有一個容器的容器群組，而該容器會在 `/mnt/secrets` 掛接「祕密」** 磁碟區。 機密卷有兩個包含機密的檔,"我的機密1"和"我的機密2"。

```yaml
apiVersion: '2018-10-01'
location: eastus
name: secret-volume-demo
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /mnt/secrets
        name: secretvolume1
  osType: Linux
  restartPolicy: Always
  volumes:
  - name: secretvolume1
    secret:
      mysecret1: TXkgZmlyc3Qgc2VjcmV0IEZPTwo=
      mysecret2: TXkgc2Vjb25kIHNlY3JldCBCQVIK
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

若要透過 YAML 範本進行部署，請將上述 YAML 儲存到名為 `deploy-aci.yaml` 的檔案，然後使用  參數執行 [az container create`--file`][az-container-create] 命令：

```azurecli-interactive
# Deploy with YAML template
az container create \
  --resource-group myResourceGroup \
  --file deploy-aci.yaml
```

## <a name="mount-secret-volume---resource-manager"></a>掛接秘密磁碟區 - Resource Manager

除了 CLI 和 YAML 部署，您可以使用 Azure [Resource Manager 範例](/azure/templates/microsoft.containerinstance/containergroups)來部署容器群組。

首先，填入範本的容器群組 `properties` 區段中的 `volumes` 陣列。 當您透過 Resource Manager 範本進行部署時，範本中的祕密值必須為 **Base64 編碼**。 不過，祕密值會出現於容器中檔案內的純文字。

接下來，針對您想要掛接祕密** 磁碟區所在容器群組中的每個容器，填入容器定義之 `properties` 區段中的 `volumeMounts` 陣列。

下列 Resource Manager 範本可定義含有一個容器的容器群組，而該容器會在 `/mnt/secrets` 掛接「祕密」** 磁碟區。 此祕密磁碟區有兩個祕密 "mysecret1" 和 "mysecret2"。

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-secret.json -->
[!code-json[volume-secret](~/azure-docs-json-samples/container-instances/aci-deploy-volume-secret.json)]

要使用資源管理員樣本進行部署,請將前面的 JSON 儲存到名為的`deploy-aci.json`檔案 ,`--template-file`然後使用 參數執行[az 部署群組建立][az-deployment-group-create]指令:

```azurecli-interactive
# Deploy with Resource Manager template
az deployment group create \
  --resource-group myResourceGroup \
  --template-file deploy-aci.json
```

## <a name="next-steps"></a>後續步驟

### <a name="volumes"></a>磁碟區

了解如何在 Azure 容器執行個體中掛接其他類型的磁碟區：

* [在 Azure 容器執行個體中掛接 Azure 檔案共用](container-instances-volume-azure-files.md)
* [在 Azure 容器執行個體中掛接 emptyDir 磁碟區](container-instances-volume-emptydir.md)
* [在 Azure 容器執行個體中掛接 gitRepo 磁碟區](container-instances-volume-gitrepo.md)

### <a name="secure-environment-variables"></a>安全的環境變數

將敏感性資訊提供給容器 (包括 Windows 容器) 的另一種方法是使用[安全的環境變數](container-instances-environment-variables.md#secure-values)。

<!-- LINKS - External -->
[tmpfs]: https://wikipedia.org/wiki/Tmpfs

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create

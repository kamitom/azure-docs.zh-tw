---
title: 從 Azure 容器註冊表部署容器映射
description: 瞭解如何通過從 Azure 容器註冊表中拉出容器映射在 Azure 容器實例中部署容器。
services: container-instances
ms.topic: article
ms.date: 02/18/2020
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 50c209483a12adc3545b63fb66685e386d9ad10a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "78252146"
---
# <a name="deploy-to-azure-container-instances-from-azure-container-registry"></a>從 Azure Container Registry 部署至 Azure 容器執行個體

[Azure Container Registry](../container-registry/container-registry-intro.md) 是 Azure 型的受控容器登錄服務，可用來儲存私人 Docker 容器映像。 本文介紹在部署到 Azure 容器實例時，如何提取存儲在 Azure 容器註冊表中的容器映射。 配置註冊表訪問的推薦方法是創建 Azure 活動目錄服務主體和密碼，並將登錄憑據存儲在 Azure 金鑰保存庫中。

## <a name="prerequisites"></a>Prerequisites

**Azure 容器註冊表**：您需要一個 Azure 容器註冊表，並且註冊表中至少有一個容器映射來完成本文中的步驟。 如果您需要登錄，請參閱[使用 Azure CLI 建立容器登錄](../container-registry/container-registry-get-started-azure-cli.md)。

**Azure CLI**：本文中的命令列範例使用 [Azure CLI](/cli/azure/)，並使用 Bash 殼層適用的格式。 您可以在本機[安裝 Azure CLI](/cli/azure/install-azure-cli)，或使用 [Azure Cloud Shell][cloud-shell-bash]。

## <a name="configure-registry-authentication"></a>設定登錄驗證

在提供對"無頭"服務和應用程式的訪問的生產方案中，建議使用[服務主體](../container-registry/container-registry-auth-service-principal.md)配置註冊表訪問。 服務主體允許您向容器映射提供[基於角色的存取控制](../container-registry/container-registry-roles.md)。 例如，您可以設定服務主體具有僅限提取登錄的存取權。

Azure 容器註冊表提供了其他[身份驗證選項](../container-registry/container-registry-authentication.md)。

> [!NOTE]
> 不能通過使用相同的容器組中配置的[託管標識](container-instances-managed-identity.md)，向 Azure 容器註冊表進行身份驗證以在容器組部署期間提取映射。

在下一節中，您會建立 Azure 金鑰保存庫和服務主體，並將服務主體的認證儲存在保存庫中。 

### <a name="create-key-vault"></a>建立金鑰保存庫

如果您在 [Azure Key Vault](../key-vault/key-vault-overview.md) 中還沒有保存庫，使用 Azure CLI 以下列命令建立一個。

將 `RES_GROUP` 變數更新為您將在其中建立金鑰保存庫之現有資源群組的名稱，將 `ACR_NAME` 更新為容器登錄的名稱。 為簡潔起見，本文中的命令假定註冊表、金鑰保存庫和容器實例都在同一資源組中創建。

 在 `AKV_NAME` 指定新金鑰保存庫的名稱。 保存庫名稱在 Azure 內必須是唯一的，長度介於 3 到 24 個英數字元之間，以字母開頭、以字母或數字作為結尾，且不可包含連續的連字號。

```azurecli
RES_GROUP=myresourcegroup # Resource Group name
ACR_NAME=myregistry       # Azure Container Registry registry name
AKV_NAME=mykeyvault       # Azure Key Vault vault name

az keyvault create -g $RES_GROUP -n $AKV_NAME
```

### <a name="create-service-principal-and-store-credentials"></a>建立服務主體並儲存認證

現在創建服務主體並將其憑據存儲在金鑰保存庫中。

下列命令使用 [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] 建立服務主體，以及使用 [az keyvault secret set][az-keyvault-secret-set] 將服務主體的**密碼**儲存在保存庫中。

```azurecli
# Create service principal, store its password in vault (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name http://$ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)
```

在前面的命令中，`--role` 引數設定服務主體具有 acrpull** 角色，授與主體僅限提取登錄的存取權。 若要同時授與發送和提取存取權，請將 `--role` 引數變更為 acrpush**。

接下來，將服務主體的*appId*存儲在保存庫中，這是您傳遞給 Azure 容器註冊表進行身份驗證的**使用者名**。

```azurecli
# Store service principal ID in vault (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

您已建立 Azure Key Vault，並在其中儲存兩個祕密：

* `$ACR_NAME-pull-usr`：服務主體識別碼，作為容器登錄的**使用者名稱**。
* `$ACR_NAME-pull-pwd`：服務主體密碼，作為容器登錄的**密碼**。

現在，當您或應用程式和服務從登錄提取映像時，您可以依名稱參考這些祕密。

## <a name="deploy-container-with-azure-cli"></a>使用 Azure CLI 部署容器

現在，服務主體認證儲存在 Azure Key Vault 祕密中，應用程式和服務可以使用它們來存取您的私人登錄。

先使用 [az acr show][az-acr-show] 命令來取得登錄的登入伺服器名稱。 登入伺服器名稱全部都是小寫，類似 `myregistry.azurecr.io`。

```azurecli
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --resource-group $RES_GROUP --query "loginServer" --output tsv)
```

執行下列 [az container create][az-container-create] 命令來部署容器執行個體。 此命令使用儲存在 Azure Key Vault 中的服務主體認證來向容器登錄驗證，並假設您已事先將 [aci-helloworld](container-instances-quickstart.md) 映像推送到登錄。 如果您想要使用不同於登錄的映像，請更新 `--image` 的值。

```azurecli
az container create \
    --name aci-demo \
    --resource-group $RES_GROUP \
    --image $ACR_LOGIN_SERVER/aci-helloworld:v1 \
    --registry-login-server $ACR_LOGIN_SERVER \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label aci-demo-$RANDOM \
    --query ipAddress.fqdn
```

`--dns-name-label` 值在 Azure 內必須是唯一的，上述命令才能將一個隨機數字附加至容器的 DNS 名稱標籤。 命令的輸出會顯示容器的完整網域名稱 (FQDN)，例如：

```output
"aci-demo-25007.eastus.azurecontainer.io"
```

一旦容器啟動成功，您可以在瀏覽器中瀏覽到容器的 FQDN，以確認應用程式順利執行。

## <a name="deploy-with-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本進行部署

通過在容器組定義中包括該屬性，`imageRegistryCredentials`可以在 Azure 資源管理器範本中指定 Azure 容器註冊表的屬性。 例如，您可以直接指定註冊表憑據：

```JSON
[...]
"imageRegistryCredentials": [
  {
    "server": "imageRegistryLoginServer",
    "username": "imageRegistryUsername",
    "password": "imageRegistryPassword"
  }
]
[...]
```

有關完整的容器組設置，請參閱[資源管理器範本引用](/azure/templates/Microsoft.ContainerInstance/2018-10-01/containerGroups)。    

如需參考 Resource Manager 範本中 Azure Key Vault 祕密的詳細資訊，請參閱[在部署期間使用 Azure Key Vault 傳遞安全的參數值](../azure-resource-manager/templates/key-vault-parameter.md)。

## <a name="deploy-with-azure-portal"></a>使用 Azure 入口網站進行部署

如果您在 Azure Container Registry 中保有容器映像，您就可以使用 Azure 入口網站在 Azure Container Instances 中輕鬆地建立容器。 使用入口網站從容器登錄部署容器執行個體時，您必須啟用登錄的[管理帳戶](../container-registry/container-registry-authentication.md#admin-account)。 管理帳戶是專為讓單一使用者存取登錄而設計，主要用於測試。 

1. 在 Azure 入口網站中，瀏覽到您的容器登錄。

1. 若要確認已啟用管理帳戶，請選取 [存取金鑰]****，然後在 [管理使用者]**** 之下選取 [啟用]****。

1. 選取 [存放庫]****，接著選取您想要從中部署的存放庫、以滑鼠右鍵按一下您想要部署之容器映像的標籤，然後選取 [執行執行個體]****。

    ![Azure 入口網站內 Azure Container Registry 中的 [執行執行個體]][acr-runinstance-contextmenu]

1. 分別為容器和資源群組輸入名稱。 如有需要，您也可以變更預設值。

    ![建立 Azure Container Instances 的功能表][acr-create-deeplink]

1. 部署完成之後，您可以從 [通知] 窗格瀏覽至容器群組，以尋找其 IP 位址和其他屬性。

    ![Azure Container Instances 容器群組的詳細資料檢視][aci-detailsview]

## <a name="next-steps"></a>後續步驟

如需有關 Azure Container Registry 驗證的詳細資訊，請參閱[向 Azure Container Registry 進行驗證](../container-registry/container-registry-authentication.md)。

<!-- IMAGES -->
[acr-create-deeplink]: ./media/container-instances-using-azure-container-registry/acr-create-deeplink.png
[aci-detailsview]: ./media/container-instances-using-azure-container-registry/aci-detailsview.png
[acr-runinstance-contextmenu]: ./media/container-instances-using-azure-container-registry/acr-runinstance-contextmenu.png

<!-- LINKS - External -->
[cloud-shell-bash]: https://shell.azure.com/bash
[cloud-shell-try-it]: https://shell.azure.com/powershell

<!-- LINKS - Internal -->
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-create]: /cli/azure/container#az-container-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set

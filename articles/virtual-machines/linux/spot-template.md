---
title: 使用樣本部署 Azure Spot VM
description: 瞭解如何使用範本部署 Spot VM 以節省成本。
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 03/25/2020
ms.author: cynthn
ms.openlocfilehash: 1352761e308aa2e26864654dae65c290df47102b
ms.sourcegitcommit: 980c3d827cc0f25b94b1eb93fd3d9041f3593036
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/02/2020
ms.locfileid: "80548245"
---
# <a name="deploy-spot-vms-using-a-resource-manager-template"></a>使用資源管理員樣本部署 Spot VM

使用[Spot VM](spot-vms.md)使您能夠利用我們未使用的容量,從而顯著節省成本。 在 Azure 需要返回容量的任何時間點,Azure 基礎結構將驅逐 Spot VM。 因此,Spot VM 非常適合處理批次處理作業、開發/測試環境、大型計算工作負載等中斷的工作負載。

現貨 VM 的定價基於區域和 SKU 是可變的。 有關詳細資訊,請參閱[Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)和[Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/)的 VM 定價。

您可以選擇為 VM 設置您願意每小時支付的最高價格。 Spot VM 的最高價格可以用美元 (USD) 設置,最多使用 5 個小數位。 例如,該值`0.98765`將是每小時 0.98765 美元的最高價格。 如果將最高價格設置為`-1`,則 VM 不會根據價格被逐出。 VM 的價格將是 Spot 的當前價格或標準 VM 的價格,只要容量和配額可用,標準 VM 的價格就更少了。 有關設置最高價格的詳細資訊,請參閱[現貨 VM - 定價](spot-vms.md#pricing)。


## <a name="use-a-template"></a>使用範本

對於 Spot 範本`"apiVersion": "2019-03-01"`部署, 請使用或更高版本。 `priority``billingProfile`新增與`evictionPolicy`屬性:

```json
"priority": "Spot",
"evictionPolicy": "Deallocate",
"billingProfile": {
    "maxPrice": -1
}
```

下面是一個範例範本,其中包含 Spot VM 的添加屬性。 將資源名稱取代為您自己的資源名稱,`<password>`並在 VM 上用本地管理員帳戶的密碼替換資源名稱。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2019-03-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {
        "vnetId": "/subscriptions/ec9fcd04-e188-48b9-abfc-abcd515f1836/resourceGroups/spotVM/providers/Microsoft.Network/virtualNetworks/spotVM",
        "subnetName": "default",
        "networkInterfaceName": "spotVMNIC",
        "publicIpAddressName": "spotVM-ip",
        "publicIpAddressType": "Dynamic",
        "publicIpAddressSku": "Basic",
        "virtualMachineName": "spotVM",
        "osDiskType": "Premium_LRS",
        "virtualMachineSize": "Standard_D2s_v3",
        "adminUsername": "azureuser",
        "adminPassword": "<password>",
        "diagnosticsStorageAccountName": "diagstoragespot2019",
        "diagnosticsStorageAccountId": "Microsoft.Storage/storageAccounts/diagstoragespot2019",
        "diagnosticsStorageAccountType": "Standard_LRS",
        "diagnosticsStorageAccountKind": "Storage",
        "subnetRef": "[concat(variables('vnetId'), '/subnets/', variables('subnetName'))]"
    },
    "resources": [
        {
            "name": "spotVM",
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2019-03-01",
            "location": "eastus",
            "dependsOn": [
                "[concat('Microsoft.Network/publicIpAddresses/', variables('publicIpAddressName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIpAddress": {
                                "id": "[resourceId(resourceGroup().name, 'Microsoft.Network/publicIpAddresses', variables('publicIpAddressName'))]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "name": "[variables('publicIpAddressName')]",
            "type": "Microsoft.Network/publicIpAddresses",
            "apiVersion": "2019-02-01",
            "location": "eastus",
            "properties": {
                "publicIpAllocationMethod": "[variables('publicIpAddressType')]"
            },
            "sku": {
                "name": "[variables('publicIpAddressSku')]"
            }
        },
        {
            "name": "[variables('virtualMachineName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2019-03-01",
            "location": "eastus",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', variables('networkInterfaceName'))]",
                "[concat('Microsoft.Storage/storageAccounts/', variables('diagnosticsStorageAccountName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[variables('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[variables('osDiskType')]"
                        }
                    },
                    "imageReference": {
                        "publisher": "Canonical",
                        "offer": "UbuntuServer",
                        "sku": "18.04-LTS",
                        "version": "latest"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('networkInterfaceName'))]"
                        }
                    ]
                },
                "osProfile": {
                    "computerName": "[variables('virtualMachineName')]",
                    "adminUsername": "[variables('adminUsername')]",
                    "adminPassword": "[variables('adminPassword')]"
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true,
                        "storageUri": "[concat('https://', variables('diagnosticsStorageAccountName'), '.blob.core.windows.net/')]"
                    }
                },
                "priority": "Spot",
                "evictionPolicy": "Deallocate",
                "billingProfile": {
                    "maxPrice": -1
                }
            }
        },
        {
            "name": "[variables('diagnosticsStorageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "location": "eastus",
            "properties": {},
            "kind": "[variables('diagnosticsStorageAccountKind')]",
            "sku": {
                "name": "[variables('diagnosticsStorageAccountType')]"
            }
        }
    ],
    "outputs": {
        "adminUsername": {
            "type": "string",
            "value": "[variables('adminUsername')]"
        }
    }
}
```

## <a name="next-steps"></a>後續步驟

您還可以使用[Azure PowerShell](../windows/spot-powershell.md)或[Azure CLI](spot-cli.md)創建 Spot VM。

如果遇到錯誤,請參閱[錯誤代碼](../error-codes-spot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
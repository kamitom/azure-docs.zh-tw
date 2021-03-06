---
title: 範本函數 - 邏輯
description: 描述 Azure Resource Manager 範本中用來決定邏輯值的函式。
ms.topic: conceptual
ms.date: 04/15/2019
ms.openlocfilehash: f058baa32e5f93a4177913287a5e9873fa7a9acb
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "80156305"
---
# <a name="logical-functions-for-arm-templates"></a>ARM 範本的邏輯函數

資源管理器提供了多個功能，用於在 Azure 資源管理器 （ARM） 範本中進行比較。

* [和](#and)
* [bool](#bool)
* [if](#if)
* [否](#not)
* [或](#or)

## <a name="and"></a>和

`and(arg1, arg2, ...)`

檢查所有參數值是否為 true。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 描述 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |boolean |要檢查是否為 true 的第一個值。 |
| arg2 |是 |boolean |要檢查是否為 true 的第二個值。 |
| 其他引數 |否 |boolean |要檢查是否為 true 的其他引數。 |

### <a name="return-value"></a>傳回值

如果所有值都是 true，則傳回 **True**，否則會傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/andornot.json)會示範如何使用邏輯函式。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [ ],
    "outputs": {
        "andExampleOutput": {
            "value": "[and(bool('true'), bool('false'))]",
            "type": "bool"
        },
        "orExampleOutput": {
            "value": "[or(bool('true'), bool('false'))]",
            "type": "bool"
        },
        "notExampleOutput": {
            "value": "[not(bool('true'))]",
            "type": "bool"
        }
    }
}
```

前述範例的輸出為：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| andExampleOutput | Bool | False |
| orExampleOutput | Bool | True |
| notExampleOutput | Bool | False |

## <a name="bool"></a>bool

`bool(arg1)`

將參數轉換為布林值。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 描述 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |字串或整數 |要轉換為布林值的值。 |

### <a name="return-value"></a>傳回值
轉換值的布林值。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/bool.json)顯示如何搭配使用 bool 與字串或整數。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "trueString": {
            "value": "[bool('true')]",
            "type" : "bool"
        },
        "falseString": {
            "value": "[bool('false')]",
            "type" : "bool"
        },
        "trueInt": {
            "value": "[bool(1)]",
            "type" : "bool"
        },
        "falseInt": {
            "value": "[bool(0)]",
            "type" : "bool"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| trueString | Bool | True |
| falseString | Bool | False |
| trueInt | Bool | True |
| falseInt | Bool | False |

## <a name="if"></a>if

`if(condition, trueValue, falseValue)`

根據條件是 true 或 false 傳回值。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 描述 |
|:--- |:--- |:--- |:--- |
| condition (條件) |是 |boolean |要檢查它是真還是假的值。 |
| trueValue |是 | 字串、int、物件或陣列 |條件為 true 時，傳回的值。 |
| falseValue |是 | 字串、int、物件或陣列 |條件為 false 時，傳回的值。 |

### <a name="return-value"></a>傳回值

當第一個參數是 **True** 時，傳回第二個參數；否則會傳回第三個參數。

### <a name="remarks"></a>備註

當條件為**True**時，僅計算真實值。 當條件為**False**時，僅計算 false 值。 使用**if**函數，可以包含僅有條件有效的運算式。 例如，可以引用在一個條件下存在但另一個條件下不存在的資源。 有條件地計算運算式的示例如下節所示。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/if.json)會示範如何使用 `if` 函式。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
    ],
    "outputs": {
        "yesOutput": {
            "type": "string",
            "value": "[if(equals('a', 'a'), 'yes', 'no')]"
        },
        "noOutput": {
            "type": "string",
            "value": "[if(equals('a', 'b'), 'yes', 'no')]"
        },
        "objectOutput": {
            "type": "object",
            "value": "[if(equals('a', 'a'), json('{\"test\": \"value1\"}'), json('null'))]"
        }
    }
}
```

前述範例的輸出為：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| yesOutput | String | 是 |
| noOutput | String | 否 |
| objectOutput | Object | { "test": "value1" } |

以下[示例範本](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/conditionWithReference.json)演示如何將此函數與僅有條件有效的運算式一起使用。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "logAnalytics": {
            "type": "string",
            "defaultValue": ""
        }
    },
    "resources": [
        {
            "condition": "[not(empty(parameters('logAnalytics')))]",
            "name": "[concat(parameters('vmName'),'/omsOnboarding')]",
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2017-03-30",
            "properties": {
                "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                "type": "MicrosoftMonitoringAgent",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "workspaceId": "[if(not(empty(parameters('logAnalytics'))), reference(parameters('logAnalytics'), '2015-11-01-preview').customerId, json('null'))]"
                },
                "protectedSettings": {
                    "workspaceKey": "[if(not(empty(parameters('logAnalytics'))), listKeys(parameters('logAnalytics'), '2015-11-01-preview').primarySharedKey, json('null'))]"
                }
            }
        }
    ],
    "outputs": {
        "mgmtStatus": {
            "type": "string",
            "value": "[if(not(empty(parameters('logAnalytics'))), 'Enabled monitoring for VM!', 'Nothing to enable')]"
        }
    }
}
```

## <a name="not"></a>否

`not(arg1)`

將布林值轉換為其相反值。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 描述 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |boolean |要轉換的值。 |

### <a name="return-value"></a>傳回值

當參數是 **False** 時，傳回 **True**。 當參數是 **True** 時，傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/andornot.json)會示範如何使用邏輯函式。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [ ],
    "outputs": {
        "andExampleOutput": {
            "value": "[and(bool('true'), bool('false'))]",
            "type": "bool"
        },
        "orExampleOutput": {
            "value": "[or(bool('true'), bool('false'))]",
            "type": "bool"
        },
        "notExampleOutput": {
            "value": "[not(bool('true'))]",
            "type": "bool"
        }
    }
}
```

前述範例的輸出為：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| andExampleOutput | Bool | False |
| orExampleOutput | Bool | True |
| notExampleOutput | Bool | False |

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/not-equals.json)使用 **not** 搭配 [equals](template-functions-comparison.md#equals)。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
    ],
    "outputs": {
        "checkNotEquals": {
            "type": "bool",
            "value": "[not(equals(1, 2))]"
        }
    }
```

前述範例的輸出為：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| checkNotEquals | Bool | True |

## <a name="or"></a>或

`or(arg1, arg2, ...)`

檢查任何參數值是否為 true。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 描述 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |boolean |要檢查是否為 true 的第一個值。 |
| arg2 |是 |boolean |要檢查是否為 true 的第二個值。 |
| 其他引數 |否 |boolean |要檢查是否為 true 的其他引數。 |

### <a name="return-value"></a>傳回值

如果任何值為 true，傳回 **True**，否則會傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/andornot.json)會示範如何使用邏輯函式。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [ ],
    "outputs": {
        "andExampleOutput": {
            "value": "[and(bool('true'), bool('false'))]",
            "type": "bool"
        },
        "orExampleOutput": {
            "value": "[or(bool('true'), bool('false'))]",
            "type": "bool"
        },
        "notExampleOutput": {
            "value": "[not(bool('true'))]",
            "type": "bool"
        }
    }
}
```

前述範例的輸出為：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| andExampleOutput | Bool | False |
| orExampleOutput | Bool | True |
| notExampleOutput | Bool | False |

## <a name="next-steps"></a>後續步驟

* 有關 Azure 資源管理器範本中部分的說明，請參閱[創作 Azure 資源管理器範本](template-syntax.md)。
* 要合併多個範本，請參閱[使用 Azure 資源管理器使用連結範本](linked-templates.md)。
* 要反覆運算創建資源類型時指定的次數，請參閱[在 Azure 資源管理器中創建多個資源實例](copy-resources.md)。
* 若要了解如何部署已建立的範本，請參閱[使用 Azure Resource Manager 範本部署應用程式](deploy-powershell.md)。


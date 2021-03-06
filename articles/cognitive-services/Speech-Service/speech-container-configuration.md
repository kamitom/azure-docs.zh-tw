---
title: 設定語音容器
titleSuffix: Azure Cognitive Services
description: 語音服務為每個容器提供一個通用配置框架,以便您可以輕鬆配置和管理容器的存儲、日誌記錄和遙測以及安全設置。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 5c74aa48b18661236eb55278d1e5a05215b2432c
ms.sourcegitcommit: 2d7910337e66bbf4bd8ad47390c625f13551510b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80877570"
---
# <a name="configure-speech-service-containers"></a>設定語音服務容器

語音容器使客戶能夠構建一個經過優化的語音應用程式體系結構,以利用強大的雲端功能和邊緣區域性。 我們現在支援的四個語音容器是:**文字、****自訂語音到文字**、**文字到語音**轉換和**自訂文字到語音**。

**使用**`docker run`指令參數設定語音容器執行時環境。 此容器有數個必要的設定，和一些選擇性的設定。 命令有相關[範例](#example-docker-run-commands)可供參考。 容器專屬設定包括計費設定。

## <a name="configuration-settings"></a>組態設定

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> [`Billing`](#billing-configuration-setting)與[`ApiKey`](#apikey-configuration-setting)設定一起使用,您必須為所有三個設置提供有效值;因此,您必須為所有三個設置[`Eula`](#eula-setting)提供有效值。否則您的容器將無法啟動。 如需使用這些組態設定來將容器具現化的詳細資訊，請參閱[帳單](speech-container-howto.md#billing)。

## <a name="apikey-configuration-setting"></a>ApiKey 組態設定

`ApiKey` 設定會指定用來追蹤容器帳單資訊的 Azure資源金鑰。 您必須為 ApiKey 指定值,並且該值必須[`Billing`](#billing-configuration-setting)是為 設定設定指定的_語音_資源的有效鍵。

此設定可在下列位置找到：

- Azure 門戶:**語音**資源管理,在**鍵**下

## <a name="applicationinsights-setting"></a>ApplicationInsights 設定

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Billing 組態設定

該`Billing`設置指定 Azure 上的_語音_資源的終結點 URI,用於計量容器的計費資訊。 您必須為此設定設定指定值,並且該值必須是 Azure 上的_語音_資源的有效終結點 URI。 容器會每隔 10 到 15 分鐘回報使用量。

此設定可在下列位置找到：

- Azure 門戶:**語音**概述 標記為`Endpoint`

| 必要 | 名稱 | 資料類型 | 描述 |
| -------- | ---- | --------- | ----------- |
| 是 | `Billing` | String | 計費終結點URI。 有關取得計費 URI 的詳細資訊,請參閱[收集所需的參數](speech-container-howto.md#gathering-required-parameters)。 如需詳細資訊和完整的區域端點清單，請參閱[認知服務的自訂子網域名稱](../cognitive-services-custom-subdomains.md)。 |

## <a name="eula-setting"></a>Eula 設定

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Fluentd 設定

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>HTTP 代理認證設定

[!INCLUDE [Container shared HTTP proxy settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>記錄設定

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>裝載設定

使用繫結裝載將資料讀取和寫入至容器，及從中讀取和寫入。 您可以在 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令中指定 `--mount` 選項，以指定輸入裝載或輸出裝載。

標準語音容器不使用輸入或輸出裝載來存儲訓練或服務數據。 但是,自定義語音容器依賴於卷裝載。

主機裝載位置的正確語法會隨著主機作業系統而有所不同。 此外,由於 Docker 服務帳戶使用的許可權與主機裝載位置許可權之間的衝突,[主機](speech-container-howto.md#the-host-computer)的裝載位置可能無法訪問。

| 選用 | 名稱 | 資料類型 | 描述 |
| -------- | ---- | --------- | ----------- |
| 不允許 | `Input` | String | 標準語音容器不使用它。 自訂語音容器使用[捲入載入](#volume-mount-settings)。                                                                                    |
| 選用 | `Output` | String | 輸出裝載的目標。 預設值是 `/output`。 這是記錄的位置。 這包括容器記錄。 <br><br>範例：<br>`--mount type=bind,src=c:\output,target=/output` |

## <a name="volume-mount-settings"></a>捲安裝設定

自定義語音容器使用[卷裝載](https://docs.docker.com/storage/volumes/)來保留自定義模型。 您可以通過`-v`將`--volume`(或 ) 選項新增到 Docker [run](https://docs.docker.com/engine/reference/commandline/run/)命令來指定捲載入。

首次下載自定義模型時,新模型被引入為自定義語音容器 Docker 運行命令的一部分。 自定義語音容器的序列`ModelId`運行將使用以前下載的模型。 如果未提供卷裝載,則無法保留自定義模型。

捲安裝設定由三個顏色`:`分隔欄位組成:

1. 第一個字段是主機上的卷的名稱,例如_C:\input_。
2. 第二個字段是容器中的目錄,例如 _/usr/local/模型_。
3. 第三個字段(選擇)是逗號分隔的選項清單,有關詳細資訊,請參閱[使用卷](https://docs.docker.com/storage/volumes/)。

### <a name="volume-mount-example"></a>碟機載範例

```bash
-v C:\input:/usr/local/models
```

此命令將主機_C:\輸入_目錄裝載到容器 _/usr/local/模型_目錄。

> [!IMPORTANT]
> 捲裝載設定僅適用於**自訂語音到文本**和**自訂文本到語音轉換**容器。 標準 **「語音到文字」** 和 **「文字到語音」** 容器不使用捲裝載。

## <a name="example-docker-run-commands"></a>範例 docker run 命令

下列範例會使用組態設定來說明如何撰寫和使用 `docker run` 命令。 開始執行後，容器就會持續執行，直到您加以[停止](speech-container-howto.md#stop-the-container)。

- **行延續字元**:以下部分中的 Docker 命令使用反`\`斜杠 , 作為行延續字元。 請根據您主機作業系統的需求加以替換或移除。
- **參數順序**:除非熟悉 Docker 容器,否則不要更改參數的順序。

請將 {_argument_name_} 取代為您自己的值：

| 預留位置 | 值 | 格式或範例 |
| ----------- | ----- | ----------------- |
| **[API_KEY]** | Azure`Speech``Speech`鍵頁上資源的終結點鍵。   | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`                                                                                  |
| **[ENDPOINT_URI]** | 計費終結點值在 Azure`Speech`概述頁上可用。 | 有關顯式範例,請參閱[收集所需的參數](speech-container-howto.md#gathering-required-parameters)。 |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> 必須指定 `Eula`、`Billing` 及 `ApiKey` 選項以執行容器，否則容器將不會啟動。 如需詳細資訊，請參閱[帳單](#billing-configuration-setting)。
> ApiKey 值是 Azure 語音轉換資源鍵頁中的 **「鍵**」。

## <a name="speech-container-docker-examples"></a>語音容器 Docker 範例

以下 Docker 範例適用於語音容器。

## <a name="speech-to-text"></a>[語音轉文字](#tab/stt)

### <a name="basic-example-for-speech-to-text"></a>語音到文字的基本範例

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
containerpreview.azurecr.io/microsoft/cognitive-services-speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-to-text"></a>記錄語音到文字的範例

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
containerpreview.azurecr.io/microsoft/cognitive-services-custom-speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-speech-to-text"></a>[自訂語音到文字](#tab/cstt)

### <a name="basic-example-for-custom-speech-to-text"></a>自訂語音到文字的基本範例

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
containerpreview.azurecr.io/microsoft/cognitive-services-custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-speech-to-text"></a>記錄自訂語音到文字的範例

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
containerpreview.azurecr.io/microsoft/cognitive-services-custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="text-to-speech"></a>[文字轉換語音](#tab/tss)

### <a name="basic-example-for-text-to-speech"></a>文字到語音的基本範例

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-text-to-speech"></a>紀錄文字到語音轉換的範例

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-text-to-speech"></a>[自訂文字到語音](#tab/ctts)

### <a name="basic-example-for-custom-text-to-speech"></a>自訂文字到語音轉換的基本範例

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
containerpreview.azurecr.io/microsoft/cognitive-services-custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-text-to-speech"></a>記錄自訂文字到語音轉換的範例

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
containerpreview.azurecr.io/microsoft/cognitive-services-custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

---

## <a name="next-steps"></a>後續步驟

- 檢閱[如何安裝及執行容器](speech-container-howto.md)

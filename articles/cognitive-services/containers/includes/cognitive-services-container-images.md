---
title: 容器存放庫及影像
services: cognitive-services
author: aahill
manager: nitinme
description: 兩個表,表示所有認知服務產品提供的容器註冊表、存儲庫和圖像名稱。
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: a854a090af908da691e9b26f5b0714c6560fc0ba
ms.sourcegitcommit: 2d7910337e66bbf4bd8ad47390c625f13551510b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/08/2020
ms.locfileid: "80876811"
---
### <a name="container-repositories-and-images"></a>容器存放庫及影像

下表是 Azure 認知服務提供的可用容器映像的清單。 有關所有可用容器映像名稱及其可用標記的完整清單,請參閱[認知服務容器映像標記](../container-image-tags.md)。 目前,沒有一般可用的認知服務容器 (GA)。 目前,在進一步公告之前,容器可作為*公共封閉*預覽或*公共門控預覽*提供。

 - *公共隔離*:容器在沒有門控機制的情況下公開提供。
 - *公共門控預覽*:容器是公開的,但首先需要正式請求才能訪問容器註冊表。

#### <a name="public-ungated-container-registry-mcrmicrosoftcom"></a>公開已註冊(容器註冊: `mcr.microsoft.com`)

Microsoft 容器註冊表 (MCR) 將認知服務的所有公開可用的"已開放"容器聯合在一起。 容器也可以直接從[Docker 集線器獲得](https://hub.docker.com/_/microsoft-azure-cognitive-services)。

| 服務 | 容器 | 容器註冊表 / 儲存函式庫 / 映像名稱 |
|--|--|--|
| [路易士](../../LUIS/luis-container-howto.md) | LUIS | `mcr.microsoft.com/azure-cogni'ive-services/luis` |
| [文字分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 關鍵片語擷取 | `mcr.microsoft.com/azure-cognitive-services/keyphrase` |
| [文字分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 語言偵測 | `mcr.microsoft.com/azure-cognitive-services/language` |
| [文字分析](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | 情感分析 | `mcr.microsoft.com/azure-cognitive-services/sentiment` |

#### <a name="public-gated-preview-container-registry-containerpreviewazurecrio"></a>公開「封閉」預覽 (容器註冊表`containerpreview.azurecr.io`: )

容器預覽註冊表承載認知服務的所有公開可用的「門控」容器。 這些容器需要正式請求通過其容器註冊表訪問它們。

| 服務 | 容器 | 容器註冊表 / 儲存函式庫 / 映像名稱 |
|--|--|--|
| [例外偵測器](../../anomaly-detector/anomaly-detector-container-howto.md) | 異常偵測器 | `containerpreview.azurecr.io/microsoft/cognitive-services-anomaly-detector` |
| [電腦視覺](../../Computer-vision/computer-vision-how-to-install-containers.md) | 讀取 | `containerpreview.azurecr.io/microsoft/cognitive-services-read` |
| [臉部](../../face/face-how-to-install-containers.md) | 臉部 | `containerpreview.azurecr.io/microsoft/cognitive-services-face` |
| [表單辨識器](https://go.microsoft.com/fwlink/?linkid=2083826&clcid=0x409) | 表單辨識器 | `containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer` |
| [語音服務 API](../../speech-service/speech-container-howto.md?tab=stt) | 語音轉文字 | `containerpreview.azurecr.io/microsoft/cognitive-services-speech-to-text` |
| [語音服務 API](../../speech-service/speech-container-howto.md?tab=cstt) | 自訂語音到文字 | `containerpreview.azurecr.io/microsoft/cognitive-services-custom-speech-to-text` |
| [語音服務 API](../../speech-service/speech-container-howto.md?tab=tts) | 文字轉換語音 | `containerpreview.azurecr.io/microsoft/cognitive-services-text-to-speech` |
| [語音服務 API](../../speech-service/speech-container-howto.md?tab=ctts) | 自訂文字到語音 | `containerpreview.azurecr.io/microsoft/cognitive-services-custom-text-to-speech` |

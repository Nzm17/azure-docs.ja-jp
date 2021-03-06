---
title: チュートリアル - Azure IoT Central でビデオ分析 (物体とモーションの検出) アプリケーションを作成する (YOLO v3)
description: このチュートリアルでは、IoT Central でビデオ分析アプリケーションを作成する方法について説明します。 それを作成してカスタマイズし、他の Azure サービスに接続します。 このチュートリアルでは、YOLO v3 リアルタイム物体検出システムを使用します。
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
author: KishorIoT
ms.author: nandab
ms.date: 10/06/2020
ms.openlocfilehash: ecc32908aea2fb474d2ebe5bd94f556527eda814
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763448"
---
# <a name="tutorial-create-a-video-analytics---object-and-motion-detection-application-in-azure-iot-central-yolo-v3"></a>チュートリアル:Azure IoT Central でビデオ分析 (物体とモーションの検出) アプリケーションを作成する (YOLO v3)

ソリューション ビルダーとして、IoT Central の "*ビデオ分析 (物体とモーションの検出)* " アプリケーション テンプレート、Azure IoT Edge デバイス、Azure Media Services、YOLO v3 リアルタイム物体およびモーション検出システムを使用してビデオ分析アプリケーションを作成する方法について学習します。 このソリューションでは、小売店を例に、セキュリティ カメラを監視するにあたっての一般的なビジネス ニーズを満たす方法について説明します。 このソリューションでは、ビデオ フィードの自動物体検出を使用して、関心のあるイベントをすばやく識別して見つけ出します。

> [!TIP]
> 物体とモーションの検出のために、YOLO v3 ではなく、OpenVINO&trade; を使用するには、「[チュートリアル: Azure IoT Central でビデオ分析 (物体とモーションの検出) アプリケーションを作成する (OpenVINO&trade;)](tutorial-video-analytics-create-app-openvino.md)」を参照してください。

[!INCLUDE [iot-central-video-analytics-part1](../../../includes/iot-central-video-analytics-part1.md)]

- [Scratchpad.txt](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/Scratchpad.txt) - このファイルは、これらのチュートリアルを実行する際に必要なさまざまな構成オプションを記録するのに役立ちます。
- [deployment.amd64.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/deployment.amd64.json)
- [LvaEdgeGatewayDcm.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/LvaEdgeGatewayDcm.json)
- [state.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/state.json) - このファイルは、2 番目のチュートリアルで Intel NUC デバイスを使用する場合にのみダウンロードする必要があります。

> [!NOTE]
> IoT Edge モジュールである **LvaEdgeGatewayModule** と **lvaYolov3** のソース コードも GitHub リポジトリにあります。 ソース コードを使った作業について詳しくは、[LVA ゲートウェイ モジュールの作成](tutorial-video-analytics-build-module.md)に関するページを参照してください。

[!INCLUDE [iot-central-video-analytics-part2](../../../includes/iot-central-video-analytics-part2.md)]

## <a name="edit-the-deployment-manifest"></a>配置マニフェストを編集する

IoT Edge モジュールは、配置マニフェストを使用してデプロイします。 IoT Central で、このマニフェストをデバイス テンプレートとしてインポートすれば、モジュールのデプロイを IoT Central に管理させることができます。

配置マニフェストを準備するには:

1. テキスト エディターを使用して、*lva-configuration* フォルダーに保存した *deployment.amd64.json* ファイルを開きます。

1. `LvaEdgeGatewayModule` 設定を見つけて、イメージの名前が次のスニペットに示されているとおりであることを確認します。

    ```json
    "LvaEdgeGatewayModule": {
        "settings": {
            "image": "mcr.microsoft.com/lva-utilities/lva-edge-iotc-gateway:1.0-amd64",
    ```

1. `LvaEdgeGatewayModule` セクションの `env` ノードに、Media Services アカウントの名前を追加します。 *scratchpad.txt* ファイルに Media Services アカウントの名前を書き留めました。

    ```json
    "env": {
        "lvaEdgeModuleId": {
            "value": "lvaEdge"
        },
        "amsAccountName": {
            "value": "<YOUR_AZURE_MEDIA_SERVICES_ACCOUNT_NAME>"
        }
    }
    ```

1. これらのプロパティが、テンプレートによって IoT Central に公開されることはありません。Media Services の構成値を自分で配置マニフェストに追加する必要があります。 `lvaEdge` モジュールを探し、プレースホルダーを、Media Services アカウントの作成時に *scratchpad.txt* ファイルに書き留めた値に置き換えます。

    `azureMediaServicesArmId` は、Media Services アカウントの作成時に *scratchpad.txt* ファイルに書き留めた **リソース ID** です。

    次の表は、配置マニフェストで使用する必要がある、*scratchpad.txt* ファイル内の **Media Services への接続 API (JSON)** の値を示しています。

    | 配置マニフェスト       | Scratchpad  |
    | ------------------------- | ----------- |
    | aadTenantId               | AadTenantId |
    | aadServicePrincipalAppId  | AadClientId |
    | aadServicePrincipalSecret | AadSecret   |

    > [!CAUTION]
    > 前の表を使用して、配置マニフェストに適切な値が追加されていることを確認してください。そうでないと、デバイスが動作しません。

    ```json
    {
        "lvaEdge":{
        "properties.desired": {
            "applicationDataDirectory": "/var/lib/azuremediaservices",
            "azureMediaServicesArmId": "[Resource ID from scratchpad]",
            "aadTenantId": "[AADTenantID from scratchpad]",
            "aadServicePrincipalAppId": "[AadClientId from scratchpad]",
            "aadServicePrincipalSecret": "[AadSecret from scratchpad]",
            "aadEndpoint": "https://login.microsoftonline.com",
            "aadResourceId": "https://management.core.windows.net/",
            "armEndpoint": "https://management.azure.com/",
            "diagnosticsEventsOutputName": "AmsDiagnostics",
            "operationalMetricsOutputName": "AmsOperational",
            "logCategories": "Application,Event",
            "AllowUnsecuredEndpoints": "true",
            "TelemetryOptOut": false
            }
        }
    }
    ```

1. 変更を保存します。

このチュートリアルでは、物体とモーションの検出のために YOLO v3 モジュールを使用するようにソリューションを構成します。 次のスニペットは、モジュールの構成を示しています。

```json
"lvaYolov3": {
    "settings": {
        "image": "mcr.microsoft.com/lva-utilities/yolov3-onnx:1.0"
    },
    "type": "docker",
    "status": "running",
    "restartPolicy": "always",
    "version": "1.0"
}
```

[!INCLUDE [iot-central-video-analytics-part3](../../../includes/iot-central-video-analytics-part3.md)]

### <a name="replace-the-manifest"></a>マニフェストを置き換える

**[LVA Edge Gateway v2]** ページで、 **[+ Replace manifest]\(+ マニフェストを置き換える\)** を選択します。

:::image type="content" source="./media/tutorial-video-analytics-create-app-yolo-v3/replace-manifest.png" alt-text="マニフェストを置き換える":::

*lva-configuration* フォルダーに移動し、先ほど編集した *deployment.amd64.json* マニフェスト ファイルを選択します。 **[アップロード]** を選択します。 検証が完了したら、 **[Replace]\(置き換え\)** を選択します。

[!INCLUDE [iot-central-video-analytics-part4](../../../includes/iot-central-video-analytics-part4.md)]

## Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure


Microsoft Azureを使用したNVIDIAハードウェアを使用したIntelligent Video Analyticsのエンドツーエンドアーキテクチャを実証するリポジトリです。 

このプロジェクトには、様々なビデオ入力ソースを処理し、カスタムオブジェクト検出モデルを活用し、分析とレポートのためのクラウドサービスを提供するカスタムIntelligent Video Analyticsアプリケーションを開発する際にユーザーをガイドする、セルフペースの学習モジュールのコレクションが含まれています。

* [Module 1 - Introduction to NVIDIA DeepStream](./docs/Module%201%20-%20Introduction%20to%20NVIDIA%20DeepStream.md)
* [Module 2 - Configure and Deploy "Intelligent Video Analytics" to IoT Edge Runtime on NVIDIA Jetson](./docs/Module%202%20-%20Configure%20and%20Deploy%20Intelligent%20Video%20Analytics%20to%20IoT%20Edge%20Runtime%20on%20NVIDIA%20Jetson.md)
* [Module 3 - Develop and deploy Custom Object Detection Models with IoT Edge DeepStream SDK Module](./docs/Module%203%20-%20Develop%20and%20deploy%20Custom%20Object%20Detection%20Models%20with%20IoT%20Edge%20DeepStream%20SDK%20Module.md)
* [Module 4 - Filtering Telemetry with Azure Stream Analytics at the Edge and Modeling with Azure Time Series Insights](./docs/Module%204%20-%20Filtering%20Telemetry%20with%20Azure%20Stream%20Analytics%20at%20the%20Edge%20and%20Modeling%20with%20Azure%20Time%20Series%20Insights.md)
* [Module 5 - Visualizing Object Detection Data in Near Real-Time with PowerBI](./docs/Module%205%20-%20Visualizing%20Object%20Detection%20Data%20in%20Near%20Real-Time%20with%20PowerBI.md)

これらの各モジュールには、詳細に再現するための手順を説明するLiveStreamが付属しています。Youtubeでは、以下の5部構成の[プレイリスト](https://www.youtube.com/playlist?list=PLzgEG9tLG-1QLc-DPPABoW1YWFMPNQl4t)をチェックすることで、プロジェクト全体を一から見ることができます。



[![5 part video playlist](./assets/Playlist.PNG)](https://www.youtube.com/playlist?list=PLzgEG9tLG-1QLc-DPPABoW1YWFMPNQl4t)

## Overview

[![5 minute teaser](./assets/5minTeaser.PNG)](https://www.youtube.com/watch?v=-DWrxUITSbc)

このプロジェクトでは、[NVIDIA Jetson Embedded hardware](https://www.nvidia.com/en-us/autonomous-machines/jetson-store/)上で動作する[NVIDIA DeepStream SDK](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/nvidia.deepstream-iot?tab=Overview&WT.mc_id=julyot-iva-pdecarlo)を利用して、インテリジェントな動画解析パイプラインを作成しています。

このソリューションでは、[Azure IoT Edge](https://azure.microsoft.com/en-us/services/iot-edge/?WT.mc_id=julyot-iva-pdecarlo)ランタイムを使用して計測されたNVIDIAハードウェアデバイス上で実行される多くのモジュールを採用しています。これらのモジュールには、オブジェクト検出のトレーニングサンプルをキャプチャし、ペアリングされた[Camera Tagging Module](https://dev.to/azure/introduction-to-the-azure-iot-edge-camera-tagging-module-di8)を介してクラウドにミラーリングするための[Azure Blob Storage on IoT Edge Module](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-blob?WT.mc_id=julyot-iva-pdecarlo)が含まれています。これらのキャプチャしたサンプルは、[Azure Cognitive Services](https://docs.microsoft.com/en-us/azure/cognitive-services/?WT.mc_id=julyot-iva-pdecarlo)の[Custom Vision AI](https://www.customvision.ai/?WT.mc_id=julyot-iva-pdecarlo)サービスを使用して、カスタムオブジェクト検出モデルをトレーニングするために使用されます。このサービスによって生成されたモデルは、[Custom Yolo Parser](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/tree/master/services/DEEPSTREAM/YoloParser/CustomVision_DeepStream5.0_JetPack4.4)を使用してDeepStream SDKモジュールによって活用されます。

オブジェクト検出がDeepStream SDKによって生成されると、出力を要約された検出に変換する[Azure Stream Analytics on Edge Job](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-edge?WT.mc_id=julyot-iva-pdecarlo)を使用してフィルタリングされます。これらのオブジェクト検出結果は[Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/?WT.mc_id=julyot-iva-pdecarlo)に送信され、処理とレポートのために追加のクラウドサービスに転送されます。

採用されているクラウドサービスには、時間の経過とともにデータを分析するための完全に管理されたイベント処理サービスである[Time Series Insights](https://docs.microsoft.com/en-us/azure/time-series-insights/?WT.mc_id=julyot-iva-pdecarlo)が含まれています。また、物体検出データを[PowerBI](https://docs.microsoft.com/en-us/power-bi/?WT.mc_id=julyot-iva-pdecarlo)のデータセットに転送して、PowerBIのレポートやダッシュボード内で結果をライブで可視化する方法も実演しています。

![](./assets/IoTHubArchitecture.PNG)


これがどのように機能するかの詳細については、[IoT Show](https://channel9.msdn.com/Shows/Internet-of-Things-Show)のこのエピソードをご覧ください。

[![IoT Show Episode](./assets/IoTShow.PNG)](
https://www.youtube.com/watch?v=EiB1j0FZjgU)

## Prerequisites


ハードウェア

- [JetPack 4.4](https://developer.nvidia.com/embedded/jetpack)を実行している[NVIDIA Jetson 組み込みデバイス](https://www.nvidia.com/en-us/autonomous-machines/jetson-store/)
- Nvidia Jetson Nano デバイスにインストールされている、または Nvidia Jetson Nano デバイスに向けられている冷却ファン
- RTSP対応カメラ（オプション
  - 注：フォスカムの[FI9821P](https://amzn.to/2XzBRFC) をお勧めします。
- USB ウェブカメラ（オプション
  - 注：[Jetson Nano](https://amzn.to/2WFE5zF)を使用する場合、消費電力は、[Open-CV対応カメラ](https://web.archive.org/web/20120815172655/http://opencv.willowgarage.com/wiki/Welcome/OS/)を使用して、ここに記載されているような[5V/4A barrel adapter](https://amzn.to/32DFsTq)を使用するようにデバイスが設定されている必要があります。

開発環境
- [Visual Studio Code (VSCode)](https://code.visualstudio.com/Download?WT.mc_id=github-IntelligentEdgeHOL-pdecarlo)
  - 注意: VSCodeのARM64ビルドは公式にはサポートされていませんが、NVIDIA Jetsonデバイスに開発ツールをインストールして実行することは可能です。リソースの制限があるため、Jetson Nanoハードウェアではお勧めできません。詳細については、「Nvidia JetsonデバイスでのIoTエッジ開発の始め方」の記事を参照してください。
- Visual Studioコード拡張
  - [Azure IoT Tools Extension](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
- Git tool
  [Git command line](https://git-scm.com/) 

クラウドサービス。
- アクティブな[Microsoft Azure Subscription](https://azure.microsoft.com/en-us/get-started?WT.mc_id=julyot-iva-pdecarlo)

## Learn more, get certified

Azure IoT サービスを使ったソリューションの構築について詳しく学びたい方は、以下の無料学習リソースをチェックしてみてください。

* [IoT learning paths on Microsoft Learn](https://docs.microsoft.com/learn/browse/?term=IOT&WT.mc_id=julyot-iva-pdecarlo)
* [The IoT show on Channel9](https://channel9.msdn.com/Shows/Internet-of-Things-Show/?WT.mc_id=julyot-iva-pdecarlo)

IoT開発者としてスキルアップしたら、[AZ-220 Azure IoT Developer certification](https://docs.microsoft.com/learn/certifications/azure-iot-developer-specialty?WT.mc_id=julyot-iva-pdecarlo)で正式なものにしましょう。

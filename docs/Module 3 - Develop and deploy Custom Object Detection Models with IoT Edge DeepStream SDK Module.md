## Module 3 : Develop and deploy Custom Object Detection Models with IoT Edge DeepStream SDK Module

この時点で、必要なソースからの入力を消費できるカスタムのDeepStream構成がデプロイされているはずです。ここでは、その構成で採用されているオブジェクト検出モデルをカスタマイズして、完全にカスタマイズされたIntelligent Video Analytics Pipelineを作成する方法を見ていきます。

このセクションでは、コンピュータビジョン/人工知能の世界に慣れていない方や、カスタムオブジェクト検出モデルを使用して、訓練して検出するオブジェクトを検出するという最終目標に興味がある方を想定しています。カスタムモデルをトレーニングする必要なく、すぐに一般的な物体の正確な検出を得ることに興味がある場合は、[80個の一般的物体](../services/YOLOV3/labels.txt)でトレーニングされたアカデミックグレードの事前トレーニング済み物体検出モデル[(YoloV3)](https://pjreddie.com/darknet/yolo/)を採用する方法を実演します。

このモジュールのステップに沿って進みたい場合は、「[Develop and deploy Custom Object Detection Models with IoT Edge DeepSteam SDK Module](https://www.youtube.com/watch?v=kv0eTobemug)」と題したライブストリーム・プレゼンテーションを録画しましたので、以下のステップを詳細に説明します。

[![Develop and deploy Custom Object Detection Models with IoT Edge DeepSteam SDK Module](../assets/LiveStream3.PNG)](https://www.youtube.com/watch?v=kv0eTobemug)


### Module 3.1 : Training a custom object detection model using Custom Vision AI


注：これらの手順は、NVIDIADeepStreamSDKモジュールが`DSConfig-CustomVisionAI.txt`のDeepStream構成を参照するように構成されていることを前提としています。

Microsoftは[Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/?WT.mc_id=julyot-iva-pdecarlo) の一部として[Custom Vision](https://www.customvision.ai/?WT.mc_id=julyot-iva-pdecarlo)を提供しており、カスタムオブジェクト検出モデルのトレーニングやエクスポートを非常に簡単に行うことができます。開始するには、[customvision.ai](http://customvision.ai/?WT.mc_id=julyot-iva-pdecarlo)でアカウントを作成し、以下のオプションで新しいプロジェクトを開始します。

![New Custom Vision AI Project](../assets/CreateNewCustomVisionAI.PNG)

作成したら、検出したいタグ（オブジェクト）ごとに少なくとも15枚の画像をアップロードします。ここでは、「人」検出器を訓練するために、15枚の自分の画像を使用した例を示します。

![Upload Samples](../assets/UploadSamplesCustomVisionAI.PNG)

訓練セット内の各画像が、目的の対象物の周囲に関連する境界ボックスで適切にタグ付けされていることを確認します。

![Tag Samples](../assets/TagSamplesCustomVisionAI.PNG)

画像にタグを付けた状態で、「トレーニング」を選択し、「クイックトレーニング」を選択します。

![Train Samples](../assets/TrainSamplesCustomVisionAI.PNG)

次に「パフォーマンス」を選択し、「エクスポート」を選択します。

![Export Model](../assets/ExportCustomVisionAI.PNG)

ONNXを選択し、「ダウンロード」ボタンを右クリックして、結果のモデルをダウンロードするためのリンクをコピーします。

![Download Model](../assets/DownloadCustomVisionAI.PNG)

次のセクションで使用するので、この値に注意してください。コピーされたテキストは以下のようになるはずです。

```
https://irisscuprodstore.blob.core.windows.net/m-ad5281beaf20440a8f3f046e0e7741af/e3ebcf6e22934c7e89ae39ffe2049a46.ONNX.zip?sv=2017-04-17&sr=b&sig=A%2F9raRar12TSTCvH7D72OxD6mBqvRY5doovtwV4Bjt0%3D&se=2020-04-15T20%3A43%3A26Z&sp=r
```

### Module 3.2 : Using a Custom Vision AI Object Detection Model with NVIDIA DeepStream

新しいモデルを既存のアプリケーションで使用したい場合は、`/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI`にあるモデルを更新するだけです。

これは以下のコマンドを使ってJetsonデバイス上で行うことができます。前のステップでエクスポートしたカスタムビジョンAIモデルを指すように、`wget`コマンドのリンクを置き換えるだけです。

```
#Delete the contents of the CUSTOM_VISION_AI directory to remove existing model and TensorRT engine
rm -rf /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI/*

#Navigate to CUSTOM_VISION_AI directory
cd /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI

#Download the exported model from CustomVision.AI
#Note: It is important that the link to your model is quoted when running this command!
sudo wget -O model.zip "https://irisscuprodstore.blob.core.windows.net/m-ad5281beaf20440a8f3f046e0e7741af/e3ebcf6e22934c7e89ae39ffe2049a46.ONNX.zip?sv=2017-04-17&sr=b&sig=A%2F9raRar12TSTCvH7D72OxD6mBqvRY5doovtwV4Bjt0%3D&se=2020-04-15T20%3A43%3A26Z&sp=r"

#Unzip the model.zip that we just downloaded
unzip model.zip

#Install dos2unix
sudo apt install -y dos2unix

#Convert labels.txt to Unix format (otherwise DeepStream will append '/r' to object value)
dos2unix labels.txt
```

モデルを更新するたびに（`CUSTOM_VISON_AI`ディレクトリの内容を削除することも含めて）このプロセスに従うことが重要です。 このファイルが既に存在する場合、新しいモデルのために新しいエンジンを再生成したいときに、古いモデルで生成されたエンジンを参照してしまう可能性があります。

data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI`のディレクトリの内容は以下のようになっているはずです。

```
CSharp  
cvexport.manifest  
labels.txt  
LICENSE  
model.onnx  
model.zip  
python
```

技術的には、DeepStreamの設定で必要なのはlabel.txtとmodel.onnxファイルへのアクセスだけです。labels.txtファイルには、モデルがサポートするオブジェクト検出クラスが含まれています。 これは、CustomVision.AIでモデルを訓練するために使用されるタグで構成されます。 model.onnxファイルは、[ONNX](http://onnx.ai/)ベースの[YoloV2Tiny](https://pjreddie.com/darknet/yolov2/)タイプのオブジェクト検出モデルです。 

モデルをダウンロードして抽出した後、DeepStreamSDKモジュールを再起動して、新しいモデルの使用を開始できるようにします。

```
docker rm -f NVIDIADeepStreamSDK
```

NVIDIADeepStreamSDK モジュールが再起動するまでしばらく待ってから、以下を実行して NVIDIADeepStreamSDK モジュールのログを表示します。
```
docker logs -f NVIDIADeepStreamSDK
```

以下のような出力が表示されるはずです。
```
2020-06-10 19:39:48.485400: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudart.so.10.2

 *** DeepStream: Launched RTSP Streaming at rtsp://localhost:8555/ds-test ***


(deepstream-test5-app:1): GLib-CRITICAL **: 19:39:49.193: g_strrstr: assertion 'haystack != NULL' failed
nvds_msgapi_connect : connect success
Opening in BLOCKING MODE

Using winsys: x11
ERROR: Deserialize engine failed because file path: /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs/../../CUSTOM_VISION_AI/model.onnx_b1_gpu0_fp32.engine open error
0:00:03.779721457     1     0x21e4b4d0 WARN                 nvinfer gstnvinfer.cpp:599:gst_nvinfer_logger:<primary_gie> NvDsInferContext[UID 1]: Warning from NvDsInferContextImpl::deserializeEngineAndBackend() <nvdsinfer_context_impl.cpp:1566> [UID = 1]: deserialize engine from file :/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs/../../CUSTOM_VISION_AI/model.onnx_b1_gpu0_fp32.engine failed
0:00:03.779815000     1     0x21e4b4d0 WARN                 nvinfer gstnvinfer.cpp:599:gst_nvinfer_logger:<primary_gie> NvDsInferContext[UID 1]: Warning from NvDsInferContextImpl::generateBackendContext() <nvdsinfer_context_impl.cpp:1673> [UID = 1]: deserialize backend context from engine from file :/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs/../../CUSTOM_VISION_AI/model.onnx_b1_gpu0_fp32.engine failed, try rebuild
0:00:03.779842136     1     0x21e4b4d0 INFO                 nvinfer gstnvinfer.cpp:602:gst_nvinfer_logger:<primary_gie> NvDsInferContext[UID 1]: Info from NvDsInferContextImpl::buildModel() <nvdsinfer_context_impl.cpp:1591> [UID = 1]: Trying to create engine from model files
----------------------------------------------------------------
Input filename:   /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI/model.onnx
ONNX IR version:  0.0.3
Opset version:    7
Producer name:
Producer version:
Domain:           onnxml
Model version:    0
Doc string:
----------------------------------------------------------------
```

エラーメッセージは、参照された`model.onnx`に対してTensorRTエンジンがまだ存在しないことを示しています。 前述のように、エンジンが存在しない場合、DeepStreamはエンジンの再生成を試みます。 最初の実行では、TensorRTエンジンが生成されるため、起動に時間がかかりますが、その後の実行では新しく生成されたエンジンが使用されます。

TensorRTエンジンが再生成された後、`/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI`のディレクトリの内容は以下のようになるはずです(注: `model.onnx_b1_gpu0_fp32.engine`が存在するようになりました)。


```
CSharp
cvexport.manifest  
labels.txt  
LICENSE  
model.onnx  
model.onnx_b1_gpu0_fp32.engine
model.zip  
python
```

### Module 3.3 : Using the Deployed CameraTaggingModule to gather Training Samples

[Intelligent Video Analytics deployment ships with a Camera Tagging Module](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L97) の導入には、CustomVision.AIで使用するためのトレーニング・サンプルの収集を支援するカメラ・タギング・モジュールが含まれています。

![CameraTaggingModule](../assets/CameraTaggingModule.PNG)

[Azure IoT Edge Camera Tagging Module](https://github.com/microsoft/vision-ai-developer-kit/tree/master/samples/official/camera-tagging)は、オープンネットワークとエアギャップネットワークのアクセス可能なRTSPストリームからトレーニングサンプルをキャプチャしてアップロードする自動化された方法を提供することで支援することができ、サンプルを直接CustomVision.AIにアップロードする機能や、Azure Storageコンテナに保存して転送するための[`azureblobstorageoniotedge` module](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L161)をデプロイすることができます。これにより、ソリューションビルダーは、任意のIoT Edge対応デバイス上で実行されているモジュールから収集したデータを使用して、多様で正確なAIモデルを作成することができます。



CameraTaggingModuleの機能を使用する方法の詳細については、こちらの[記事](https://dev.to/azure/introduction-to-the-azure-iot-edge-camera-tagging-module-di8)を参照してください。

### Module 3.4 : The CustomVisionAI YoloParser 


CustomVision.AIからエクスポートしたonnxフォーマットのモデルは、[YoloV2Tiny](https://pjreddie.com/darknet/yolov2/)ベースのオブジェクト検出器です。[Yolo](https://pjreddie.com/darknet/yolo/)は、[Joseph Redmon](https://pjreddie.com/)によって作成されたオブジェクト検出器の包括的な命名規則です。Tiny」は、リソースに制約のあるデバイスを対象としたYolov2*のバージョンを使用していることを示しています。Tiny」バージョンは、「non-Tiny」バージョンの精度と引き換えに、スピードを重視しています。このタイプのモデルは Jetson Nano に適していますが、Jetson Xavier ファミリーのようなより強力なハードウェアではそれを超える可能性があります。

このセクションでは、(`libnvdsinfer_custom_impl_Yolo.so`)を介して.onnxフォーマットされたCustomVision.AIモデルをTensorRTエンジンにパースするためにDeepStreamが使用するCustom Yolo Parserについて説明します。この共有オブジェクトは、[the Primary Inference Engine configuration file](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/DSConfig-CustomVisionAI.txt#L143)では、[`custom-lib-path`の値](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_CustomVisionAI.txt#L79)で参照されています。このファイルでは、[`nvdsparsebbox_Yolo.cpp`](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/YoloParser/CustomVision_DeepStream5.0_JetPack4.4/nvdsparsebbox_Yolo.cpp#L457) に由来する関数に対応する [`parse-bbox-func-name`](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_CustomVisionAI.txt#L78) も言及されています。

CustomVisionAI YoloParserは以下にあります。

```
/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/YoloParser/CustomVision_DeepStream5.0_JetPack4.4
```


このフォルダ内には、カスタムYoloパーサー、`libnvdsinfer_custom_impl_Yolo.so`、ソースからパーサーをビルドする方法を説明するMakefileがあります。このソースと説明は、JetPack 4.4（JetsonデバイスのOS/ライブラリバージョン）上で動作するDeepStream5.0に対してパーサをビルドするためのものです。もし、異なるバージョンのいずれかで動作させたい場合は、ソースから再コンパイルする必要があるでしょう。

[`nvdsparsebbox_Yolo.cpp`](../services/DEEPSTREAM/YoloParser/CustomVision_DeepStream5.0_JetPack4.4/nvdsparsebbox_Yolo.cpp) を修正することに興味があるかもしれません。例えば、432-437行目のコメントを外して、検出されたオブジェクトの信頼度を出力することができます。また、非最大抑制閾値（kNMS_THRESH）と信頼度閾値（`kPROB_THRESH`）を変更して、より良い精度を得るためにモデルを調整することもできます。これらのパラメータについては、以下の[記事](https://towardsdatascience.com/you-only-look-once-yolo-implementing-yolo-in-less-than-30-lines-of-python-code-97fb9835bfd2)で詳しく説明します。

### Module 3.5 : The CustomYolo YoloParser


DeepStream 5.0には、純正のYoloV3のウェイトと構成を扱うことができるパーサーが同梱されています。このパーサーがどのように動作するかの詳細については、DeepStream 4.0のドキュメント「["Custom YOLO Model in the DeepStream YOLO App"](https://docs.nvidia.com/metropolis/deepstream/4.0/Custom_YOLO_Model_in_the_DeepStream_YOLO_App.pdf)」を参照してください。

CustomYolo YoloParserは以下にあります。

```
/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/YoloParser/CustomYolo_DeepStream5.0_JetPack4.4
```

このフォルダ内には、カスタムYoloパーサー、`libnvdsinfer_custom_impl_Yolo.so`と、ソースからパーサーをビルドする方法を説明するMakefileがあります。このソースと説明は、JetPack 4.4 (JetsonデバイスOS/ライブラリバージョン)上で動作するDeepStream5.0に対してパーサをビルドするためのもので、DeepStream5.0がホストにインストールされている必要があります。異なるバージョンのどちらかに対して実行する場合は、ソースから再コンパイルする必要があります。

`/opt/nvidia/deepstream/deepstream-5.0/sources/objectDetector_Yolo/nvdsinfer_custom_impl_Yolonvdsparsebbox_Yolo.cpp` を変更することに興味があるかもしれません。このファイルは、ホストシステムにDeepStream 5.0をインストールした場合にのみ存在することに注意してください。このパーサーでカスタムYoloV3*モデルを使用する場合、Yoloモデルに存在するオブジェクト検出クラスの数を反映させるために、このファイルの33行目を修正する必要があることに注意してください。

```
static const int NUM_CLASSES_YOLO = 80
```


また、より良い精度を得るためにモデルを調整するために、Non-Maximal Suppression Threshold (`nms-iou-threshold`)とConfidence Threshold (`pre-cluster-threshold`)を変更することもできます。それらは[config_infer_primary_yoloV3_tiny.txt](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_yoloV3_tiny.txt#L91) と [config_infer_primary_yoloV3.txt](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_yoloV3.txt)で入手可能です．

これらのパラメータについては、以下の[記事](https://towardsdatascience.com/you-only-look-once-yolo-implementing-yolo-in-less-than-30-lines-of-python-code-97fb9835bfd2)で詳しく説明しています。

### Module 3.6 : Using a YoloV3* Object Detection Model with NVIDIA DeepStream


このセクションでは、DeepStreamの設定でYoloV3またはYoloV3Tinyを使用するための手順を説明します。

Yoloは.weightsと.cfgファイルが必要で、このプロジェクトには含まれていません。これらのファイルは実行することで入手できます。



```
cd /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/YOLOV3

./downloadYoloWeights.sh
```


スクリプトが終了すると、`/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/YOLOV3`ディレクトリに以下のファイルが表示されるはずです。



```
downloadYoloWeights.sh  
yolov3.cfg                        
yolov3-tiny.cfg                        
yolov3-tiny.weights
labels.txt              
yolov3.weights
```


YoloV3 または YoloV3Tiny を NVIDIADeepStreamSDK モジュールで使用するには、[deployment.template.json](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L71) の NVIDIADeepStreamSDK モジュールの Entrypoint を更新して [DSConfig-YoloV3.txt](../services/DEEPSTREAM/configs/DSConfig-YoloV3.txt) または [DSConfig-YoloV3Tiny.txt](../services/DEEPSTREAM/configs/DSConfig-YoloV3Tiny.txt) を参照し、[Module 2.5](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/docs/Module%202%20-%20Configure%20and%20Deploy%20Intelligent%20Video%20Analytics%20to%20IoT%20Edge%20Runtime%20on%20NVIDIA%20Jetson.md#module-25--generate-and-apply-the-iot-hub-based-deployment-configuration) の手順にしたがってディプロイメントを再作成して適用する必要があります。また、[Module 2.6](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/docs/Module%202%20-%20Configure%20and%20Deploy%20Intelligent%20Video%20Analytics%20to%20IoT%20Edge%20Runtime%20on%20NVIDIA%20Jetson.md#module-26--customizing-the-sample-deployment)で説明したカスタマイズを参照するために、新しく参照された設定(DSConfig-YoloV3.txtまたはDSConfig-YoloV3Tiny.txt)を更新する必要があります。

最初の実行時、デフォルトでは、DeepStreamはYoloモデル用のTensorRTエンジンファイルを[現在の作業ディレクトリ](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L88)に作成します。

```
/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs
```


[config_infer_primary_yoloV3.txt](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_yoloV3.txt#L67)と[config_infer_primary_yoloV3_tiny.txt](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_yoloV3_tiny.txt#L66)のYOLOV3*コンフィギュレーションは、DeepStreamで生成された結果のモデルファイル(model_b*_gpu0_fp16.engine)をパスにコピーして名前を変更することで、高速化することができます。



```
/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/YOLOV3/
```

デフォルトでは、YOLOV3の構成ではエンジンの名前が`yolov3_model_b1_gpu0_fp16.engine`となるのに対し、YOLOV3では`yolov3_tiny_model_b1_gpu0_fp16.engine`という名前を期待しています。

DeepStreamが既存のエンジンを参照できる場合（YoloV3の[config_infer_primary_yoloV3.txt](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_yoloV3.txt#L67)または[config_infer_primary_yoloV3_tiny.txt](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/services/DEEPSTREAM/configs/config_infer_primary_yoloV3_tiny.txt#L66)を参照）、その後の実行でTensorRTエンジンを再構築する必要がなくなり、DeepStreamのワークロードの開始時間を大幅に短縮することができます。

特に、Jetson Nano上で複数のビデオソースを使用してYoloV3*推論を実行しているときに、低いフレームレートやラグのあるパフォーマンスに気付いた場合は、以下の手順を実行してください。DeepStream構成の`[primary-gie]``interval`の値を増やすことで、パフォーマンスを改善できる可能性があります。これにより、推論実行を実行する前にスキップする連続したフレームバッチの数が設定され、推論を実行する頻度を減らすことで推論結果をよりリアルタイムに近い形で処理できるようになります。


### Module 3.6: Next Steps

これで、CustomVision.AIモデルまたはYoloV3*を参照するDeepStream構成が動作しているはずです。これで、カスタムのIntelligent Video AnalyticsソリューションからMicrosoft Azure Servicesにオブジェクト検出のテレメトリをプッシュする準備が整いました。
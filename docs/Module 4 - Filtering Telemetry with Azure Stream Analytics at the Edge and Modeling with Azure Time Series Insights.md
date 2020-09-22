## Module 4 : Filtering Telemetry with Azure Stream Analytics at the Edge and Modeling with Azure Time Series Insights

この時点で、NVIDIADeepStreamSDKモジュールによって参照されるDeepStream構成が動作し、[ビデオ入力ソースに合わせてカスタマイズ](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/docs/Module%202%20-%20Configure%20and%20Deploy%20Intelligent%20Video%20Analytics%20to%20IoT%20Edge%20Runtime%20on%20NVIDIA%20Jetson.md#module-26--customizing-the-sample-deployment)され、[カスタムのオブジェクト検出モデルを使用するように構成されている](./Module%203%20-%20Develop%20and%20deploy%20Custom%20Object%20Detection%20Models%20with%20IoT%20Edge%20DeepSteam%20SDK%20Module.md)必要があります。

このモジュールでは、[Azure Stream Analytics on Edge](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-edge?WT.mc_id=julyot-iva-pdecarlo)を使用してDeepStreamオブジェクト検出結果を平坦化、集計、要約し、そのテレメトリを[Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/?WT.mc_id=julyot-iva-pdecarlo)に転送する方法を説明します。次に、[Time Series Insights](https://docs.microsoft.com/en-us/azure/time-series-insights/?WT.mc_id=julyot-iva-pdecarlo)として知られる新しいAzureサービスを導入します。このサービスは、IoT Hubからイベントソースを介して入力を取り込み、IoT Edgeデバイスが生成するオブジェクト検出データ内の異常を分析、クエリ、検出できるようにします。

このモジュールのステップに従いたい場合は、「[Consuming and Modeling Object Detection Data with Azure Time Series Insights](https://www.youtube.com/watch?v=l5tjaD-qYwY)」と題して、以下のステップを詳細に説明するライブストリーム・プレゼンテーションを録画しました。

[![Consuming and Modeling Object Detection Data with Azure Time Series Insights](../assets/LiveStream4.PNG)](https://www.youtube.com/watch?v=l5tjaD-qYwY)

## Module 4.1 : Recreating the Azure Stream Analytics job in Microsoft Azure

ここまでのところ、[デプロイテンプレートは、外国の Azure サブスクリプションに存在する Azure Stream Analytics ジョブを参照しています](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L240)。 この手順では、独自の Azure サブスクリプションの下でそのジョブを再作成し、そのジョブがソリューションに提供する機能の詳細な説明を行います。 これにより、提供されているStream Analytics Queryを好みに合わせてカスタマイズし始めるのに十分な知識が得られるはずです。 次に、新しく作成したAzure Stream Analyticsジョブを参照するために、現在の設定を更新する方法を実演します。

Azureマーケットプレイスに移動し、「Azure Stream Analytics on IoT Edge」を検索します。

![Azure SAS on Edge Marketplace](../assets/AzureSASonEdgeMarketplace.PNG)

Select "Create":

![Create Azure SAS on Edge](../assets/CreateSASonEdge.PNG)


Stream Analyticsジョブに名前を付け、元のIoT Hubと同じ地域に配置されていることを確認し、ホスティング環境が「Edge」に設定されていることを確認し、「このジョブが必要とするすべてのプライベートデータ資産を私のストレージアカウントに確保する」というラベルの付いたボックスがチェックされていないことを確認してから、「Create」を選択します。

![Deploy Azure SAS on Edge](../assets/DeploySASonEdge.PNG)

新しく作成されたジョブに移動し、”Inputs"を選択します。ここでは、 [deployment template route configuration](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L210)で使用される入力エイリアスを構成します。 NVIDIADeepStreamSDK出力が入力としてDeepStreamAnalyticsモジュールに流れるように構成するため、この設定は重要です。 このステップでは名前付けが非常に重要で、ルート構成で使用されるエイリアスと一致している必要があります。 

"Add stream input "を選択し、"Input Alias "に "DeepStreamInput "という名前を付けます。
![Azure SAS on Edge Input](../assets/AzureSASonEdgeInput.PNG)

次に、[deployment template route configuration](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L211)で使用する出力エイリアスを設定する"出力"に移動します。 この設定は、DeepStreamAnalytics Streaming Analyticsのジョブ出力がIoTハブに流れるように構成するために重要です。 このステップでは名前の付け方が非常に重要で、ルート構成で使用するエイリアスと一致している必要があります。 

下図のように「追加」→「Edge Hub」を選択します。
![Azure SAS on Edge Output](../assets/AzureSASonEdgeOutput.PNG)

結果のウィンドウで、`出力エイリアス`に「AggregatedDetections」という名前を付け、「保存」を選択します。
![Azure SAS on Edge Output Aggregated](../assets/AzureSASonEdgeOutputAggregated.PNG)

下図のように、「追加」を選択し、再度「Edge Hub」を選択します。
![Azure SAS on Edge Output](../assets/AzureSASonEdgeOutput.PNG)

結果のウィンドウで、`出力エイリアス`に「SummarizedDetections」という名前を付け、「保存」を選択します。

![Azure SAS on Edge Output Summarized](../assets/AzureSASonEdgeOutputSummarized.PNG)

これで、「AggregatedDetections」用と「SummarizedDetections」用の2つの出力が定義されているはずです。

![Azure SAS on Edge Outputs](../assets/AzureSASonEdgeOutputs.PNG)

新しく作成したジョブに戻り、「クエリ」を選択し、[DeepStreamAnalytics.sql](../services/AZURE_STREAMING_ANALYTICS/Edge/DeepStreamAnalytics.sql)の内容を含むようにクエリを編集し、クエリを保存します。


![Azure SAS on Edge Query](../assets/AzureSASonEdgeQuery.PNG)

次に、「サンプル入力のアップロード」を選択し、[SampleInput.json](../services/AZURE_STREAMING_ANALYTICS/Edge/SampleInput.json)の内容をアップロードします。


![Azure SAS on Edge Query Test](../assets/AzureSASonEdgeQueryTest.PNG)

OKを選択し、「Test query」を選択すると、以下のような結果が得られます（実際のテスト実行のサンプルデータを含む[DemoData.json](../services/AZURE_STREAMING_ANALYTICS/Edge/DemoData.json)を使用して、最後のステップを繰り返すこともできます）。

![Azure SAS on Edge Query Tested](../assets/AzureSASonEdgeQueryTested.PNG)

## Module 4.2 : A Brief Overview of the DeepStreamAnalytics SQL Query 

Azure Stream Analytics offers a SQL query language for performing transformations and computations over streams of events.  The [Stream Analytics Query Language Reference](https://docs.microsoft.com/en-us/stream-analytics-query/stream-analytics-query-language-reference?WT.mc_id=julyot-iva-pdecarlo) provides detailed information of the available syntax.

The DeepStreamAnalytics query works by first flattening the DeepStream message output by taking advantage of the [`REGEXMATCH`](https://docs.microsoft.com/en-us/stream-analytics-query/regexmatch-azure-stream-analytics?WT.mc_id=julyot-iva-pdecarlo) function.

Given the following example output from DeepStream where `objects` is formatted as [ *trackingId | bboxleft | bboxtop | bboxwidth | bboxheight | object* ]:

Azure Stream Analytics は、イベントのストリームに対して変換や計算を実行するための SQL クエリ言語を提供しています。[Stream Analytics Query Language Reference](https://docs.microsoft.com/en-us/stream-analytics-query/stream-analytics-query-language-reference?WT.mc_id=julyot-iva-pdecarlo)には、利用可能な構文の詳細情報が記載されています。

DeepStreamAnalyticsクエリは、[`REGEXMATCH`](https://docs.microsoft.com/en-us/stream-analytics-query/regexmatch-azure-stream-analytics?WT.mc_id=julyot-iva-pdecarlo)関数を利用してDeepStreamメッセージ出力を最初に平坦化することで動作します。

以下の例では、DeepStreamからの出力は、`オブジェクト`が [ trackingId | bboxleft | bboxtop | bboxwidth | bboxheight | object* ] としてフォーマットされています。

```
     {
      "version" : "4.0",
      "id" : 4348,
      "@timestamp" : "2020-04-29T10:15:22.439Z",
      "sensorId" : "Yard",
      "objects" : [
        "1|409|351|544|465|Car",
        "2|410|351|543|465|Car",
        "3|480|351|543|465|Person"
      ]
    }
```

最初のクエリでは、この出力が以下の形式に変換され、`FlattenedDetections`として参照されます。


| sensorId | object | @timestamp               | matches |
|----------|--------|--------------------------|---------|
| Yard     | Car    | 2020-04-29T10:15:22.439Z | 1       |
| Yard     | Car    | 2020-04-29T10:15:22.439Z | 1       |
| Yard     | Person | 2020-04-29T10:15:22.439Z | 1       |


次のステップでは、`matches`値を使用して重複しているものをフィルタリングします。このテーブルは `AggregatedDetections` としてエイリアスされています。



|count  | sensorId | object | @timestamp               |
|-------|----------|--------|--------------------------|
| 2     | Yard     | Car    | 2020-04-29T10:15:22.439Z |
| 1     | Yard     | Person | 2020-04-29T10:15:22.439Z |


最後に，平滑化関数は，同じ15秒間隔の`カウント`の平均を，`SummarizedDetections` として別名を持つテーブルにフロアリングします．この例のメッセージでは、前のクエリと同じ結果が得られますが、もし平均値に小数が含まれていた場合は、小数点以下の整数に切り捨てられることは明らかです。

この情報を使用して、より長い時間ウィンドウまたはより短い時間ウィンドウで結果を生成したり、平滑化関数で切り捨てるのではなく`真`の平均を報告したりするようにクエリを変更することができるはずです。

## Module 4.3 : Publishing the DeepStream Analytics Job 

Azure Stream Analytics on Edgeでは、Azure Storageを使用して、クエリをパブリックアクセス可能なZIPファイルにパッキングしてDeepStream Analyticsのジョブをホストしています。そのため、別のサブスクリプションに存在するジョブを参照することができます。たとえば、[デフォルトの配置テンプレートで指定されたジョブなど](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L240)。

新しいジョブを独自のサブスクリプションに公開するには、新しく作成したAzure Stream Analytics on Edgeジョブに戻り、「ストレージアカウントの設定」⇒「ストレージアカウントの追加」を選択します。ここで、[CameraTaggingModule用に設定した既存のストレージアカウント](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/docs/Module%202%20-%20Configure%20and%20Deploy%20Intelligent%20Video%20Analytics%20to%20IoT%20Edge%20Runtime%20on%20NVIDIA%20Jetson.md#module-24--configure-the-blob-storage-module-dependencies)を選択して、ジョブをホストするための新しいコンテナを作成するか、[モジュール2.4の関連手順](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/docs/Module%202%20-%20Configure%20and%20Deploy%20Intelligent%20Video%20Analytics%20to%20IoT%20Edge%20Runtime%20on%20NVIDIA%20Jetson.md#module-24--configure-the-blob-storage-module-dependencies)を繰り返して、まったく新しいストレージアカウントを作成して、適切に設定することができます。

CameraTaggingModuleとAzure Stream Analytics on Edgeジョブの両方に単一のストレージアカウントを使用するか、別々のストレージアカウントを使用してそれぞれにサービスを提供するかは、あなた次第です。

ストレージアカウントを適切に設定したら、「保存」を選択します。

![Azure SAS Storage Account](../assets/AzureSASstorage.PNG)

次に、新しく作成したAzure Stream Analytics on Edgeジョブに戻り、「発行」⇒「発行」を選択します。プロンプトが表示されるので、「はい」を選択して次に進みます。これで、以下のようにSASのURLが生成されます。次のステップでは、配置テンプレートでこのSAS URLを参照して、現在配置されているジョブを新しいジョブに更新します。

 ![Azure SAS Publish](../assets/AzureSASpublish.PNG)

## Module 4.4 : Deploying the DeepStream Analytics Job 

開発マシンでVisual Studio Codeを開き、モジュール2の間に開発マシンにクローンしたリポジトリのフォルダを開きます。新しく公開されたDeepStream Analyticsジョブを参照するように配置を再構成する必要があります。ここまでのところ、配置テンプレートは、外国のAzureサブスクリプションにあるDeepStream Analyticsを参照しています。

開始するには、`deployment-iothub/deployment.template.json`を開き、次のハイライトされたセクションを削除します。

 ![Azure SAS Prep](../assets/AzureSASprep.PNG)

これにより、配置テンプレートからDeepStreamAnalyticsモジュールのエントリが削除され、再作成できるようになります。

次に、(CTRL+SHIFT+P)でコマンドパレットを起動し、「Azure IoT Edge: IoT Edgeモジュールの追加」と入力してタスクを起動します。配置テンプレートファイルを選択してください」というプロンプトが表示されたら、`deployment-iothub/deployment.template.json`オプションを選択します。

![Azure SAS Select template](../assets/AzureSASselectTemplate.PNG)

次に、「Azure Stream Analytics」モジュールのテンプレートを選択します。

![Azure SAS Select module](../assets/AzureSASselectModule.PNG)

次に、モジュールに "DeepStreamAnalytics "という名前を付けます。この名前は非常に重要で、後で参照されます。

![Azure SAS Name module](../assets/AzureSASnameModule.PNG)

次に、新しく公開されたAzure Stream Analyticsのジョブを選択する必要があります。

![Azure SAS Select Job](../assets/AzureSASselectJob.PNG)

これらのオプションが完了すると、deployment.template.jsonは、新しくデプロイしたAzure Stream Analytics on EdgeジョブからSASジョブを引き出すために、DeepStreamAnalyticsモジュールを再適用します。ツールが考慮していないいくつかの追加変更を行う必要があります。

まず、Jetsonデバイス上で実行可能なARM互換イメージをプルするように更新する必要があります。次のイメージを参照するようにDeepStreamAnalyticsモジュールのイメージエントリを変更し、`mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.6-linux-arm32v7`というタグを付けて、示されているように変更します。

![Azure SAS update Image](../assets/AzureSASupdateImage.PNG)

次に、"SummarizedDetections "のみを送信するようにルートを更新する必要があります。

デフォルトでは、ツールはすべての出力を IoTHub に流すように設定します（"AggregatedDetections" と "SummarizedDetections"）。
`DeepStreamAnalyticsToIoTHub`のルートエントリを `"FROM /messages/modules/DeepStreamAnalytics/outputs/SummarizedDetections INTO $upstream",` に変更します。


![Azure SAS update Route](../assets/AzureSASupdateRoute.PNG)

You can validate the changes are correct by looking at the diff of changes to `deployment-iothub/deployment.template.json`, there will be number of entries added to `DeepStream Analytics: { "properties.desired": {` and it should be the only section in that file that shows any modifications.

変更が正しいかどうかは、`deployment-iothub/deployment.template.json`の変更のdiffを見ることで確認できます。`DeepStream Analytics: { "properties.desired": {`  に追加されるエントリの数が増え、そのファイルの中で唯一変更が表示されるセクションになるはずです。


![Azure SAS Git Diff](../assets/AzureSASgitDiff.PNG)

エントリが何であるか、そしてStream Analytics on Edgeジョブでどのように使用されているかを簡単に概観してみましょう。

* **ASAJobInfo** [required]は、ブロブストレージ内のzipパッケージを指すSASURLです。このzipパッケージには、DLL、ジョブ設定、定義、顧客コード（定義されている場合）を含むコンパイルされたASAジョブが含まれています。ASA モジュールはこのパッケージをダウンロードし、そこからジョブを開始します。これは、モジュールがストリーミングジョブを開始するために必要な唯一のプロパティです。
* **ASAJobResourceId** [optional] は、デプロイする ASA ジョブのリソース ID です。このプロパティは、最初のデプロイ後にジョブの更新があるかどうかを確認するために ASA サービスにコールバックするためにモジュールが使用します。
* **ASAJobEtag** [オプション] は、現在デプロイされているジョブのハッシュ ID です。これはモジュールが最初のデプロイ後にジョブの更新があるかどうかをチェックするためにも使用されます。
PublishTimestamp [optional]はジョブの公開時間です。これはストリーミングジョブがコンパイルされ、ブロブストレージに公開された時間を示すだけです。
これらのエントリを使用して、公開したStream AnalyticsジョブとEdge上で実行しているDeepStreamAnalyticsモジュールとの間で変更を同期することができるようになりました。

変更を行い、現在更新されている`deployment-iothub/deployment.template.json`を保存したら、`deployment-iothub`フォルダを展開し、`deployment.template.json`ファイルを右クリックして、「Generate IoT Edge Deployment Manifest（IoT Edge デプロイメント マニフェストの生成）」を選択します。これにより、そのディレクトリ内に「config」という名前の新しいフォルダが作成され、 `deployment.arm64v8.json`という名前の関連する配置が作成されます。`deployment.arm64v8.json` ファイルを右クリックして、「単一デバイス用の配置を作成」を選択します。

ドロップダウンが表示され、現在選択しているIoT Hubに登録されているすべてのデバイスが表示されます。Jetsonデバイスを表すデバイスを選択すると、ディプロイメントがデバイス上でアクティブになります（IoT Edgeランタイムがアクティブで、デバイスがインターネットに接続されている場合）。

## Module 4.5 : Configure and deploy Azure Time Series Insights with an IoT Hub event source 


[Azure Time Series Insights](https://docs.microsoft.com/en-us/azure/time-series-insights/?WT.mc_id=julyot-iva-pdecarlo)では、IoTデバイスから入ってくるデータを文脈的に可視化し、実用的なインサイトを特定して生成することができます。これは、「コールド」アーカイブ分析と、ほぼリアルタイムの「ウォーム」クエリ機能をサポートしています。また、時系列インサイトを[Azure Machine Learning](https://docs.microsoft.com/en-us/azure/machine-learning/?WT.mc_id=julyot-iva-pdecarlo), [Azure Databricks](https://docs.microsoft.com/en-us/azure/azure-databricks/?WT.mc_id=julyot-iva-pdecarlo), [Apache Spark](https://docs.microsoft.com/en-us/azure/hdinsight/spark/apache-spark-overview?WT.mc_id=julyot-iva-pdecarlo)などの高度なアナリティクスサービスと統合することも可能です。

このセクションでは、Jetsonデバイスによって生成されたオブジェクト検出データを可視化して操作するために、時系列インサイトのインスタンスを設定してデプロイします。コストを最小限に抑えるために、"Pay-As-You-Go "のデプロイメントをデモします。時系列インサイトの利用可能な価格オプションの詳細については、[こちら](https://azure.microsoft.com/en-us/pricing/details/time-series-insights/?WT.mc_id=julyot-iva-pdecarlo)をご覧ください。

開始するには、Azure Marketplaceに移動し、'Time Series Insights'を検索してください。

![Azure TSI Marketplace](../assets/TSIMarketplace.PNG)

"作成"を選択

![Azure TSI Create](../assets/TSICreate.PNG)

Review + Create セクションで、IoT Hubを含む同じ理由にデプロイしていることを確認します。PAYG」層が選択されていることを確認します。これにより、「Time Series Id」を設定するためのドロップダウンが下に作成されます。適切なタイムシリーズIDを選択することは非常に重要です。時系列IDを選択することは、データベースのパーティションキーを選択するようなものです。時系列IDは、Time Series Insights Preview環境を作成する際に必要であり、一度設定すると再設定できません。このセクションでは、`sensorId` と `iothub-connection-device-id` を使用するように設定し、データを整理された方法でパーティショニングできるようにします。これは、Stream Analyticsジョブが出力で`sensorId`値を生成するためです。`iothub-connection-device-id`は、IoT Edgeランタイムによって提供され、IoT Hubへのメッセージを生成したデバイスに対応します。以下の画像のようなオプションが設定されていることを確認してください。

![Azure TSI Config Part 1](../assets/TSIConfig1.PNG)

もう少し下にスクロールすると、「Cold」と「Warm」ストアのオプションを設定するセクションがあります。Cold」ストレージとして動作するためには新しいストレージアカウントを作成する必要がありますが、オプションで「Warm」ストア機能を使用するかどうかを決めることができます。下の画像は、これらのオプションの両方を設定したものです。

![Azure TSI Config Part 2](../assets/TSIConfig2.PNG)

次に、「次へ」を選択します。イベントソース」を選択して、TSI インスタンスにデータを提供するイベントソースを構成します。イベントソースの名前を入力し、既存の IoT Hub を選択します。`IoT Hub アクセスポリシー名`には、`iothubowner` ポリシーを選択します。 利用可能なポリシーの詳細については、利用可能な[ドキュメント](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-security?WT.mc_id=julyot-iva-pdecarlo)を参照してください。
Consumer Group」エリアのIoT Hubコンシューマーグループの設定セクションで、「Add」を選択し、図のように「TSI」という名前の新しいコンシューマーグループを作成します。コンシューマグループの詳細については、[こちらの記事](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-messages-read-builtin?WT.mc_id=julyot-iva-pdecarlo)を参照してください。最後に、"TIMESTAMP "領域で、"プロパティ名 "の値を、DeepStreamAnalyticsのクエリ出力のタイムスタンプフィールドに対応する`@timestamp`に設定します。

![Azure TSI Config Part 3](../assets/TSIConfig3.PNG)

次の画面で「Create」を選択して、TSIインスタンスのデプロイを開始します。
![Azure TSI Config Part 4](../assets/TSIConfig4.PNG)

配置が完了したら、TSI インスタンスに移動し、「Time Series Insights explorer URL」の隣にあるリンクをクリックします。

![Azure TSI Deployed](../assets/TSIDeployed.PNG)

デフォルトでは、`sensorId` `iothub-connection-device-id`で示される、割り当てられたイベントソースに現在データを生成しているすべてのデバイスが表示されます。

![Azure TSI Default Explorer](../assets/TSIDefaultExplorer.PNG)

左側の "Model "を選択して、インスタンスのデータモデルと階層を設定し始めます。

![Azure TSI Default Model](../assets/TSIDefaultModel.PNG)


まず、「Types」を選択し、「Upload JSON」を選択します。プロンプトが表示されたら、`../services/TIME_SERIES_INSIGHTS/Types`にある[`ObjectDetectionType.json`](../services/TIME_SERIES_INSIGHTS/Types/ObjectDetectionType.json)ファイルをアップロードし、「アップロード」を選択します。



![Azure TSI Types](../assets/TSITypes.PNG)

この ObjectDetectionType 定義を見ると、"Detections"、"Person"、および "Vehicle" をレポートする機能があることに気づくでしょう。これらの例で使用されているクエリ構文に注意して、要件を満たすように修正してください。これにより、TSI エクスプローラのダッシュボードでクエリ基準を満たすデータをモデル化することができます。

次に、"Hierarchies "を選択し、次に "Upload JSON "を選択します。プロンプトが表示されたら、`../services/TIME_SERIES_INSIGHTS/Hierarchies`にある[`Locations.json`](../services/TIME_SERIES_INSIGHTS/Hierarchies/Locations.json)ファイルをアップロードし、「アップロード」を選択します。

![Azure TSI Hierarchies](../assets/TSIHierarchies.PNG)

この定義により、TSI ダッシュボードに表示されたときに、デバイスを場所別に整理できるようになります。

次に、「インスタンス」を選択し、各デバイスについて「編集」を選択し、「タイプ」を「ObjectDetectionType」に設定し、デバイスの名前を好ましいものに変更します。

![Azure TSI Instances Part 1](../assets/TSIInstancePart1.PNG)

次に「インスタンス」を選択し、再度、各デバイスごとに「編集」を選択し、「インスタンスフィールド」を選択して「アドレス」に値を入力します。 これで、TSIダッシュボードでデバイスをアドレス値で整理することができるようになります。

![Azure TSI Instances Part 2](../assets/TSIInstancePart2.PNG)

すべてのデバイスについてこれを完了すると、以下のような結果が得られるはずです。

![Azure TSI Instances Part 3](../assets/TSIInstancePart3.PNG)

次に、「分析」を選択して、TSI Explorerのダッシュボードに戻ります。 これで、「場所」エントリを展開し、関心のある場所の値を選択して、そのデバイスから検出、人物、または車両のデータをプロットし始めることができます。

![Azure TSI Explorer Modified](../assets/TSIExplorerModified.PNG)

## Module 4.6 : Next Steps

DeepStreamAnalytics Azure Stream Analytics ジョブによって生成されるフィルタリングされたデータと時系列インサイトでモデル化されたデータとの関係を理解したところで、ユースケースに合わせてソリューションをさらにカスタマイズすることができます。 

以下のリソースをチェックして、利用可能な機能と機能についてさらに詳しく知ることを強くお勧めします。

[Azure Time Sereies Insights Documentation](https://docs.microsoft.com/en-us/azure/time-series-insights/time-series-insights-overview?WT.mc_id=julyot-iva-pdecarlo)

[![Azure Time Series Insights Video](../assets/TSIVideo.PNG)](https://www.youtube.com/watch?v=GaARrFfjoss)
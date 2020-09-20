## Module 5 : Visualizing Object Detection Data in Near Real-Time with PowerBI


Power BIは、Microsoftが提供するビジネス分析サービスです。これは、セルフサービスのビジネスインテリジェンス機能を備えたインタラクティブなビジュアライゼーションを提供し、エンドユーザーは、情報技術スタッフやデータベース管理者に頼ることなく、自分自身でレポートやダッシュボードを作成することができます。

このモジュールでは、クラウドベースのAzure Stream Analyticsジョブを使用して、Azure IoT Hubからオブジェクト検出テレメトリをPowerBIデータセットに転送する方法を説明します。これにより、検出が行われるたびに更新できるレポートを作成することができます。次に、PowerBIレポートをPublishして、ライブダッシュボードに変換します。そこから、自然言語を使ってデータを照会し、ほぼリアルタイムでデータと対話することができます。

このモジュールを完了するためには、アクティブなPowerBIアカウントを持っている必要があります。アカウントを作成する必要がある場合は、この[video](https://channel9.msdn.com/Blogs/BretStateham/Signing-up-for-Power-BI)でプロセスを説明します。

このモジュールのステップに沿って進みたい場合は、「[Visualizing Object Detection Data in Near Real-Time with PowerBI](https://www.youtube.com/watch?v=lhvPbNF9eb4)」と題したライブストリーム・プレゼンテーションを録画しました。

[![Visualizing Object Detection Data in Near Real-Time with PowerBI](../assets/LiveStream5.PNG)](https://www.youtube.com/watch?v=lhvPbNF9eb4)

## Module 5.1 : Forwarding telemetry from IoT Hub to PowerBI using a Cloud-Based Azure Stream Analytics Job

これらの手順を試みる前に、NVIDIA Jetsonデバイスが関連するAzure IoT Hubにライブデータを送信していることを確認してください。これは、Visual Studio CodeでAzure IoT Hub Extension（関連するIoT Hubに設定されている必要があります）を展開し、"Devices "を選択し、現在のデバイスを選択し、右クリックして "Start Monitoring Built-in Event Endpoint "を選択することで行うことができます。


![Monitor Built-in Event Endpoint VSCode](../assets/MonitorEndpoint.PNG)

約15秒後、`OUTPUT`ウィンドウにデータが表示されるようになります。


![Monitor Built-in Event Endpoint VSCode Output](../assets/MonitorEndpointOutput.PNG)

Once you have confirmed that data is flowing, we can now create the backend services that will be used to ingest this live data.

[Azure Stream Analytics enables you to take advantage of one of the leading business intelligence tools](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-power-bi-dashboard?WT.mc_id=julyot-iva-pdecarlo), Microsoft [Power BI](https://docs.microsoft.com/en-us/power-bi/fundamentals/power-bi-overview?WT.mc_id=julyot-iva-pdecarlo). In this section, you will learn how to configure Power BI as an output from a Azure Stream Analytics job that forwards data arriving into our IoT Hub.

To begin, we will create a PowerBI workspace to publish our dataset and reports to. Navigate to powerbi.microsoft.com and log in.  Next, select the "Workspace" icon then "Create a workplace":

データが流れていることが確認できたら、次はこのライブデータをインジェストするためのバックエンドサービスを作成します。

[Azure Stream Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-power-bi-dashboard?WT.mc_id=julyot-iva-pdecarlo),を利用することで、主要なビジネスインテリジェンスツールの1つであるMicrosoft [Power BI](https://docs.microsoft.com/en-us/power-bi/fundamentals/power-bi-overview?WT.mc_id=julyot-iva-pdecarlo)を活用することができます。このセクションでは、IoT Hubに到着したデータを転送するAzure Stream Analyticsジョブからの出力としてPower BIを構成する方法を学びます。

まず、データセットとレポートを公開するための PowerBI ワークスペースを作成します。powerbi.microsoft.comにアクセスし、ログインします。次に、「ワークスペース」アイコンを選択し、「ワークプレイスの作成」を選択します。


![Power BI Workspace](../assets/PowerBIWorkspace.PNG)

表示されるプロンプトで、ワークスペースに "Intelligent Video Analytics "という名前を付け、"Save "を選択します。

![Power BI Workspace Add](../assets/PowerBIWorkspaceAdd.PNG)

次に、IoTハブが生成したメッセージへのアクセスを許可するために、新しいコンシューマグループを作成します。 IoTハブでは、1つのコンシューマ・グループ内のリーダの数が制限されており（5つまで）、これにより、将来的なソリューションの変更がメッセージ処理能力に影響を与えないことを保証します。これを実現するには、IoT Hubインスタンスに移動して「Built-in Endpoints」を選択し、以下のように「sas」という名前の新しい「Consumer Group」を作成します。

![Azure IoT Hub Consumer Group](../assets/IoTHubConsumerGroup.PNG)

Azure Marketplaceに移動し、「Stream Analytics Job」を検索します。

![Azure SAS on Cloud Marketplace](../assets/AzureSASonCloudMarketplace.PNG)

Select "Create":

![Azure SAS on Cloud Create](../assets/AzureSASonCloudCreate.PNG)

Stream Analyticsジョブに名前を付け、元のIoT Hubと同じリージョンにデプロイされていることを確認し、ホスティング環境が「クラウド」に設定されていることを確認し、「Stream units」を「1」に設定し、「Create」を選択します。

![Deploy Azure SAS on Cloud](../assets/DeploySASonCloud.PNG)

新しく作成されたジョブに移動し、"Inputs "を選択します。ここでは、IoTハブからデータを転送する際に使用する入力エイリアスを設定します。 このステップでは名前の付け方が非常に重要で、クエリで使用されるエイリアスと一致している必要があります。 

Add stream input "を選択し、結果のドロップダウンで "IoT Hub "を選択し、"Input Alias "に "IoTHub-Input "という名前を付けます（このステップでは名前付けが非常に重要です！）。

![Azure SAS on Cloud Input](../assets/AzureSASonCloudInput.PNG)

次に、"Output "に移動します。ここでは、データをPowerBIシンクにプッシュするために使用する出力エイリアスを設定します。 追加 "を選択し、"Power BI "を選択し、結果として表示されるプロンプトでPowerBIサービスを承認します。 

![Azure SAS on Cloud PowerBI Auth](../assets/PowerBIAuth.PNG) 

結果として表示されるプロンプトで、"Output Alias "に "StreamAnalytics-Cloud-Output "という名前を付けます（このステップでは、名前を付けることは非常に重要です！）。 Powerbi.microsoft.comで新しいワークスペースを作成したときに使用した値（"Intelligent Video Analytics"）に "Group Workspace "を設定し、"Dataset name "を "Intelligent Video Analytics Dataset "に設定し、"Table name "を "Intelligent Video Analytics Table "に設定します。 最後に、認証モードを "User Token "に設定します。

![Azure SAS on Cloud Output](../assets/AzureSASonCloudOutput.PNG)


新しく作成したジョブに戻り、"Query "を選択し、[IoTHubToPowerBI.sql](../services/AZURE_STREAMING_ANALYTICS/Cloud/IoTHubToPowerBI.sql) の内容を含むようにQueryを編集し、Queryを保存します。

![Azure SAS on Cloud Query](../assets/AzureSASonCloudQuery.PNG)

データが IoT Hub に流れている限り、「クエリのテスト」を選択して、結果を生成することができます。 そうでない場合は、入力と出力のエイリアス名がクエリで指定されたものと一致していることを再確認してください。

![Azure SAS on Cloud Test](../assets/AzureSASonCloudTest.PNG)

次に、新しいStream Analyticsジョブの「概要」セクションに移動し、「開始」を選択してジョブの実行を開始します。

![Azure SAS on Cloud Start](../assets/AzureSASonCloudStart.PNG)

数分後、powerbi.microsoft.comのワークスペースの下に新しいデータセットが作成されているのがわかるはずです。

![PowerBI Dataset](../assets/PowerBIDataset.PNG)

## Module 5.2 : Import PowerBI Template and publish PowerBI Report 

次のステップでは、[Power BI Desktop application](https://powerbi.microsoft.com/en-us/desktop/)をインストールする必要があります（最新のWindows OSが必要です）。事前に提供されているテンプレートを使用して、オブジェクト検出遠隔測定を表示するためのダッシュボードを迅速に作成します。

まず、Power BI Desktopを開き、「ファイル」⇒「インポート」⇒「Power BIテンプレート」を選択します。


![PowerBI Import](../assets/PowerBIImport.PNG)

次に、このレポのソースフォルダに移動し、「services\POWER_BI\DeepStream+PowerBI.pbit」テンプレートファイルを開きます。

![PowerBI Import File](../assets/PowerBIImportFile.PNG)

PowerBIがデータソースに接続できないことを示すメッセージが表示される場合がありますが、"Transform data" => "Data Source Settings "を選択して、以前に公開されたデータセット（"Intelligent Video Analytics Dataset"）を選択することで、新しく公開されたデータセットにアタッチするようにテンプレートを設定することができます。 前のモジュールで提案された命名規則に従っていれば、データは問題なくインポートされ表示されるはずですが、そうでない場合は、事前に提供されたテンプレートのパラメータの一部を再設定する必要があるかもしれません。

![PowerBI Dataset](../assets/PowerBIDatasource.PNG)

テンプレートを公開するには、"Publish "を選択し、レポートを保存して "Intelligent Video Analytics Report "という名前を付け、"Intelligent Video Analytics "ワークスペースを公開すると、成功を示すプロンプトが表示されるはずです。

![PowerBI Publish Report](../assets/PowerBIPublishReport.PNG)

## Module 5.3 : Create a live PowerBI Dashboard in PowerBI Online 

これで、PowerBI オンラインで公開されたレポートのビューが表示されるはずですが、「Pin Live」ボタンを選択して、レポートをライブダッシュボードに固定します。 ダッシュボードに「Intelligent Video Analytics Dashboard」という名前を付けます。

![PowerBI Pin Live](../assets/PowerBIPinLive.PNG)

Intelligent Video Analytics" ワークスペースに移動し、"Dashboard" を選択すると、新たに固定されたライブ ダッシュボードが表示されますので、それを選択します。
![PowerBI Pin Live](../assets/PowerBINewDashboard.PNG)

「あなたのデータについて質問する」という項目があることに注目してください。
![PowerBI Ask Question](../assets/PowerBIAskQuestion.PNG)

Try some of these examples:
```
Max of count person in last minute by sensor id
MAX of count car in yard in last minute  
AVG count of car in Street in last minute    
LAST person by sensorid
Pie Chart count of person by sensorid in last day (@timestamp)
```

それぞれの例では、[オブジェクト]の値を自由に変更し、望ましい結果が得られたら、ビジュアルをダッシュボードに固定することができます。

![PowerBI Pin Visual](../assets/PowerBIPinViz.PNG)

望ましい結果が得られるまで、このプロセスを繰り返します。

![PowerBI Pinned](../assets/PowerBIPinned.PNG)

## Module 5.4 : Add a Streaming Data Tile to PowerBI Dashboard in PowerBI Online 

次に、「タイルの追加」⇒「カスタムストリーミングデータ」を選択して、ストリーミングデータのタイルを追加します。

![PowerBI Tile](../assets/PowerBITile.PNG)

Intelligent Video Analytics Dataset」を選択し、「Next」をクリックします。

![PowerBI Tile Dataset](../assets/PowerBITileDataset.PNG)

次の画面で「可視化の種類」を「折れ線グラフ」に設定し、「軸」「凡例」「値」を図のように設定し、「適用」を選択します。

![PowerBI Tile Viz](../assets/PowerBITileViz.PNG)

新しいビジュアライゼーションがDasbhoardに表示され、ほぼリアルタイム（デフォルトのStream Analytics on Edgeジョブを使用している場合は約15秒ごと）に検出テレメトリのプロットが開始されます。
![PowerBI Tile Viz name](../assets/PowerBITileVizName.PNG)

望ましい結果が得られるまで、このプロセスを繰り返します。

## Module 5.5 : Next Steps

おめでとうございます!  この時点で、オブジェクト検出データをPowerBIにレポートして可視化するための完全なエンドツーエンドのIntelligent Video Analyticsパイプラインが開発されました!  PowerBIダッシュボードに適用できるその他のテクニックに興味がある場合は、以下のリソースをチェックしてください。

* [PowerBI Documentation](https://docs.microsoft.com/en-us/power-bi/)
* [PowerBI Themes Gallery](https://community.powerbi.com/t5/Themes-Gallery/bd-p/ThemesGallery)
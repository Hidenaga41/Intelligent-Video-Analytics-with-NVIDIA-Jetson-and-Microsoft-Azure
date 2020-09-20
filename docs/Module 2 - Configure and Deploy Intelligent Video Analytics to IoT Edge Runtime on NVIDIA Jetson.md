## Module 2 : Configure and Deploy "Intelligent Video Analytics" to IoT Edge Runtime on NVIDIA Jetson

このセクションでは、NVIDIA Jetson デバイスに [IoT Edge Runtime](https://docs.microsoft.com/en-us/azure/iot-edge/about-iot-edge?WT.mc_id=julyot-iva-pdecarlo)をインストールして設定します。これには、関連する [IoT Edge Deployment for IoT Hub](../deployment-iothub/deployment.template.json) で定義されているモジュールをサポートするために、Azure サービスのコレクションをデプロイする必要があります。

デプロイを見ると、以下のモジュールが含まれていることがわかります。

| Module                    | Purpose                                                                                                                         | Backing Azure Service                                                    |
|---------------------------|---------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| edgeAgent                 | デバイスデプロイで定義されたモジュールのデプロイとアップタイムを確保するためにIoT Edgeで使用されるシステムモジュール                               | [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/?WT.mc_id=julyot-iva-pdecarlo) (認証とデプロイ構成を取得するためのもの) |
| edgeHub                   | モジュール間の通信とAzure IoT Hubへのメッセージバックを担当するシステムモジュール                                       | [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/?WT.mc_id=julyot-iva-pdecarlo) (デバイスからクラウドテレメトリへの取り込み)                   |
| NVIDIADeepStreamSDK       | DeepStreamワークロードを実行し、出力は要約のためにDeepStreamAnalyticsモジュールに転送するためのモジュール               | テレメトリはDeepStreamAnalyticsモジュールにルーティングされ（参照：[IoT Edge - Declare Routes](https://docs.microsoft.com/en-us/azure/iot-edge/module-composition#declare-routes?WT.mc_id=julyot-iva-pdecarlo)）、そこでフィルタリングされ、[Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/?WT.mc_id=julyot-iva-pdecarlo)に転送されます。                                                                    |
| CameraTaggingModule       | カスタム物体検出モデルのトレーニングに使用するために利用可能なRTSPソースから画像を取得するためのカスタムモジュール               | カスタム物体検出モデルのトレーニングに使用するためにキャプチャした画像をエクスポートするための [CustomVision.AI](https://www.customvision.ai/?WT.mc_id=julyot-iva-pdecarlo)             |
| azureblobstorageoniotedge |  Azure Storage Account へのデータのレプリケーションを提供するためのカスタムモジュール                                              | キャプチャした画像のレプリケーションと長期保存のための [Azure Storage Account](https://docs.microsoft.com/en-us/azure/storage/?WT.mc_id=julyot-iva-pdecarlo)   |
| DeepStreamAnalytics       | NVIDIADeepStreamSDKからのオブジェクト検出結果をまとめる「Stream Analytics on IoT Edge」モジュールを採用したカスタムモジュール | [Azure Stream Analytics on Edge](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-edge?WT.mc_id=julyot-iva-pdecarlo) ジョブの定義とAzureからの提供                                   |



ここでは、[Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/?WT.mc_id=julyot-iva-pdecarlo)と[Azure Storage Account](https://docs.microsoft.com/en-us/azure/storage/?WT.mc_id=julyot-iva-pdecarlo)をデプロイするだけです。これらのサービスの価格について気になる方は、以下にまとめてみました。

* [IoT Hubの価格](https://azure.microsoft.com/en-us/pricing/details/iot-hub/?WT.mc_id=julyot-iva-pdecarlo)
* [Azure Storage Account](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure?WT.mc_id=julyot-iva-pdecarlo)
* [Azure Stream Analytics on Edge の価格設定](https://azure.microsoft.com/en-us/pricing/details/stream-analytics/?WT.mc_id=julyot-iva-pdecarlo) (技術的には、エンドユーザーのサブスクリプションに含まれていないジョブを使用しているにもかかわらず、課金はDeepStreamAnalyticsモジュールを実行するデバイスごとに発生します。)

追加サービスである[CustomVision.AI](https://www.customvision.ai/?WT.mc_id=julyot-iva-pdecarlo)と[Azure Stream Analytics on Edge](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-edge?WT.mc_id=julyot-iva-pdecarlo)については、今後のセクションで説明しますので、現時点では必要ありません。

このモジュールのステップに沿って進みたい場合は、「[Configure and Deploy "Intelligent Video Analytics" to IoT Edge Runtime on NVIDIA Jetson](https://www.youtube.com/watch?v=RKwwP4XsZdw)」と題した、以下のステップを詳細に説明するライブストリームプレゼンテーションを録画しました。

[![Configure and Deploy "Intelligent Video Analytics" to IoT Edge Runtime on NVIDIA Jetson](../assets/LiveStream2.PNG)](https://www.youtube.com/watch?v=RKwwP4XsZdw)


### Module 2.1 : Install IoT Edge onto the Jetson  Device

IoT Edgeをインストールする前に、いくつかのユーティリティをNvidia Jetsonデバイスにインストールする必要があります。

```
sudo apt-get install -y curl nano 
```

ARM64 builds of IoT Edge that are compatible with NVIDIA Jetson Hardware are provided beginning in the [1.0.8 release tag](https://github.com/Azure/azure-iotedge/releases/tag/1.0.8) of IoT Edge.  To install the latest release of IoT Edge, run the following from a terminal on your Nvidia Jetson device or consult the [official documentation](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux?WT.mc_id=julyot-iva-pdecarlo):

NVIDIA Jetson Hardwareと互換性のあるIoT EdgeのARM64ビルドは、IoT Edgeの[1.0.8 release tag](https://github.com/Azure/azure-iotedge/releases/tag/1.0.8)から提供されています。IoT Edgeの最新リリースをインストールするには、Nvidia Jetsonデバイスのターミナルから以下を実行するか、[公式ドキュメント](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux?WT.mc_id=julyot-iva-pdecarlo)を参照してください。

```
# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

# Install the IoT Edge repository configuration
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list

# Copy the generated list
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/

# Install the Microsoft GPG public key
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/

# Perform apt update
sudo apt-get update

# Install IoT Edge and the Security Daemon
sudo apt-get install iotedge

```

インストール後、デバイスの設定を更新する必要があることを示す以下のメッセージが表示されます。

```
===============================================================================

                              Azure IoT Edge

  IMPORTANT: Please update the configuration file located at:

    /etc/iotedge/config.yaml

  with your device's provisioning information. You will need to restart the
  'iotedge' service for these changes to take effect.

  To restart the 'iotedge' service, use:

    'systemctl restart iotedge'

    - OR -

    /etc/init.d/iotedge restart

  These commands may need to be run with sudo depending on your environment.

===============================================================================
```

### Module 2.2 : Provision the IoT Edge Runtime on the Jetson Device

このセクションでは、Jetson ハードウェアを IoT Edge デバイスとして手動でプロビジョニングします。このためには、アクティブな IoT Hub をデプロイして、新しい IoT Edge デバイスを登録し、そこから IoT Hub インスタンスへの安全な認証を可能にするデバイス接続文字列を取得する必要があります。

新しいIoT Hubを作成し、IoT Edgeデバイスを登録し、[AzureポータルでIoT Edgeデバイスを登録](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-register-device-portal?WT.mc_id=julyot-iva-pdecarlo)するためのドキュメントに従うか、[Azure-CLIでIoT Edgeデバイスを登録](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-register-device-cli?WT.mc_id=julyot-iva-pdecarlo)することで、これを達成するために必要なデバイス接続文字列を取得することができます。

接続文字列を取得したら、IoT Edgeのデバイス設定ファイルを開きます。

```
sudo nano /etc/iotedge/config.yaml
```

ファイルのプロビジョニングセクションを見つけ、手動プロビジョニングモードのコメントを外します。`device_connection_string` の値をIoT Edgeデバイスからの接続文字列で更新します。

```
provisioning:
  source: "manual"
  device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
  
# provisioning: 
#   source: "dps"
#   global_endpoint: "https://global.azure-devices-provisioning.net"
#   scope_id: "{scope_id}"
#   registration_id: "{registration_id}"

```

`device_connection_string`の値を更新したら、以下のコマンドでiotedgeサービスを再起動してください。

```
sudo service iotedge restart
```

以下のコマンドを使ってIoT Edge Daemonの状態を確認することができます。

```
systemctl status iotedge
```

以下のコマンドで，デーモンのログを調べます。
```
journalctl -u iotedge --no-pager --no-full
```

以下のコマンドで，実行中のモジュールをリストアップします。
```
sudo iotedge list
```

以下のコマンドで，IoT Edge Runtimeが設定され、実行されていることを確認します．

```
sudo service iotedge status
```

正常に設定されたデバイスは、以下のような出力が確認されます。エラーがある場合は、設定が適切に設定されていることを再確認してください。

```
● iotedge.service - Azure IoT Edge daemon
   Loaded: loaded (/lib/systemd/system/iotedge.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2020-06-08 13:04:44 CDT; 15s ago
     Docs: man:iotedged(8)
 Main PID: 9029 (iotedged)
    Tasks: 11 (limit: 4183)
   CGroup: /system.slice/iotedge.service
           └─9029 /usr/bin/iotedged -c /etc/iotedge/config.yaml
```

IoT Edgeランタイムは、edgeAgentとedgeHubシステムモジュールのプルダウンを開始します。 これらのモジュールは、追加モジュールを含むデプロイ構成を提供するまで、デフォルトで実行されます。


### Module 2.3 : Prepare the Jetson Device to use the "Intelligent Video Analytics" sample configurations

このモジュールでは、このリポジトリに含まれるサンプル設定を Jetson デバイスにミラーリングします。 そのためには、これらの設定で参照されている非常に特殊なパスを利用する必要があります。 

まず、Jetson デバイス上に設定を保存するためのディレクトリを作成します。

```
sudo mkdir -p /data/misc/storage
```

次に、`/data`ディレクトリとすべてのサブディレクトリに、非特権ユーザアカウントからアクセスできるように設定します。

```
sudo chmod -R 777 /data
```

次に、`/data/misc/storage`に移動します。
```
cd /data/misc/storage
```

そして，このリポジトリを，作成したディレクトリにクローンします。
```
git clone https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure.git
```

次に、`iotedge`のユーザアカウントにX11ソケットのローカル権限を付与することで、コンテナからX11 WindowサーバにアクセスできるようにJetson OSを設定する必要があります。

```
xhost local:iotedge
```

これにより、現在のログインセッションの権限が有効になりますが、再起動時には持続しません。で編集できるように `/etc/profile` を開いて設定を永続化します。

```
sudo nano /etc/profile
```

そして、そのファイルの一番上に次のテキストを追加します。
```
xhost local:iotedge
```

その後の再起動時に、`iotedge`ユーザーはホストのX11ソケットを使用してグラフィカルユーザーインターフェースを生成できるようになります。 これにより、IoT Edgeモジュールとして実行中(つまりコンテナとして実行中)にDeepStreamSDKモジュールのバウンディングボックス検出を表示できるようになります。

潜在的な問題の診断を容易にするために、ユーザーアカウントからのdockerサービスへのアクセスを有効にする必要があります。 これは、以下の方法で実現できます。

```
sudo usermod -aG docker $USER
```

以降のログインセッションでは、`sudo`を前置しなくても `docker` コマンドを起動できるようになりました。

これで、「Intelligent Video Analytics」のサンプル設定を使用するためのJetson Deviceの準備が完了しました。 次に、適切な前提条件であるAzure Storage Accountと、Blob Storage Module(azureblobstorageoniotedge)に必要な設定を行います。

### Module 2.4 : Configure the Blob Storage Module Dependencies

このステップでは、CameraTaggingModuleと組み合わせて使用する[IoT Edge Blob Storage Module](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-blob?WT.mc_id=julyot-iva-pdecarlo)を設定し、画像キャプチャをローカルに保存し、クラウドにレプリケートします。技術的には、このモジュールはオプションであり、CameraTaggingModuleはそれなしで直接クラウドまたはCustomVision.AIに画像をアップロードすることができますが、アウトバウンドのインターネットアクセスを必要とせずに画像をキャプチャして保存することができるエンドユーザーのためのより堅牢なソリューションを提供します。CameraTaggingModuleとそのサポート機能については、この[詳細な記事](https://dev.to/azure/introduction-to-the-azure-iot-edge-camera-tagging-module-di8)を参照してください。

このモジュールは、Visual StudioCodeを使用する必要があり、できればJetsonデバイスではない開発マシンで実行してください。このリポジトリを開発マシンにクローンすることから始めてください。

```
git clone https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure.git
```

次に、Visual Studio Codeを開き、「ファイル⇒フォルダを開く」を選択し、新しく作成された「Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure」フォルダに移動して選択します。

新しく開いたプロジェクトの中で、`deployment-iothub`フォルダ内に.envという名前のファイルを作成し、以下の内容を指定します。

```
CONTAINER_REGISTRY_NAME=
LOCAL_STORAGE_ACCOUNT_KEY=
LOCAL_STORAGE_ACCOUNT_NAME=camerataggingmodulelocal
DESTINATION_STORAGE_NAME=camerataggingmodulecloud
CLOUD_STORAGE_CONNECTION_STRING=
```

このファイルには、deployment.template.jsonの値を置き換えるために使用されるキー/値が格納され、動作するデプロイメント マニフェストが作成されます。deployment.template.json のこれらのエントリには '$' 記号が付けられていることに気づくでしょう。 これは、デプロイメント マニフェストの生成中に置き換えるためのトークンとしてマークされます。

今のところ、`CONTAINER_REGISTRY_NAME`はプライベートリポジトリからコンテナイメージを取得する場合にのみ必要なので、省略します。 このデプロイメントのモジュールはすべて公開されているので、現時点では必要ありません。

[GeneratePlus](https://generate.plus/en/base64)にアクセスして `LOCAL_STORAGE_ACCOUNT_KEY` の値を生成します。 これは、ローカル・ブロブ・ストレージ・インスタンスへの安全な接続を構成するために使用される、ランダムなbase64エンコードされた文字列を生成します。 結果全体を指定したい場合は、2つの等号(*==*)で終わるようにしてください。

`LOCAL_STORAGE_ACCOUNT_NAME` はそのままにしておくのがベストですが、名前を変更しても構いません。このフィールドには小文字と数字のみを含めることができ、名前は3文字から24文字の間でなければなりません。

`DESTINATION_STORAGE_NAME` は、Azureクラウドに存在すると仮定したブロブストレージコンテナから提供されます。 このコンテナは、以下の手順を実行することで作成できます。

Azureマーケットプレイスに移動して「blob」を検索し、ストレージアカウント - blob、ファイル、テーブル、キューを選択します。

![Storage Marketplace](../assets/AzureStorageMarkeplace.PNG)

以下のような設定を使用してストレージアカウントを作成します（注意：ストレージアカウント名はグローバルに一意でなければなりません）。

![Storage Create](../assets/AzureStorageCreate.PNG)

レビュー + 作成 => 「作成」を選択して、新しいストレージアカウントリソースを展開します。

新しくデプロイしたストレージアカウントに移動し、`コンテナ`を選択します。
![Storage Overview](../assets/AzureStorageOverview.PNG)

以下のように「camerataggingmodulecloud」という名前の新しいストレージコンテナを作成します(.envの値と一致するので、名前は重要です)。

![New Container](../assets/AzureStorageContainerCreate.PNG)

CLOUD_STORAGE_CONNECTION_STRING は、新しく作成したストレージアカウントにアクセスして **Settings** => **Access Keys** を選択することで取得できます。 **接続文字列**の内容全体をコピーし、これを値として指定します。

![Obtain Connection String](../assets/AzureStorageConnectionString.PNG)

完成した.envファイルは以下のようになっているはずです。
```
CONTAINER_REGISTRY_NAME=
LOCAL_STORAGE_ACCOUNT_KEY=9LkgJa1ApIsISmuUHwonxg==
LOCAL_STORAGE_ACCOUNT_NAME=camerataggingmodulelocal
DESTINATION_STORAGE_NAME=camerataggingmodulecloud
CLOUD_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=camerataggingmodulestore;AccountKey=00000000000000000000000000000000000000000000000000000000000000000000000000000000000000==;EndpointSuffix=core.windows.net
```

これで、`deployment-iothub`で指定したサンプルデプロイメントを作成して適用する準備が整いました。

### Module 2.5 : Generate and Apply the IoT Hub based deployment configuration 

これで、前提となるサービス、セットアップ、設定がすべて完了したので、Jetsonデバイス上でサンプルのIntelligent Video Analyticsパイプラインを実行するためのデプロイメントを作成する準備が整いました。 次のステップは、Visual Studio Codeで実行します。

前のセクションでは、Blob Storage Moduleが必要とする設定パラメータをサポートするために、.envファイルを作成しました。 この.envファイルは `deployment-iothub` フォルダにあるはずです。 先に進む前に、適切なパラメータを提供し、.envファイルが存在することを確認してください。

次に、arm64v8プラットフォームをターゲットとするようにプロジェクトを設定します。これを行うには、(CTRL+SHIFT+P)でコマンドパレットを起動し、以下のタスクを検索します。

```
Azure IoT Edge: Set Default Target Platform for Edge Solution
```

「Azure IoT Edge: エッジソリューションのデフォルトターゲットプラットフォームの設定」タスクを選択すると、ドロップダウンが表示され、利用可能なプラットフォームがすべて表示されます。リストから`arm64v8`を選択します。これにより、プロジェクトに追加されたモジュールやソースからビルドされたモジュールが、Jetson アーキテクチャをターゲットにしていることが確認できます。

注: 上記のタスクを検索しても結果が表示されない場合は、[Azure IoT Tools Extension](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)がインストールされていることを確認してください。

次に、ホスト上で実行しているX11サーバーとの通信を可能にするために、NVIDIADeepStreamSDKモジュールの`DISPLAY`環境変数を設定する必要があります（すなわち、コンテナからGUIベースのアプリケーションを実行できるようにします）。ほとんどのインスタンスでは、`DISPLAY`環境変数は:1に設定され、これはホストに接続されているかどうかにかかわらず、参照される物理ディスプレイに対応しています。実際の値を取得するには、Jetsonデバイスのターミナルで以下のコマンドを実行します。

```
echo $DISPLAY
```

`DISPLAY` 値を取得したら、開発マシンで `deployment-iothub\deployment.template.json` フォルダを開き、Jetson Device で前のコマンドを実行して得られた結果と一致するように [`DISPLAY` 変数](https://github.com/toolboc/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/blob/master/deployment-iothub/deployment.template.json#L93)の値を更新します。この値が不一致の場合、NVIDIADeepStreamSDKのログにEGLシンクを作成できないというエラーが表示されることがあります。修正を行う場合は、必ずこのファイルを保存してください。

次に、(CTRL+SHIFT+P)で再度コマンドパレットを起動し、今度は以下のコマンドを検索します。

```
Azure IoT Hub: Select IoT Hub
```

Azure IoT Hub」を選択します。IoT Hubを選択」タスクを選択し、プロンプトに従って、Jetsonデバイス上のIoT Edgeランタイムの登録と設定に使用したIoT Hubに接続します。これは、Visual Studio CodeインスタンスをMicrosoft Azureで認証したことがない場合に必要になる場合があります。

適切なIoTハブを選択したら、`deployment-iothub`フォルダを展開して`deployment.template.json`ファイルを右クリックし、「Generate IoT Edge Deployment Manifest」を選択します。これにより、そのディレクトリ内に「config」という名前の新しいフォルダが作成され、 `deployment.arm64v8.json`という名前の関連する配置が生成されます。`deployment.arm64v8.json` ファイルを右クリックして、「単一デバイス用の配置を作成」を選択します。

ドロップダウンが表示され、現在選択しているIoT Hubに登録されているすべてのデバイスが表示されます。Jetsonデバイスを表すデバイスを選択すると、デバイス上で配置がアクティブになります（IoT Edgeランタイムがアクティブで、デバイスがインターネットに接続されている場合）。

デプロイで指定したイメージがデバイスにプルダウンされるまでに時間がかかる場合があります。すべてのイメージがプルダウンされているかどうかは、以下の方法で確認できます。


```
sudo docker images
```

完了したデプロイメントでは、最終的に以下のような結果が表示されるはずです。

```
REPOSITORY                                              TAG                   IMAGE ID            CREATED             SIZE
mcr.microsoft.com/azureiotedge-hub                      1.0                   9b62dd5f824e        7 days ago          237MB
mcr.microsoft.com/azureiotedge-agent                    1.0                   ae9bfb3081c5        7 days ago          219MB
nvcr.io/nvidia/deepstream-l4t                           5.0-dp-20.04-iot      7b4457646f87        5 weeks ago         2.16GB
toolboc/camerataggingmodule                             latest                704e9e0ce6dc        6 weeks ago         666MB
mcr.microsoft.com/azure-stream-analytics/azureiotedge   1.0.6-linux-arm32v7   bb2d6fbc5a3b        4 months ago        566MB
mcr.microsoft.com/azure-blob-storage                    latest                76f2e7849a91        11 months ago       203MB
```

デプロイが完了したことを確認したら、次はニーズに合わせてソリューションを修正することができます。これについては次のセクションで説明します。

### Module 2.6 : Customizing the Sample Deployment 

このセクションは、Jetson Device上でどのようにビデオ入力を処理するかに依存しますので、少し自由度の高いものになっています。

変更を行う前に、[DeepStream Documentation for Configuration Groups](http://aka.ms/DeepStreamDevGuide)のドキュメントを参照することを強くお勧めします。

変更されていないサンプルデプロイメントは、`/data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs`にあるDeepStream設定を参照しています。このディレクトリ内には、DeepStreamの設定例がいくつかあります。

| DeepStream Sample Configuration Name | Description                                                                                                                                                                                                |
|--------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DSConfig-CustomVisionAI.txt          | Employs an example object detection model created with CustomVision.AI that is located in /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/CUSTOM_VISION_AI  |
| DSConfig-YoloV3.txt                  | Employs an example object detection model based on [YoloV3](https://pjreddie.com/darknet/yolo/)                                                                                                            |
| DSConfig-YoloV3Tiny.txt              | Employs an example object detection model based on [YoloV3Tiny](https://pjreddie.com/darknet/yolo)   

これらの各例は、デフォルトでは、[Big Buck Bunnyの公開されているRTSPストリーム](https://www.wowza.com/html/mobile.html)からの単一のビデオ入力を処理するように設定されています。これは、インターネット上で唯一信頼性が高く、公開されているRTSPストリームであることと、既存の例を変更して、例えばIP対応のセキュリティカメラなどのカスタムRTSPエンドポイントを指すようにするのが非常に簡単になるようにするために部分的に設定しています。

配置でアクティブなDeepStream設定を変更するには、`depository.template.json`を修正して、`NVIDIADeepStreamSDK`モジュールの`ENTRYPOINT`仕様内で別の設定ファイルを指定し、モジュール2.5の手順を繰り返して、修正した配置を再生成して適用します。YoloV3*構成を使用する場合、モジュール3で説明する追加の依存関係を導入する必要があることに注意してください。

今回適用したデフォルトの配置では、DeepStreamのコンフィグレーションであるDSConfig-CustomVisionAI.txtをJetsonデバイス上で以下のように変更することができます。

```
nano /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs/DSConfig-CustomVisionAI.txt
```

この設定を編集した後、NVIDIADeepStreamSDKモジュールを再起動してテストしてください。
```
docker restart NVIDIADeepStreamSDK
```

ログを監視するには
```
iotedge logs NVIDIADeepStreamSDK
```

OR

```
docker logs -f NVIDIADeepStreamSDK
```

入力ソースのそれぞれについて、msgconv_config.txt のエントリを指定するには、以下のように変更します。
```
nano /data/misc/storage/Intelligent-Video-Analytics-with-NVIDIA-Jetson-and-Microsoft-Azure/services/DEEPSTREAM/configs/msgconv_config.txt
```

このファイルは、Azure IoT Hubへのテレメトリを生成するために使用され、特定のオブジェクト検出がどのビデオ入力/カメラから発信されたかを指定します。

最後の注意点として、複数のビデオソースを使用するようにDeepStream構成を変更している場合、最適なパフォーマンスを得るために使用するビデオソースの数と等しくなるように、`[streammux]` `batch-size`プロパティを変更する必要があります。 たとえば、4つの入力RTSPストリームを使用するようにDeepStream構成を変更した場合、変更したDeepStream構成で`[streammux]` `batch-size` = 4を設定します。

希望する入力からビデオソースを取得するように設定を変更したら、次はCustomVision.AIからカスタムオブジェクト検出モデルを作成して展開する方法と、YOLOV3*設定を使用してアカデミックグレードモデルの使用法を探る準備ができました。
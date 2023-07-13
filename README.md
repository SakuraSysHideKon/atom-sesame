# ATOM-SESAME
## M5Stack ATOM Lite を使って セサミスマートロックを操作。
### 目的
インターフォン越しに来客を確認してスマートロックを解除したい場合に、スマホ取り出し – ロック解除 – アプリを開く – スマートロック解除と、結構な手間がかかるので玄関まで走って手動で解除というのが定番でした。
インターフォンの近くにスマートロック操作用の施解錠スイッチがあれば、ポチッとするだけで解錠・施錠ができる。
まるで、オートロックのマンションのような仕組みが、自宅で安価に構築したい。
### プログラム説明
1. ArukasSmartKeys - ArukasSmartKeys.ino       ATOM Liteからセサミスマートロックを操作するスケッチ。
2. GetChipInfomation - GetChipInfomation.ino   ATOM LiteデバイスのMACアドレス等をシリアルモニタに表示するスケッチ。
### プログラム仕様
#### WiFi環境の設定について。
1. WiFi環境の設定については、スマホアプリ「EspTouch」を使い、WiFi経由で外部から設定できるものとします。
1. ボタン5秒以上長押しで、WiFi環境再設定モードに移行します。
1. WiFi環境再設定モードではLED赤点滅(1秒間隔)します。
1. WiFi環境設定は「EspTouch」対応のためSmartConfigにて内部設定します。
1. WiFi正常接続時はLED青点灯し、接続不可時は黄色点灯します。
1. WiFi接続状態は随時チェックし切断したら再接続をおこなう。
#### SESAME Web API APIキーについて。
1. APIキーはATOM Liteデバイスそれぞに個別のAPIキーを持つものとする。
1. ATOM Liteの標準MACアドレスを使ってAPIキーを紐付けします。
1. ATOM Liteの標準MACアドレスが、APIキー紐付けテーブルに設定されていない場合は、指定された既定APIキーを使うものとします。
#### SESAMEスマートロックの状態チェックについて。
1. SESAMEスマートロックの状態チェックは、Tickerタイマーにより6分に1回おこなう。(1時間に10回)
1. SESAMEスマートロックの状態が解錠されていればLEDを緑に点灯する。(セサミアプリに準ずる)
1. SESAMEスマートロックの状態が施錠されていればLEDを赤に点灯する。(セサミアプリに準ずる)
1. SESAMEスマートロックの状態チェック停止時間を指定できるようにする。
また、指定時間は日をまたいで指定できるようにする。(22時から6時等)
### 使用ライブラリ
使用ライブラリについては記述漏れもあるかとおもいますので、随時必要なライブラリを追加してください。
1. M5ATOM
1. NTPClient
1. ArduinoJson
1. arduino-AES_CMAC
1. Crypto
1. base64_encode
### WiFi設定
WiFi設定はESPRESSIF社の「ESP-TOUCH」を使って設定します。
### セサミWeb API利用のためにはAPI Keyが必要
下記ページににて、Web API利用のためのAPI Keyを取得する必要があります。
API Keyを取得するためには、スマートロックデバイスのUUIDとSecret Keyが必要です。

[Sesame Web API API Key 取得](https://partners.candyhouse.co/)
### ATOM LiteのMACアドレスにAPI Keyを紐付け
複数台のATOM Liteデバイス個々にAPI Keyを紐付けできます。
この機能により、複数台のデバイスを同じスケッチで動作させることができます。

ATOM LiteデバイスのMACアドレスを調べるには、「GetChipInformation.ino」をATOM Liteに書き込むとシリアルモニタにチップ情報が表示されます。
「MAC Adress:」に続く12桁の英数字がMACアドレスとなります。
### 注意事項
#### setup()の「おまじない」は、絶対消さないでください。
ATOM LiteはWiFiモジュールを使い高負荷なループの処理を行うと、M5.Btn.wasPressed()が頻繁にTrueになり、ボタンを押していないにも関わらず、ボタンが連打されるというオカルトチックな動作をします。
これは、デバッグ時などシリアルモニタに出力しながら処理をしていると発現しないのですが、デバッグ用のコードを除去した途端に高負荷になり、おばけ連打が発動します。
私のATOM Liteでは、このおまじないで何台ものATOM Liteを手懐けることに成功しています。
#### SESAME Web API が反応しない時がある。
Web API に対してPOSTでリクエストを送信するときに、スマートロックのシークレットキーをAESCMACにて暗号化して送信しますが、この暗号化のときにUTCのタイムスタンプ(秒)を使って暗号化しているのですが、どうもWeb API側で復号化に失敗しているようなのです。

この現象は、2台あるATOM Liteデバイスのどちらか一方で起きると、API Keyが違う他方のデバイスも同じ現象が発生するので、ロジカルな問題であることだけは確かなようです。

NGになった元の秒数はどれも暗号化キーが変わる時間までが、タイトな場合が多いようです。
(サンプル抽出数が少ないので絶対原因とは言い切れませんが。)
NTPClientライブラリで取得するタイムスタンプの正確性も関係してきますし、WiFiの通信速度やAPI側のタイムスタンプの正確さも加味されるので、ある程度は仕方がないと諦めるしかないようですね。

NGになったアクセスも、使用回数にカウントされますので注意が必要です。

ボタンを押しても反応がないから「何度も繰り返して押す」を繰り返すと、1ヶ月の上限使用回数を超えてしまい料金が発生する可能性があるので注意しましょう。

暗号化ライブラリが完璧かどうかも保証の限りではありませんし、真相は闇の中です。
#### 免責
1. 当モジュール及びサンプルプログラムは、技術検証を目的として作成されていますので、負荷試験を始めとする製品化を行うための基準を満たすための試験を一切行っておりませんので、予期せぬ結果を招く場合があります。その場合であっても、上記免責事項により有限会社さくらシステム及び製作者はその責を一切負いません。
2. 住居の安全を守る鍵のネタ記事なので、もしこの記事を参考にして、ハッキング等の被害により鍵を操作されて盗難や強盗にあったとしても、有限会社さくらシステム及び製作者はその責を一切負いません。
3. この記事は、あくまで技術検証のための記事であり、スマートロックを始めとするいかなる設備について利用方法を推奨・保証するものではありません。
4. また、バグ等でいかなる損害を被っても当方は一切の責任負いません。
### 動作確認
- OS：Windows10 Pro
- 開発環境： Arduino IDE バージョン：2.1.0
- 動作実行機器： M5Stack ATOM Lite
### 参照
[M5Stack ATOM Lite を使って セサミスマートロックを操作してみました。](https://sakura-system.com/?p=3497)

[SESAME WEB API](https://doc.candyhouse.co/ja/SesameAPI)

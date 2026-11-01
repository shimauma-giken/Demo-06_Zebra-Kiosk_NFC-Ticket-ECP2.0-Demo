# Zebra KC401 向け NFC VAS デモ 実行ガイド

UPDATE:2026/11/01
Source Info: https://techdocs.zebra.com/nfc-vas/latest/guide/demo/

<br/>

本ドキュメントは、Zebra の対話型 Android キオスクコンピュータ **KC401** において、**NFC VAS SDK デモアプリケーション（NFC Ticket Demo）** をセットアップし、Apple Wallet（Apple VAS）、Google Wallet（Google Smart Tap）、Samsung Wallet のデジタルパスおよび各種 NFC タグの読み取り検証を行うための手順書です。

---

## 1. 概要 (Overview)

**NFC Ticket Demo** アプリケーションは、ECP 1.0/ECP 2.0 に対応した Zebra デバイス上で、以下のデジタルウォレットパスや NFC タグの読み取りを実演・検証するツールです。

* **Apple Wallet**（Apple VAS / ECP 1.0 & ECP 2.0）
* **Google Wallet**（Google Smart Tap）
* **Samsung Wallet** パス
* **一般 NFC タグ**（NfcA, NfcB, NfcF / FeliCa, NfcV, IsoDep, Ndef）

> 💡 **KC401 の特長（Tap-on-Glass）**:  
> KC401 はディスプレイ表面に NFC アンテナが統合された **Tap-on-Glass** 方式を採用しています。一般的なハンドヘルド端末のように背面アンテナを探す必要がなく、**画面の表側**にスマートフォンを直接かざすだけで直感的な読み取りが可能です。

---

## 2. 前提条件と必要機器 (Requirements)

### 必要なハードウェア
1. **Zebra KC401 キオスクコンピュータ**（Android 15 以上搭載、NFC / VAS 対応）
   * 電源供給：PoE+（802.3at / 802.3bt Class 4）または DC 電源アダプタ
   * ネットワーク接続（Wi-Fi 7 または 有線LAN/Ethernet）
  
2. **検証用スマートフォン**（以下のいずれか、または両方）:
   * **iPhone / Apple Watch**: 
    - Apple Wallet
    - Apple VAS 対応端末
   * **Android スマートフォン / スマートウォッチ**: 
   　- Google Wallet（Google Smart Tap 対応）
   　- Samsung Wallet 対応端末

3. **PC（ホストコンピュータ）**:
   * KC401 と USB-C ケーブルで接続し、APK ファイル転送や ADB 操作を行うために使用

4. **（オプション）各種 NFC / FeliCa タグ**:
   * Mifare、FeliCa、ISO 15693 などの物理カード・タグ検証用

### 必要なソフトウェア・ファイル
* **Zebra VAS SDK パッケージ**（Zebra Support Portal よりダウンロード）
  * `ZebraVasService-x.x.x.apk`（VAS 通信サービス用バックグラウンドアプリ）
  * `ZebraVasDemoApp-x.x.x.apk`（デモ UI アプリケーション）

---

## 3. 検証用デジタルパスの準備 (Store VAS Pass)

スマートフォンのウォレットアプリ（Apple Wallet / Google Wallet）に、テスト用 Zebra VAS パスを登録します。

### Apple Wallet（iPhone）への登録手順
1. iPhone をインターネットに接続します。
2. ホーム画面右上から下にスワイプして「コントロールセンター」を開き、**コードスキャナー**（カメラ）をタップします。
3. 開発・検証用パス発行ポータルの QR コード（または TechDocs / 社内ポータル記載の登録用 QR コード）をスキャンします。<br/>
   <img height="250" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/qrcode.jpg">

4. パス発行用 URL がブラウザで開き、データベース設定画面に接続されます。
5. 画面の指示に従い、必要な情報（氏名、メールアドレスなど）を入力して **「Apple Wallet に追加」** を完了します。

### Google Wallet（Android）への登録手順
1. Android スマートフォンをインターネットに接続します。
2. カメラアプリまたは QR コードスキャナーを起動します。
3. Google Smart Tap 用パス発行 QR コードをスキャンします。<br/>
   <img height="250" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/qrcode.jpg">

4. パス設定 URL に接続され、必要情報を入力して **「Google ウォレットに追加」** をタップし、登録を完了します。

> ⚠️ **注意**: Samsung Wallet パスを検証する場合は、事前に Samsung Developer サイト等を通じて登録・発行されたパスを使用してください。

---

## 4. KC401 へのデモ環境セットアップ (Installation)

KC401 にデモアプリと必要なバックグラウンドサービスをインストールします。

```
[ホスト PC]
   │
   ├── (USB-C 転送 または ADB)
   ▼
[Zebra KC401]
   ├── 1. ZebraVasService-x.x.x.apk  (バックグラウンド通信サービス)
   └── 2. ZebraVasDemoApp-x.x.x.apk    (デモ操作アプリ)
```

### ステップ 1: パッケージの展開
1. Zebra Support Portal からダウンロードした `Zebra VAS SDK` の zip アーカイブをホスト PC 上で解凍します。
2. 解凍されたフォルダ内から以下の 2 つの APK ファイルを確認します:
   * `ZebraVasService-x.x.x.apk`
   * `ZebraVasDemoApp-x.x.x.apk`

### ステップ 2: KC401 への転送とインストール
KC401 とホスト PC を USB-C ケーブルで接続します。以下のいずれかの方法でインストールします。

#### 方法 A: ADB コマンドによるインストール（推奨）
```bash
# 接続確認
adb devices

# 1. Zebra VAS Service のインストール
adb install -r "ZebraVasService-x.x.x.apk"

# 2. NFC Ticket Demo アプリのインストール
adb install -r "ZebraVasDemoApp-x.x.x.apk"
```

#### 方法 B: デバイス画面からの手動インストール
1. KC401 の内部ストレージ（`Download` フォルダ等）へ両方の APK ファイルをファイル転送（MTP モード）でコピーします。
2. KC401 のアプリ一覧から **「Files（ファイル）」** アプリを開きます。
3. 最初に **`ZebraVasService-x.x.x.apk`** をタップしてインストールを実行します。
4. 次に **`ZebraVasDemoApp-x.x.x.apk`** をタップしてインストールを完了します。

---

## 5. デモの実行・検証手順 (Using the Demo)

### ステップ 1: アプリケーションの起動
1. KC401 のアプリアクティビティ一覧から **「NFC Ticket Demo」** アイコンをタップして起動します。
2. アプリ画面が開き、NFC 読み取り待機状態になります。<br/>
    <img height="300" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/app-main-screen.png">

### ステップ 2: スマートフォンのパスを表示
1. 検証用スマートフォン（iPhone または Android）でウォレットアプリを開きます。
2. 事前に登録したテスト用パス（Zebra VAS Pass）を画面上にアクティブ表示させます。  <br/>
    <img height="300" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/ios-demo-1.jpg">  <img height="300" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/android-demo.jpg">

### ステップ 3: KC401 ディスプレイへのタップ（Tap-on-Glass）
1. スマートフォンの NFC アンテナ位置を確認します:
   * **iPhone**: 上部先端 / 背面上部
   * **Android**: 背面中央〜上部
   * **Apple Watch / Smart Watch**: ディスプレイ表面
2. スマートフォンを **KC401 のディスプレイ表面（タップエリア）** に近づけます。
   * KC401 は **Tap-on-Glass** をサポートしているため、画面中央付近に軽くタッチするようにかざします。

### ステップ 4: 読み取り結果の確認
* **読み取り成功時**:
  * KC401 の画面上に **緑色の円で囲まれたチェックマーク（✓）** が表示されます。
  * 復号されたパスデータ（Collector ID、パス番号、顧客属性情報など）が画面にリスト表示されます。
  * スマートフォン側にも読み取り完了を示すバイブレーションやチェックマークが表示されます。<br/>
    <img height="300" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/read-success.png">  <img height="300" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/ios-demo-2.jpg">  <img height="300" src="https://techdocs.zebra.com/nfc-vas/latest/guide/demo/android-demo.jpg">

---

## 6. KC401 向けトラブルシューティング & 運用Tips

| 現象 / 課題 | 主な原因 | 対処方法 |
|---|---|---|
| **サービス接続エラーが表示される** | `Zebra VAS Service` が未インストールまたは停止 | `Zebra VAS Service.apk` が正しくインストールされているか確認し、デバイスを再起動してください。 |
| **スマートフォンをかざしても反応しない** | NFC 設定が無効、またはタップ位置のずれ | ① KC401 の Android 設定 > **「接続済みのデバイス」** > **「NFC」** が ON になっているか確認します。<br>② スマートフォンの NFC アンテナ位置（iPhone は先端部）を KC401 のディスプレイ中央にしっかりかざしてください。 |
| **LED バーが赤色に点滅している** | 給電能力不足（PoE クラス不適合） | KC401 は **PoE+ (802.3at / 30W)** 以上が必要です。通常の 15W PoE ハブでは給電不足警告（赤点滅）となるため、PoE+ スイッチまたは付属 DC 電源アダプタをご使用ください。 |
| **パスデータが暗号化されたまま表示される** | 秘密鍵（Private Key）の不一致 | デモアプリ既定の Zebra Collector ID パス以外を使用している場合、該当プロファイル用の復号キーがアプリ側に設定されているか確認してください。 |


# 環境センサー基板（SCD41 / SHT35 / BMP581）

部屋の **CO2・温度・湿度・気圧** を測る、I2C のセンサー基板の製造データです。[EasyEDA Pro](https://pro.easyeda.com/) で設計し、[JLCPCB](https://jlcpcb.com/) で基板の製造と部品の実装（PCBA）まで頼める形になっています。

| 項目 | 内容 |
|---|---|
| センサー | SCD41（CO2・温度・湿度）/ SHT35（温度・湿度）/ BMP581（気圧・温度） |
| 接続 | I2C（Qwiic コネクタ）、**3.3V 専用** |
| サイズ | 50.3 × 24.9 mm、2 層 |
| OSHWLab | <https://oshwlab.com/yunayuna/project_ibxqleze> |

> [!NOTE]
> OSHWLab のページは審査中のため、まだ開けないことがあります。開けない場合は、このリポジトリのファイルで注文できます。

## 使用例

この基板を実際に使っているプロジェクトです。センサーを読むコードもそれぞれにあります。

### [tab5-env-monitor](https://github.com/ynug/tab5-env-monitor)（M5Stack Tab5）

M5Stack Tab5 の画面に CO2・温度・湿度・気圧を大きく表示する据え置き型のモニターです。24 時間分のグラフや、換気・熱中症の音声アラームがあります。

- つなぎ方: Tab5 の PORT.A は 5V なので、[変換基板（grove2qwiic-3v3）](https://github.com/ynug/tab5-env-monitor/tree/main/hardware) で 3.3V にしてから Qwiic ケーブルでつなぎます
- センサーのドライバー: C（ESP-IDF）

### [home-display](https://github.com/ynug/home-display)（Raspberry Pi）

Raspberry Pi とモバイルディスプレイで、カレンダー・天気・雨雲レーダーを表示する情報ディスプレイです。室内の温度・湿度・CO2・気圧を天気カードに表示します。

- つなぎ方: GPIO の 3.3V と I2C に直接つなぎます（[下の配線](#raspberry-pi-につなぐ)）
- センサーのドライバー: Rust（`embedded-hal`）

## 回路

| 部品番号 | 部品 | I2C アドレス | LCSC 番号 |
|---|---|---|---|
| U4_SCD41 | SCD41-D-R2（CO2・温度・湿度） | `0x62`（固定） | C3659294 |
| U2_SHT35 | SHT35-DIS-B2.5KS（温度・湿度） | `0x44`（ADDR = GND） | C90161 |
| U1_BMP581 | BMP581（気圧・温度） | `0x47`（SDO = 3.3V） | C5362283 |
| CN1 | Qwiic コネクタ SM04B-SRSS-TB | | C160404 |
| R_I2C_SDA, R_I2C_SCL | 4.7kΩ 0603（I2C のプルアップ） | | C23162 |
| C_SCD41_0V1, C_SHT35_0V1, C_BMP581_VDD, C_BMP581_VDDIO | 100nF 0603 | | C14663 |
| C_SENS_BULK | 10µF 0805 | | C15850 |

- 電源は **3.3V 専用**です（レギュレーターは載っていません）。5V をつなぐと BMP581 などの定格を超えて壊れます
- Qwiic（CN1）のピン: 1 GND / 2 3.3V / 3 SDA / 4 SCL
- SDA / SCL は 4.7kΩ で 3.3V にプルアップしてあります
- BMP581 は CSB を 3.3V につないで I2C モードにしています。SHT35 の ALERT・nRESET と BMP581 の INT は未接続です
- SHT35 のまわりと、SCD41 と BMP581 の間には、熱を伝えにくくするための切り欠き（スロット）があります

## Raspberry Pi につなぐ

Qwiic とジャンパーワイヤー（メス）の変換ケーブルで、Raspberry Pi の 40 ピンヘッダーに直接つなぎます。

| Qwiic（CN1） | 線の色（Qwiic 標準） | Raspberry Pi |
|---|---|---|
| 1 GND | 黒 | 6 番ピン（GND） |
| 2 3.3V | 赤 | **1 番ピン（3.3V）** |
| 3 SDA | 青 | 3 番ピン（GPIO2 / SDA） |
| 4 SCL | 黄 | 5 番ピン（GPIO3 / SCL） |

> [!WARNING]
> 赤の線を **2 番・4 番ピン（5V）につながないでください**。1 番ピン（3.3V）です。

- Raspberry Pi の GPIO2 / GPIO3 には、本体側にもプルアップ抵抗（1.8kΩ）があります。この基板の 4.7kΩ と並列になりますが、そのまま使えます
- つないだあと `sudo i2cdetect -y 1` で `44` `47` `62` が表示されれば配線は正しいです
- Raspberry Pi 本体は熱を持つので、センサー基板はケーブルで少し離して置いてください

## 注文のしかた

### OSHWLab から（かんたん）

1. 上の OSHWLab のページを開く
2. プロジェクトを自分の EasyEDA アカウントにコピー（クローン）する
3. EasyEDA Pro で開き、PCB 画面から JLCPCB に注文する（基板と部品実装をまとめて頼めます）

### ファイルから

[`EnvMonitor_SCD41-SHT35-BMP581_I2C/`](EnvMonitor_SCD41-SHT35-BMP581_I2C/) の 3 つのファイルを JLCPCB の注文画面でアップロードします。

| ファイル | 使う場所 |
|---|---|
| `Gerber_*.zip` | 最初の基板の注文画面（ガーバー） |
| `BOM_*.xlsx` | 部品実装（PCB Assembly）を ON にしたあとの BOM |
| `PickAndPlace_*.csv` | 同じく CPL（部品の位置） |

### 注文時の設定

作者が注文したときの設定です。書いていない項目は標準のままです。

| 項目 | 設定 |
|---|---|
| Base Material | FR-4（TG135） |
| Layers | 2 |
| PCB Thickness | 1.6mm |
| PCB Color / Silkscreen | Green / White |
| Surface Finish | **ENIG**（1U"） |
| Outer Copper Weight | 1 oz |
| Via Covering | Tented |
| Electrical Test | Flying Probe Fully Test |
| Mark on PCB | Order Number |

- 表面処理を ENIG にしているのは、SCD41・BMP581（LGA）や SHT35（DFN）のように足が裏に隠れる部品を、平らなパッドにきれいにはんだ付けするためです
- 切り欠き（スロット）を JLCPCB に消されないよう、Remark に次の文を入れてください

  ```
  The slots near U2 and between U4 and U1 are intentional for thermal isolation. Please keep them as designed.
  ```

- 部品実装（PCBA）は **表面（Top）のみ** です。全部品を JLCPCB で実装できます（手ではんだ付けする部品はありません）
- 部品実装のため、JLCPCB 側でまわりにレール（捨て板）を付けた状態で製造され、注文画面のサイズは約 70 × 71 mm と表示されます。届く前にレールは切り離されるので、届くのは 50.3 × 24.9 mm の基板です

### 部品実装オプション

**SCD41（C3659294）は、JLCPCB 標準の高温はんだでは壊れます。** 注文時に自分で中温はんだ（Medium temp.）を選んでください。

作者が標準のはんだで注文したときは、JLCPCB の製造前チェックで気づいてもらえて、メールで確認が来ました（「Confirm Production file」は No のままでした）。ただし、毎回気づいてもらえるとは限らないので、最初から選んでおくのが安全です。

| 項目 | 設定 | 理由 |
|---|---|---|
| Solder Paste | **Medium temp.** | SCD41 を高温で壊さないため（追加料金あり） |
| Packaging | ESD + Cardboard | 中温はんだはつきが弱いので、輸送中に部品が外れないよう梱包の指定が必要 |
| Depanel boards & edge rail before delivery | Yes | レールを切り離してから届けてもらう |
| Special parts for cleaning | Yes | 中温はんだと梱包を指定したときに JLCPCB 側で追加された項目（追加料金なし） |

中温はんだは高温より接合が弱く、はんだに黒い点が出ることがあると JLCPCB から説明があります。

Assembly remark（部品実装へのメモ）は書かなくてかまいません。作者は部品の向きの確認をお願いしましたが、このデータの向きは問題ないことを確認済みです。

> [!IMPORTANT]
> 注文画面の実装プレビューで、**部品の向き**を必ず確認してください。特に IC の 1 番ピンの印と、コネクタの向きがずれやすいです。注文後、製造用データができたら、注文履歴（Order History）でも確認できます。

注意:

- 基板は 5 枚から、部品実装は 2 枚からの注文になります
- SCD41・SHT35・BMP581・コネクタなどは JLCPCB の Extended パーツで、部品の種類ごとに追加料金がかかります
- LCSC の在庫は変わります。欠品しているときは、上の部品表の「LCSC 番号」と同じ仕様の部品に置き換えてください

## ライセンス

[MIT License](LICENSE)（© 2026 yuna）

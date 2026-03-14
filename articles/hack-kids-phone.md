---
title: "キッズケータイを解析しよう"
emoji: "📱"
type: "tech"
topics:
  - "android"
  - "linux"
  - "adb"
  - "cve"
published: false
---

# はじめに
こんにちは
初めての人ははじめまして
そうでない方の貴方は誰ですか
今回、いろいろあってDocomoのF-03Jというキッズケータイを入手したのでこれで遊んでいきます

# 基本情報　(すっ飛ばしておｋ
## 1. デバイス

| 項目 | 値 |
|------|------|
| モデル名 | F-03J |
| メーカー | FUJITSU (富士通) |
| ブランド | DOCOMO |
| コードネーム | MONAD |
| Android バージョン | 5.1.1 (SDK 22) |
| ビルド ID | V33R125A |
| セキュリティパッチレベル | 2017-01-05 |
| ベースバンド | 7088.0001.0294 |
| カーネル | Linux 3.10.49 (SMP PREEMPT, 2019-02-22) |
| ビルドタイプ | user (リリースビルド) |

## 2. ハードウェアスペック

### SoC / CPU
| 項目 | 値 |
|------|------|
| チップセット | Qualcomm MSM8916 (Snapdragon 210) |
| CPU | ARM Cortex-A53 (ARMv7) x4コア |
| クロック | 38.40 BogoMIPS |
| 対応ABI | armeabi-v7a, armeabi |
| GPU | Adreno (OpenGL ES 3.0) |

### メモリ
| 項目 | 値 |
|------|------|
| RAM 合計 | 約470MB (481,636 KB) |
| zRAM | 256MB (スワップ用圧縮RAM) |
| low_ram デバイス | はい (`ro.config.low_ram=true`) |
| バックグラウンドアプリ上限 | 16個 |

### ストレージ
| パーティション | サイズ | 使用量 | 空き |
|------|------|------|------|
| /system | 848.2MB | 357.5MB | 490.7MB |
| /data | 1.5GB | 177.4MB | 1.3GB |
| /cache | 248.0MB | 236.0KB | 247.7MB |
| /persist | 192.8MB | 9.3MB | 183.5MB |
| /firmware | 85.0MB | 48.4MB | 36.6MB |
| /fota | 232.2MB | 140.0KB | 232.1MB |
| eMMC 合計 | 約3.6GB | - | - |

SDカードスロットあり（現在未挿入）

### ディスプレイ
| 項目 | 値 |
|------|------|
| 解像度 | 240 x 320 (QVGA) |
| DPI | 160 (mdpi) |
| 物理DPI | 196.645 x 198.243 |
| リフレッシュレート | 60Hz |
| タイプ | BUILT_IN (内蔵スクリーン) |
| タッチスクリーン | なし (`-touch` in config) |

### バッテリー
| 項目 | 値 |
|------|------|
| タイプ | Li-ion |
| ワイヤレス充電 | 非対応 |
| ホルダー充電 | 対応（専用） |

### LED
- `lcd-backlight` - 画面バックライト
- `button-backlight` - ボタンバックライト
- `red` - 赤色LED（通知用）
- `white` - 白色LED（通知用）
- `keyboard` - キーボードライト

### センサー
ハードウェアセンサー: 何もなし

## 3. 通信機能

### モバイル通信
| 項目 | 値 |
|------|------|
| キャリア | NTT DOCOMO |
| MCC/MNC | 440/10 |
| ネットワークタイプ | UMTS / HSDPA (3G) |
| SIM 状態 | READY |
| ローミング | OFF |
| 緊急番号 | 110, 118, 119, 911, 112 |
| データ通信 | 可能（dataEnabled） |

### Bluetooth
| 項目 | 値 |
|------|------|
| 状態 | ON |
| デバイス名 | F-03J |
| BLE (Bluetooth Low Energy) | 対応 |
| プロファイル | A2DP Sink, AVRCP Controller, GATT, HFP Client, HeadsetClient |
| OPP/FTP/PBAP/MAP | 非対応 |

### WiFi
非搭載 (多分)

### GPS
| 項目 | 値 |
|------|------|
| ハードウェア | 搭載 |
| プロバイダー | Qualcomm (`com.qualcomm.location`) |
| aGPS | 有効 (`ro.gps.agps_provider=1`) |
| 位置情報サービス | Fused, Passive, Network, GPS |

## 4. 物理キー

DPADという専用Padがある
スクリーンの下部にあるボタン群のとこ

![Devices](https://www.docomo.ne.jp/support/product/f03j/images/f03j_pc.jpg)

## 5. ソフトウェア

### OS / フレームワーク
- ART ランタイム (`libart.so`)
- SELinux 有効
- セーフモード: 起動時にどっかのボタンを一緒に押すとできる
- 暗号化: 非暗号化
- root: 不可 suも無い
- USB デバッグ: 有効化可能
- 
## 6. 特殊な富士通独自機能

### ブザー機能
カーネルレベルで `fj_buzzer` ドライバが組み込まれている。防犯ブザー機能を実現。

### LED制御
`ledctrld` デーモン（/system/xbin/）が常駐し、通知LED（赤/白）を制御。

### 温度監視
`FjTemperatureServer` が常駐し、バッテリー温度等を監視。

### 自動電源OFF
`FjAutoPowerOff` で指定時間に自動シャットダウン可能。

### 見守り機能 (KidsMonitor)
- Bluetooth 状態監視
- 時刻変更監視
- メール定型文管理（FSKARENと連携）
- 起動時自動起動（BOOT_COMPLETED）

### 機能制限 (FunctionLimitManagement)
- signature|system レベルの権限で保護
- 親が設定する機能制限を管理

### ケースカラー検出
`/system/bin/FjCasecolorDecider`があり、端末カラーを識別可能。

# 何をするか
- adbでapk入れたり
- インターネットに接続できるようにしたり
- rootとったり

などなど
なお、今回からclaudeとかをぶん回してます
あんまやってること(CVEのPoCとか)は詳しくは理解してないので、ふんわり掴めたらなと

# やったこと1: adbで遊ぶ

adb shellで色々できるので

- claudeに自由にさせる
- termuxいれる
- webサイト見る

などなどした


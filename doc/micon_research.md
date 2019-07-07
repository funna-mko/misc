# マイコン調査
## 1.概要
IOT用途で利用するロガーなどのモジュールを作成する目的で、
モジュールのキーパーつとなる市販マイコンの仕様と一般的な用途を調べる。

対象とするマイコンは以下の2通り
- adruino
- RasberryPi

## 2.Arduino
### 2.1.ラインナップ

|--------|Arduino UNO  |Arduino Micro| Mega 2560|
|--------|-------------|-------------|-------------|
|CPU     | ATmega328P  | ATmega32u4  | ATmega256k  |
|Bus/Clk | 8bit/16M    | 8bit/16M    | 8bit/16M    |
|電圧    | 5V/7-12V    | 5V / 7-12V  | 5V / 7-12V  |
|DIO／PWM| 14/6        | 20/7        | 54/15       |
|A In/Out| 6/0         | 12/0        | 16/0        |
|EEPROM  | 1kB         | 1kB         | 4kB         |
|SRAM    | 2kB         | 2.5kB       | 8kB         |
|Flas    | 32kB        | 32kB        | 256kB       |
|USB     | あり        | あり        | あり        |
|価格    | 1,100       | 800-1,000   | 1,150-5,000 |

## 3.RasberryPi
### 3.1.ラインナップ

|---------|RasPi3 modelB+ |RasPi Zero WH |
|---------|---------------|--------------|
|CPU      | ARM cortex-A53| ARM11763ZF-S |
|Core/Clk | 4 / 1.2GHz    | 1 / 1GHz     |
|電源     | 5V 2.5A       | 5V           |
|GPIO     | 40pin 2.54mm  | 40pin 2.54mm |
|AIn/Out  | 無し          | 無し         |
|DRAM     | 1GB           | 512MB        |
|Strage   | microSD       | microSD      |
|LAN      | 1GB overUSB2.0| なし         |
|Wifi     | 2.4 + 5 GHz   | 2.4GHz       |
|Bluetooth| 4.2 BLE       | 4.1 BLE      |
|USB      | 2.0 * 4port   | 2.0mini*1port|
|価格     | $35           | $14          |

### 3.2.OS
#### ◆ Raspbian
 - Debian ベース。
 - Pyhon, Scratch, Mathematica, Java, Sonic Pi がプリインストールされている。
 - SDカードのイメージとして配布される。
#### ◆ NOOBS / NOOBS Lite
  - Raspberry Pi 公式OSとしてリリースされたもの以下のOSが含まれる。
    - Ubuntu (MATE)
    - Windows10 IOT Core
    - RiscOS
    - SUSE
  - NOOBSは各OSのインストーラーをSDカードに格納しておき、初回起動時にインストーラを実行する方式。（NOOBSから Raspbian をインストールすることも出来る）
  - NOOBS Lite は各OSのインストーらが含まれていない版


### 4.インターフェース

|         |I2C            |SPI           |UART            |
|---------|---------------|--------------|----------------|
|転送速度 |100k-1Mbps     | nMbps        | 〜115kbps      |
|接続線数 |2本            | 3-4本        | 1-2本          |
|通信方式 |クロック同期   | クロック同期 | 調歩同期       |
|         |Master-Slv     | Master-Slv   | 全二重／半二重 |


### 5. RaspberryPi 環境構築

#### 5.1 Raspbian インストール

インストール方法は大きく分けて次の2通り。
1) NOOBS（Raspbian用のインストーラ）を microSD に焼いて、NOOBSから Raspbianをインストールする。
1) Raspberry Pi の Bootable イメージを直接 microSD に書き込む
   公式サイトからダウンロードできる Raspbianのブータブルイメージは3通り。
   - Raspbian Stretch with Desktop And Recommended Software
   - Raspbian Stretch with Desktop
   - Raspbian Stretch Lite


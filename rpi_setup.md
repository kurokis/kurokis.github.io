---
title: Raspberry Pi Setup
---

# Raspberry Pi 3のセットアップ

Raspberry Piのプログラムを動かすためには以下をインストールする必要がある。
- gcc: C/C++コンパイラ
- CMake: Makefileの自動生成ツール、これを使うことでC/C++のコンパイルが楽になる

gccをインストール
```bash
$ sudo apt-get install build-essential
```

CMakeをインストール
```bash
$ sudo apt-get install cmake
```

## Wifiの優先度設定とIPアドレス固定

外部サイト: [RaspberryPi で複数 Wifi 環境に個別設定を行う方法](http://kouki-hoshi.hatenablog.com/entry/2016/07/16/153836)

以下を試す予定。（未検証）

### /etc/wpa_supplicant/wpa_supplicant.conf

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
ｃountry=GB

network={
    ssid="ssid1-????????"
    psk="????????"
    priority=0
    key_mgmt=WPA-PSK
}

network={
    ssid="ssid2-????????"
    psk="????????"
    priority=1
    key_mgmt=WPA-PSK
}
```

### /etc/network/interfaces

```
iface eth0 inet manual

allow-hotplug wlan0
iface wlan0 inet manual
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

### /etc/dhcpcd.conf

192.168.1.100と192.168.20.30に固定したい場合：

```
interface wlan0
ssid SSID-xxxx
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1

ssid SSID-yyyy
static ip_address=192.168.20.30/24
static routers=192.168.20.1
static domain_name_servers=192.168.20.1
```

## GPSデバイス

Raspberry Piに接続するGPSデバイスとしては **BU-353-S4 SiRFstar IV 搭載 USB GPS レシーバ** を用いる。

### GPSの永続的なアクセス権変更とデバイス名固定

#### アクセス権とは
USBデバイスのアクセス権(permission)はマシン起動のたびに変わってしまう可能性がある。
ターミナルでは、アクセス権の変更はchmodで行えるが（``chmod 666 /dev/ttyUSB0``, 666は全員読み書き可能: rw-rw-rw- を意味する）、
起動のたびにアクセス権を変更していたのでは組み込みアプリケーションに使えない。
したがって、Raspberry PiではGPSデバイスのアクセス権を永続的に変更するべきである。

#### デバイス名とは
USBデバイスをUSBポートに接続されるとLinux上からは /dev/ttyUSB0 のように見える。
現在接続されているUSBデバイスは以下のコマンドで確認できる。
```bash
ls /dev/ttyUSB*
```
デバイス名は、USBポートにデバイスを差し込む順番などにより変わる可能性がある。
例えば、GPSデバイスが/dev/ttyUSB0の時もあれば、/dev/ttyUSB1の時もあるが、
デバイス名がこのように可変であるとプログラム上でシリアル通信を設定するのが困難になる。
したがって、Raspberry PiではGPSデバイスのデバイス名を固定するべきである。
より具体的には、GPSデバイスにsymbolic link(SYMLINK)を付加し、USBポートに接続されたら
それがttyUSB0であろうとttyUSB1であろうとttyUSB_GPSという名前で参照できるようにする

#### やること


1. GPSデバイスのidVendor, idProductの取得

    GPSデバイスを識別するために、デバイスが保有するidVendorとidProductを確認する。ターミナルで``lsusb -v``を実行し、デバイスのidVendorとidProductを覚えておく。

    BU-353-S4の場合は以下のようになる。
    厳密には、これはPL2303 Serial Portのプロパティである。

    | Attributes | Value |
    |------------|-------|
    | idVendor   | 067b  |
    | idProduct  | 2303  |


1. udevルールを変更する

    USBデバイスの設定は /etc/udev/rules.d に格納されており、ここに新しくルールを定義する。50-myusb.rulesというファイルを作り、テキストエディタ(例えばgedit)でその内容を編集する。

    geditを起動
    ```bash
    $ sudo gedit /etc/udev/rules.d/50-myusb.rules
    ```

    50-myusb.rulesの内容を以下のように記述する。
    なお、idVendor,idProductはそれぞれ067b,2303であったと仮定する(これはPL2303 Serial Portに該当)。
    ```
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="067b", ATTRS{idProduct}=="2303", GROUP="users", MODE="0666", SYMLINK+="ttyUSB_GPS"
    ```

    意味は以下の通り
    - SUBSYSTEMS=="usb": USBデバイス
    - ATTRS{idVendor}=="067b": idVendorが067bのものを対象とする
    - ATTRS{idProduct}=="2303": idProductが2303のものを対象とする
    - MODE="0666": 全ユーザーに対して読み書きのアクセス権を与える
    - SYMLINK+="ttyUSB_GPS": このデバイスにttyUSB_GPSのシンボリックリンクを付加

2. 新しいルールをロードする

    Raspberry Piを再起動するか、以下を実行してルールをリロードする。
    ```
    $ sudo udevadm trigger
    ```

3. アクセス権が変更されていることを確認する

    ``ls -al /dev/ttyUSB*``を実行し、以下のように表示されていれば成功である。"rw-rw-rw"はUser,Group,Other(つまり全員)に読み書き(rw: read & write)の許可を与えていることを意味する。
    ```bash
    crw-rw-rwT 1 root dialout 188,  ... /dev/ttyUSB0
    ```

4. デバイス名が変更されていることを確認する

    ``ls -l /dev/ttyUSB*``を実行する。上手くいっていれば、/dev/ttyUSB_GPS -> ttyUSB0のような表示がある。


## 製品のシリアル番号を使う方法

外部サイト：
[usb-serial のデバイスファイル名を固定する方法](http://d.hatena.ne.jp/pyopyopyo/20160223/p1)

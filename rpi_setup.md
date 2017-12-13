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

## 開発に必要なハードウェア


|ハード|製品名|値段|
|------|-----|---|
|Raspberry Pi本体|Raspberry Pi 3 Model B|4500円|
|モニター|Waveshare 7インチHDMI LCD タッチスクリーン ディスプレイ|10000円|
|HDMIケーブル|なんでも（千石電商か秋月電子で買うのが確実）|300円|
|microSDカード|16GB以上|1500円|
|ACアダプタ|定格3A 5Vのもの|500円|
|カメラ|Raspberry Pi Camera V2|3500円|
|キーボード|マイクロソフト All-in-One Media Keyboard N9Z-00029|4000円|

Raspberry Pi、モニター、HDMIケーブル、microSDカードは千石電商でまとめて買うのが楽。特にHDMIケーブルは相性の問題もあるので、店舗で買うのが安くて確実。

## セットアップ方法

はじめにPreferences>Raspberry Pi Configuration>Interfaces>CameraをEnabledにしておく．Raspberry Pi公式のCamera Moduleの動作確認のためには，ターミナルで`raspistill -o photo.jpg` を実行すればよい．開始から数秒後に写真が撮影されるはずである．

 - OpenCV

    OpenCV関係のインストール方法はここに従った．ビルドには5時間程度かかる．
    https://www.pyimagesearch.com/2017/09/04/raspbian-stretch-install-opencv-3-python-on-your-raspberry-pi/

 - Eigen

    ``$ sudo apt-get install libeigen3-dev``

 - RaspiCam

    カメラモジュールをOpenCVで楽に使うためのライブラリ
    公式サイトを参考にインストール
    https://www.uco.es/investiga/grupos/ava/node/40

    最新のもの(raspicam-0.1.6)をダウンロードし解凍．
    ※重要：OpenCVをインストールしておく必要がある．
    はじめにrpi-updateを実行する．
    ```
    $ sudo rpi-update
    ```
    RaspiCam本体をビルド．
    ```
    $ cd Downloads/raspicam-0.1.6
    $ mkdir build
    $ cd build
    $ cmake ..
    ```
    ダイアログに
    ``-- CREATE OPENCV MODULE=1``と表示されていることを確認し，
    ```
    $ make
    $ sudo make install
    $ sudo ldconfig
    ```

 - aruco

    OpenCV同梱版ではなく，公式版をインストール
    ```
    $ cd Downloads/aruco303
    $ mkdir build
    $ cd build
    $ cmake ..
    $ make -j4
    $ sudo make install
    ```

    インストール後に次の設定をする必要がある．[参考](    http://miloq.blogspot.jp/2012/12/install-aruco-ubuntu-linux.html)
    なお，leafpadはRaspbian Stretchのデフォルトのテキストエディタである．Raspbian Jessieの場合はgeditなどを使えばよい．
    ```
    $ sudo leafpad /etc/ld.so.conf.d/aruco.conf
    ```


## Wifiの優先度設定とIPアドレス固定

外部サイト: [RaspberryPi で複数 Wifi 環境に個別設定を行う方法](http://kouki-hoshi.hatenablog.com/entry/2016/07/16/153836)

以下を試す予定。（未検証）

### /etc/wpa_supplicant/wpa_supplicant.conf

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

network={
        ssid="????????"
        psk="????????"
        priority=0
        key_mgmt=WPA-PSK
        id_str="WifiPocketRouter1"
}

network={
        ssid="????????"
        psk="????????"
        priority=1
        key_mgmt=WPA-PSK
        id_str="LabWirelssRouterBuffaloG246E"
}

```

### /etc/network/interfaces

```
auto wlan0
allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp

iface WifiPocketRouter1 inet static
address 192.168.128.169
netmask 255.255.255.0
gateway 192.168.128.1


iface LabWirelessRouterBuffaloG246E inet static
address 192.168.1.101
netmask 255.255.255.0
gateway 192.168.1.1

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
    | iSerial    | ???? (optional) |


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

    Using serial number (optional)
    ```
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="067b", ATTRS{idProduct}=="2303", ATTRS{serial}=="????", GROUP="users", MODE="0666", SYMLINK+="ttyUSB_GPS"
    ```


    意味は以下の通り
    - SUBSYSTEMS=="usb": USBデバイス
    - ATTRS{idVendor}=="067b": idVendorが067bのものを対象とする
    - ATTRS{idProduct}=="2303": idProductが2303のものを対象とする
    - ATTRS{serial}="????"
    - MODE="0666": 全ユーザーに対して読み書きのアクセス権を与える
    - SYMLINK+="ttyUSB_GPS": このデバイスにttyUSB_GPSのシンボリックリンクを付加

    Note
    - == -> condition
    - = -> insert
    - += -> append

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


### 製品のシリアル番号を使う方法

外部サイト：
[usb-serial のデバイスファイル名を固定する方法](http://d.hatena.ne.jp/pyopyopyo/20160223/p1)

Suppose idVendor is 0403. Then
`dmesg | grep -iC 4 0403`
should give the serial number unique to the device.


## USB-GPIO cable

Bus 001 Device 007: ID 0403:6015 Future Technology Devices International, Ltd Bridge(I2C/SPI/UART/FIFO)
Couldn't open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0         8
  idVendor           0x0403 Future Technology Devices International, Ltd
  idProduct          0x6015 Bridge(I2C/SPI/UART/FIFO)
  bcdDevice           10.00
  iManufacturer           1
  iProduct                2
  iSerial                 3
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           32
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0x80
      (Bus Powered)
    MaxPower               90mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass       255 Vendor Specific Class
      bInterfaceSubClass    255 Vendor Specific Subclass
      bInterfaceProtocol    255 Vendor Specific Protocol
      iInterface              2
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x02  EP 2 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0

---
title: Multicopter Setup
---

# マルチコプター開発ガイド

## 概要

このガイドは、マルチコプターの開発に必要なハードウェア、ソフトウェア、通信プロトコル等の情報を網羅する目的で作成した。

記載情報は適宜更新してゆく予定である。

## カスタムファームウェア

### FlightCtrl

マルチコプター制御の根幹を担う。プロポとの通信、姿勢制御、位置制御、ESCへのコマンド送信などを行う。

[FlightCtrlの詳細](flightctrl.html)

ソースコード: [https://github.com/kurokis/FlightCtrl2](https://github.com/kurokis/FlightCtrl2)

### NaviCtrl

高レベル処理を行う。位置推定（センサ融合）、画像処理、GPS受信、ロギング、他のコンピュータとの通信などを行う。

[NaviCtrlの詳細](navictrl.html)

ソースコード: [https://github.com/Akihiro-K/RasPiMain](https://github.com/Akihiro-K/RasPiMain)

### MK-Programmer

上記FlightCtrlファームウェアの書き込みを行う。

ソースコード: [https://github.com/ctraabe/MKProgrammer](https://github.com/ctraabe/MKProgrammer)


## ハードウェア

### 構成
![](http://g.gravizo.com/g?
  digraph G {
    DronePort [shape=box]
    WaypointController [shape=box]
    GPS [shape=box]
    Camera [shape=box];
    NaviCtrl [shape=box]
    FlightCtrl [shape=box]
    ESC [shape=box]
    RCTransmitter [shape=box]
    WifiRouter [shape=box]
    MonitorPC [shape=box]
    DronePort -> WaypointController
    WaypointController -> DronePort
    WaypointController -> NaviCtrl[label="UART via USB"]
    NaviCtrl -> WaypointController
    Camera -> NaviCtrl[label="CSI"]
    GPS -> NaviCtrl[label="UART via USB"]
    NaviCtrl -> FlightCtrl[label="UART via GPIO"]
    FlightCtrl -> NaviCtrl;
    RCTransmitter -> FlightCtrl[label="SBus"]
    FlightCtrl -> ESC [label="I2C"]
    MonitorPC -> WifiRouter [label="ssh"];
    WifiRouter -> NaviCtrl [label="ssh"];
  }
)

### 使用機材
  - プロポ：[Futaba 14SG](https://www.rc.futaba.co.jp/propo/air/14sg.html)
  - FlightCtrl
    - ボード: [FlightCtrl V2.5](http://wiki.mikrokopter.de/en/FlightCtrl_ME_2_5)
    - ライター
      - [MK-USB V1.0](http://wiki.mikrokopter.de/en/MK-USB)
      - USB-micro USB cable (any)
      - [10ピンリボンケーブル](http://akizukidenshi.com/catalog/g/gC-03796/)
  - NaviCtrlボード: [Raspberry Pi 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) model?
  - Camera: ?
  - GPS: [Globalsat BU-353S4](http://www.globalsat.com.tw/s/2/product-199952/Cable-GPS-with-USB-interface-SiRF-Star-IV-BU-353S4.html)
  - 機体
    - ESC: AfroESC (Afro 20A Muti-Rotor ESC (SimonK Firmware)?)
    - Motor: ?
    - Propeller: ?

### WaypointControllerとNaviCtrlの接続について

WaypointControllerとNaviCtrlの通信はUARTであるが、NaviCtrl側の接続はRaspberry PiのGPIOピンではなく**USBポート**を使う。これは、NaviCtrlのGPIOピンがFlightCtrlとの通信で占有されているからである。一方、WaypointController側の接続はGPIOピンを使う。

## 開発用Ubuntuマシンのセットアップ

FlightCtrlはUbuntuマシンから書き込む必要がある。

[Ubuntuマシンのセットアップ](ubuntu_setup.html)

## Raspberry Pi 3のセットアップ

NaviCtrlはRaspberry Pi 3上で動作する。

[Raspberry Piのセットアップ](rpi_setup.html)

## Linux Cheat Sheet

Linux共通(開発用Ubuntuマシン、Raspberry Pi)のチートシート

[Cheat Sheet](linux_cheat_sheet.html)

## 通信プロトコル

![](http://g.gravizo.com/g?
  digraph G {
    WaypointController [shape=box]
    FlightCtrl [shape=box]
    RasPiMain [shape=box]
    Marker [shape=box]
    GPSServer [shape=box]
    WaypointController -> RasPiMain [label="UT Protocol"]
    RasPiMain -> WaypointController
    FlightCtrl -> RasPiMain [label="UT Protocol"]
    RasPiMain -> FlightCtrl
    Marker -> RasPiMain [label="TCP"]
    GPSServer -> RasPiMain [label="TCP"]
  }
)

FlightCtrl, NaviCtrl(RasPiMain, Marker, GPSServer), WaypointController間の通信はすべてUT Protocolに基づくシリアル通信で行う。
[UT Protocolの詳細](ut_protocol.html)

MainProcess, MarkerProcess, GPSProcess間の通信はTCP通信で行う。


## Drone Portとの連携によるウェイポイント制御

### シーケンス図

パイロット、DPオペレータ、WaypointController、NaviCtrl、FlightCtrl
の関係を時系列で示す。

### 配置図

ハードウェアの構成と、通信パケットを示す。


### 制御スキーム概要

プロポ入力で制御するnav modeと、ドローンポートで制御するdrone port modeを組み合わせ、flight modeを生成する。
Flight modeに基づき、適切なターゲット位置を計算し、FlightCtrlへ送信する。


![](http://g.gravizo.com/g?
digraph G {
  RCTransmitter -> FlightCtrl [label="nav_mode_request"];
  FlightCtrl -> FromFlightCtrlBuffer [label="nav_mode_request"];
  DronePort -> FromDronePortBuffer [label="drone_port_mode_request"];
  FromFlightCtrlBuffer -> FCHandler [label="nav_mode_request"];
  FromDronePortBuffer -> DPHandler [label="drone_port_mode_request"];
  FCHandler -> UpdateFlightMode [label="nav_mode"];
  DPHandler -> UpdateFlightMode [label="drone_port_mode"];
  UpdateFlightMode -> GenerateTargetPosition [label="flight_mode"];
  FCHandler -> ToFlightCtrlBuffer [label="nav_mode"];
  GenerateTargetPosition -> ToFlightCtrlBuffer [label="target_position"];
  ToFlightCtrlBuffer -> FlightCtrl;
  DPHandler -> ToDronePortBuffer [label="drone_port_mode"];
  ToDronePortBuffer -> DronePort;
}
)

### 制御スキーム詳細：Flight Modeの決定過程

Nav Mode|Nav Mode Meaning|Drone Port Mode|Drone Port Mode Meaning|Function
--------|----------------|-------------|---------------------|--------
0|Off |-|-|マニュアル飛行
1|Hold|-|-|位置保持
2|Auto|0 (default)|-|NaviCtrl内蔵ウェイポイントによるウェイポイント制御(モーターオフの時は何も起こらない)
2|Auto|1|Disarm|モーターオフ
2|Auto|2|Arm|モーターアイドリング
2|Auto|3|DPHold|位置保持
2|Auto|4|DPWaypoint|Drone Portから受信したウェイポイントによるウェイポイント制御
2|Auto|5|TakeoffToDPHold|テイクオフ後上空2mで待機
2|Auto|6|TakeoffToDPWaypoint|テイクオフ後DPウェイポイントによるウェイポイント制御
2|Auto|7|Land|着陸

![](http://g.gravizo.com/g?
digraph G {
  node[shape="oval",style="solid"]
    AutoDisarm; AutoArm; AutoDPHold; AutoDPWaypoint; AutoTakeoffToDPHold; AutoTakeoffToDPWaypoint; AutoLand;
  node[shape="oval",style="filled"]
    Off; Hold; Auto;
  node[shape="diamond",style="solid"]
    nav_mode; drone_port_mode
  node[shape="box",style="solid"]
    RCTransmitter; DronePort;
  RCTransmitter -> nav_mode
  DronePort -> drone_port_mode
  nav_mode -> Off [label="0"]
  nav_mode -> Hold [label="1"]
  nav_mode -> drone_port_mode [label="2"]
  drone_port_mode -> Auto[label="0=default"]
  drone_port_mode -> AutoDisarm[label="1"]
  drone_port_mode -> AutoArm[label="2"]
  drone_port_mode -> AutoDPHold[label="3"]
  drone_port_mode -> AutoDPWaypoint[label="4"]
  drone_port_mode -> AutoTakeoffToDPHold[label="5"]
  drone_port_mode -> AutoTakeoffToDPWaypoint[label="6"]
  drone_port_mode -> AutoLand[label="7"]
  AutoTakeoffToDPHold -> AutoDPHold [label="takeoff complete"]
  AutoTakeoffToDPWaypoint -> AutoDPWaypoint [label="takeoff complete"]
  AutoLand -> AutoArm [label="touchdown"]
})

---
title: Multicopter Setup
---

# マルチコプター開発ガイド

## 概要
---

このガイドは、マルチコプターの開発に必要なハードウェア、ソフトウェア、通信プロトコル等の情報を網羅する目的で作成した。

記載情報は適宜更新してゆく予定である。

## カスタムファームウェア
---

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
---

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
---

FlightCtrlはUbuntuマシンから書き込む必要がある。

[Ubuntuマシンのセットアップ](ubuntu_setup.html)

## Raspberry Pi 3のセットアップ
---

NaviCtrlはRaspberry Pi 3上で動作する。

[Raspberry Piのセットアップ](rpi_setup.html)

## Linux Cheat Sheet
---

Linux共通(開発用Ubuntuマシン、Raspberry Pi)のチートシート

[Cheat Sheet](linux_cheat_sheet.html)

## 通信プロトコル
---

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
---

### シーケンス図

パイロット、DPオペレータ、WaypointController、NaviCtrl、FlightCtrl
の関係を時系列で示す。

![](http://g.gravizo.com/source/drone_port_sequence?https%3a%2f%2fraw%2egithubusercontent%2ecom%2fkurokis%2fkurokis%2egithub%2eio%2fmaster%2findex%2emd)
<details>
  <summary></summary>
  drone_port_sequence
  @startuml;
  actor Pilot as pilot;
  actor "Drone Port Operator" as dpo;
  actor "PC Operator" as pco;
  participant "WaypointCtrl" as wc;
  participant "NaviCtrl" as nc;
  participant "FlightCtrl" as fc;
  pco -> nc: power on;
  pco -> nc: connect with ssh;
  pilot -> fc: power on;
  pilot -> fc: calibrate sensors;
  activate fc;
  note right: aircraft ready;
  dpo -> wc: power on;
  dpo -> wc: initialize;
  activate wc;
  note right: drone port ready;
  fc --> pco: check flightctrl is ready;
  pco -> nc: start navictrl process;
  activate nc;
  note right: position control ready;
  pilot -> fc: set nav mode to auto;
  fc -> nc: nav mode request: auto;
  activate nc;
  note left: auto mode start;
  nc --> fc: nav mode: auto;
  dpo -> wc: disarm motors;
  wc -> nc: set dp mode: Disarm;
  dpo -> wc: arm motors;
  wc -> nc: set dp mode: Arm;
  dpo -> wc: takeoff to hold;
  wc -> nc: set dp mode: AutoTakeoffToDPHold;
  nc -> wc: hold above takeoff location;
  dpo -> wc: start waypoint control;
  wc -> nc: set dp mode: AutoDPWaypoint;
  dpo -> wc: emergency hold;
  wc -> nc: set dp mode: DPHold;
  nc --> wc: hold at current position;
  wc --> dpo: check area is safe;
  dpo -> wc: resume waypoint control;
  wc -> nc: set dp mode: AudoDPWaypoint;
  nc --> wc: hold at last waypoint;
  wc --> dpo: check area is safe;
  dpo -> wc: land;
  wc -> nc: set dp mode: AutoLand;
  nc --> wc: automatic switch to Arm;
  wc --> dpo: check aircraft has landed;
  dpo -> wc: disarm;
  wc -> nc: set dp mode: Disarm;
  pilot -> fc: set nav mode to off;
  fc -> nc: nav mode request: off;
  deactivate nc;
  nc --> fc: nav mode: ;
  deactivate wc;
  deactivate nc;
  deactivate fc;
  @enduml
  drone_port_sequence
</details>

### 配置図

ハードウェアの構成と、通信パケットを示す。


### 制御スキーム概要

プロポで操作するnav mode requestと、ドローンポートで操作するdrone port mode requestに応じて制御モードやターゲット位置が計算される。
特に問題がなければリクエストはそのまま承諾されるが、異常時には機体の判断でリクエストと異なるモードが実行される。ここでいう異常とは以下のことである。

- カメラやGPSから位置情報が得られず、自律飛行が継続できない
- プロポのスティックを操作したことにより、自律飛行が強制解除された
- プロポからの信号をロストした(Go Homeモード、未実装)
- etc.

Nav modeは機体の制御モードを司り、Off、Hold、Autoの3種類がある。
Drone port modeはnav modeがAutoの時にのみ効力を持ち、ターゲット位置の計算やスロットル操作などを司る。Nav modeがAutoの時にのみ有効なのは、安全の観点から常にプロポの入力を優先させるためである。

Nav modeとdrone port modeに応じた機体の行動を下表に示す。

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

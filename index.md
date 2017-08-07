---
title: Multicopter Setup
---

# マルチコプター開発ガイド

## 概要

このガイドは、マルチコプターの開発に必要なハードウェア、ソフトウェア、通信プロトコル等の情報を網羅する目的で作成した。

記載情報は適宜更新してゆく予定である。

### カスタムファームウェア

FlightCtrl: https://github.com/kurokis/FlightCtrl2

NaviCtrl: https://github.com/Akihiro-K/RasPiMain


### ハードウェア構成
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
  }
)

#### WaypointControllerとNaviCtrlの接続について

WaypointControllerとNaviCtrlの通信はUARTであるが、NaviCtrl側の接続はRaspberry PiのGPIOピンではなく**USBポート**を使う。これは、NaviCtrlのGPIOピンがFlightCtrlとの通信で占有されているからである。一方、WaypointController側の接続はGPIOピンを使う。

~~通信にはMONOSTICKを使うことを考えている。MONOSTICKはUSBポートに差し込むことでデバイス間の無線シリアル通信を可能にする。~~

~~[MONOSTICK 公式ウェブサイト](https://mono-wireless.com/jp/products/MoNoStick/index.html)~~

~~有線での通信も検討しているが、USB->シリアル->USBという2重の変換が必要となるため、この変換用の制御ボードを別途作らなければならない可能性がある。構成のシンプルさを優先するため、現在はMONOSTICKの利用が有力と考えている。~~

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


## FlightCtrl

## NaviCtrl

### ライブラリ依存関係

![](http://g.gravizo.com/g?
  digraph G {
    aruco [shape=box3d]
    Eigen3 [shape=box3d]
    OpenCV2 [shape=box3d]
    raspicam [shape=box3d]
    disp [shape=box]
    logger [shape=box]
    mygps [shape=box]
    mymarker [shape=box]
    myserial [shape=box]
    mytcp [shape=box]
    navigator [shape=box]
    serial [shape=box]
    stateestimator [shape=box]
    shared [shape=box]
    RasPiMain [shape=Msquare]
    Marker [shape=Msquare]
    GPSServer [shape=Msquare]
    aruco -> mymarker
    Eigen3 -> mymarker
    Eigen3 -> stateestimator
    OpenCV2 -> mymarker
    raspicam -> Marker
    disp -> RasPiMain
    logger -> RasPiMain
    mygps -> RasPiMain
    mygps -> GPSServer
    mymarker -> Marker
    myserial -> RasPiMain
    mytcp -> RasPiMain
    mytcp -> Marker
    mytcp -> GPSServer
    navigator -> RasPiMain
    serial -> mygps
    serial -> myserial;
    stateestimator -> RasPiMain
    shared -> disp
    shared -> logger
    shared -> navigator
    shared -> stateestimator
    shared -> RasPiMain
    Marker -> RasPiMain
    GPSServer -> RasPiMain
  }
)

## Waypoints

詳細はTBD

![](http://g.gravizo.com/g?
  digraph G {
    subgraph cluster_2{
      label = "Route Manager"
      subgraph cluster_0 {
        label = "Route 0"
        wp00[label="waypoint 0"]
        wp01[label="waypoint 1"]
        wp02[label="waypoint 2"]
        wp03[label="waypoint 3"]
        wp00 -> wp01[label="edge 1"]
        wp01 -> wp02[label="edge 2"]
        wp02 -> wp03[label="edge 3"]
      }
      subgraph cluster_1 {
        label = "Route 1"
        wp10[label="waypoint 0"]
        wp11[label="waypoint 1"]
        wp12[label="waypoint 2"]
        wp13[label="waypoint 3"]
        wp10 -> wp11[label="edge 1"]
        wp11 -> wp12[label="edge 2"]
        wp12 -> wp13[label="edge 3"]
      }
    }
  }
)

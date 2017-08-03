---
title: Multicopter Setup
---

# マルチコプター開発ガイド

## 概要

このガイドは、マルチコプターの開発に必要なハードウェア、ソフトウェア、通信プロトコル等の情報を網羅的にカバーする目的で作成した。

記載情報は適宜更新してゆく予定である。

### ハードウェア構成
![](http://g.gravizo.com/g?
  digraph G {
    DronePort [shape=box]
    WaypointController [shape=box]
    GPS [shape=box]
    Camera [shape=box];NaviCtrl [shape=box]
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

WaypointControllerとNaviCtrlの通信はUARTであるが、物理的な接続はRaspberry PiのGPIOピンではなく**USBポート**を使う。これは、NaviCtrlのGPIOピンがFlightCtrlとの通信で占有されているからである。

通信にはMONOSTICKを使うことを考えている。MONOSTICKはUSBポートに差し込むことでデバイス間の無線シリアル通信を可能にする。

[MONOSTICK 公式ウェブサイト](https://mono-wireless.com/jp/products/MoNoStick/index.html)

有線での通信も検討しているが、USB->シリアル->USBという2重の変換が必要となるため、この変換用の制御ボードを別途作らなければならない可能性がある。構成のシンプルさを優先するため、現在はMONOSTICKの利用が有力と考えている。

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

今後記載予定

## FlightCtrl

## NaviCtrl

### Communication Payload
![](http://g.gravizo.com/g?
  digraph G {
    FlightCtrl [shape=box]
    MainProcess [shape=record]
    MarkerProcess [shape=box]
    GPSProcess [shape=box]
    StateEstimator
    Logging
    WaypointController -> MainProcess [label="FromWaypointController"]
    MainProcess -> WaypointController [label="ToWaypointController"]
    FlightCtrl -> MainProcess [label="FromFlightCtrl"]
    MainProcess -> FlightCtrl [label="ToFlightCtrl"]
    MarkerProcess -> MainProcess [label="MarkerPacket"]
    GPSProcess -> MainProcess [label="GPSPacket"]
    MainProcess -> StateEstimator [label="shared memory"]
    StateEstimator -> MainProcess
    MainProcess -> Logging [label="shared memory"]
    Logging -> MainProcess
  }
)

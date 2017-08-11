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

### 制御アルゴリズム

高レベル制御コマンドとしてターゲット位置、ターゲット方位およびこれらの最大遷移速度を受けつけ、低レベル制御コマンドの角加速度コマンドとz機軸加速度コマンドを生成する。
特徴として、中間コマンドに各速度コマンドと機体座標における重力ベクトルのコマンドを保持すること、姿勢表現にクオータニオンを用いていることが挙げられる。

この制御機構は遷移速度v_transitと遷移角速度psidot_transitによってターゲットが遠く離れている場合に制御コマンドを制限できるようになっているが、v_transitとpsidot_transitが無限大の極限でx,y,z,psi方向が互いに独立な状態量フィードバック制御に帰着する。したがって制御器の設計には極配置法や最適レギュレータなどの手法を利用することができる。

![](http://g.gravizo.com/g?
digraph G {
  subgraph cluster_0 {
    x_target
    y_target
    z_target
    v_transit
    psi_target
    psidot_transit
    label = "original commands";
  }
  subgraph cluster_1 {
    gbx_target
    gby_target
    p_command
    q_command
    r_command
    label = "intermediate commands";
  }
  subgraph cluster_2 {
    pdot_command
    qdot_command
    rdot_command
    wdot_command
    label = "control commands";
  }
  u_command[label="u_command=0"]
  v_command[label="v_command=0"]
  w_command[label="w_command=0"]
  form_angular_rate_command[shape=box]
  form_psi_command[shape=box]
  psi_target -> form_psi_command;
  psidot_transit -> form_psi_command;
  form_psi_command -> psi_command;
  psidot_transit -> form_angular_rate_command
  form_angular_rate_command -> p_command
  form_angular_rate_command -> q_command
  form_angular_rate_command -> r_command
  form_position_command[shape=box]
  x_target -> form_position_command
  y_target -> form_position_command
  z_target -> form_position_command
  v_transit -> form_position_command
  form_position_command -> x_command
  form_position_command -> y_command
  form_position_command -> z_command
  x_command -> gbx_target[label="kx"]
  y_command -> gby_target[label="ky"]
  u_command -> gbx_target[label="ku"]
  v_command -> gby_target[label="kv"]
  gbx_target -> theta_command
  gby_target -> phi_command
  q_command -> qdot_command[label="kq"]
  p_command -> pdot_command[label="kp"]
  r_command -> rdot_command[label="kr"]
  theta_command -> qdot_command[label="ktheta"]
  phi_command -> pdot_command[label="kphi"]
  psi_command -> rdot_command[label="kpsi"]
  z_command -> wdot_command[label="kz"]
  w_command -> wdot_command[label="kw"]
  pdot_command -> pdot_command[label="kpdot"]
  qdot_command -> qdot_command[label="kqdot"]
  wdot_command -> wdot_command[label="kwdot"]
})

## NaviCtrl

### ライブラリ依存関係

![](http://g.gravizo.com/g?
  digraph G {
    aruco [shape=box3d];
    Eigen3 [shape=box3d];
    OpenCV2 [shape=box3d];
    raspicam [shape=box3d];;
    disp [shape=box];
    gps [shape=box];
    logger [shape=box];
    marker [shape=box];
    tcp [shape=box];
    navigator [shape=box];
    nc [shape=box];
    serial [shape=box];
    stateestimator [shape=box];
    shared [shape=box];
    utserial [shape=box];
    RasPiMain [shape=Msquare];
    Marker [shape=Msquare];
    GPSServer [shape=Msquare];
    aruco -> marker;
    Eigen3 -> marker;
    Eigen3 -> stateestimator;
    OpenCV2 -> marker;
    raspicam -> Marker;
    disp -> nc;
    gps -> RasPiMain;
    gps -> GPSServer;
    logger -> nc;
    marker -> Marker;
    navigator -> nc;
    nc -> RasPiMain;
    serial -> gps;
    serial -> utserial;
    stateestimator -> nc;
    shared -> disp;
    shared -> logger;
    shared -> navigator;
    shared -> stateestimator;
    tcp -> RasPiMain;
    tcp -> Marker;
    tcp -> GPSServer;
    utserial -> RasPiMain;
    Marker -> RasPiMain;
    GPSServer -> RasPiMain;
  }
)

## Waypoints

詳細はTBD

![](http://g.gravizo.com/g?
  digraph G {
    subgraph cluster_2{
      label = "Route Manager";
      subgraph cluster_0 {
        label = "Route 0";
        wp00[label="waypoint 0"];
        wp01[label="waypoint 1"];
        wp02[label="waypoint 2"];
        wp03[label="waypoint 3"];
        wp00 -> wp01[label="edge 1"];
        wp01 -> wp02[label="edge 2"];
        wp02 -> wp03[label="edge 3"];
      }
      subgraph cluster_1 {
        label = "Route 1";
        wp10[label="waypoint 0"];
        wp11[label="waypoint 1"];
        wp12[label="waypoint 2"];
        wp13[label="waypoint 3"];
        wp10 -> wp11[label="edge 1"];
        wp11 -> wp12[label="edge 2"];
        wp12 -> wp13[label="edge 3"];
      }
    }
  }
)

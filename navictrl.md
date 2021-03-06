# NaviCtrl

## ライブラリ依存関係

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
    MarkerServer [shape=Msquare];
    GPSServer [shape=Msquare];
    aruco -> marker;
    Eigen3 -> marker;
    Eigen3 -> stateestimator;
    OpenCV2 -> marker;
    raspicam -> MarkerServer;
    disp -> nc;
    gps -> RasPiMain;
    gps -> GPSServer;
    logger -> nc;
    marker -> MarkerServer;
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
    tcp -> MarkerServer;
    tcp -> GPSServer;
    utserial -> RasPiMain;
    MarkerServer -> RasPiMain;
    GPSServer -> RasPiMain;
  }
)

## Waypoints

詳細はTBD

![](https://g.gravizo.com/source/route_manager?https%3a%2f%2fraw%2egithubusercontent%2ecom%2fkurokis%2fkurokis%2egithub%2eio%2fmaster%2fnavictrl%2emd)
<details>
<summary></summary>
route_manager
digraph G {
  subgraph cluster_0{
    label = "Route Manager";
    subgraph cluster_1 {
      label = "Route 0 (from file)";
      style = "filled";
      wp00[label="waypoint 0"];
      wp01[label="waypoint 1"];
      wp02[label="waypoint 2"];
      wp03[label="waypoint 3"];
      wp00 -> wp01;
      wp01 -> wp02;
      wp02 -> wp03;
    }
    subgraph cluster_2 {
      label = "Route 1 (from file)";
      style= "filled";
      wp10[label="waypoint 0"];
      wp11[label="waypoint 1"];
      wp12[label="waypoint 2"];
      wp10 -> wp11;
      wp11 -> wp12;
    }
    subgraph cluster_3 {
      label = "Route 2 (from Drone Port)";
      style= "solid";
      wp20[label="waypoint 0"];
      wp21[label="waypoint 1"];
      wp22[label="waypoint 2"];
      wp23[label="waypoint 3"];
      wp20 -> wp11;
      wp21 -> wp12;
      wp22 -> wp13;
    }
  }
}
route_manager
</details>

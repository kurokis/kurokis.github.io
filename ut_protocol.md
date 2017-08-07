# UT Protocol

## 概要

FlightCtrl, NaviCtrl, WaypointController間の通信はすべてUT Protocolに基づくシリアル通信で行う。

Settings|
--------|
Baudrate|57600
Bits|8
Parity|None
Stop bits|1
Byte order|Little endian

UT Protocolは下表で定義される。

Name|Type|Bytes|Meanings
----|----|-----|--------
Start character|char    |1|'S' (0x53)
Payload length |uint8_t |1|
ID             |uint8_t |1|
Unused         |uint8_t |1|0x0
Payload        |misc.   |N|
CRC            |uint16_t|2|CRC-16-CCITT with both input and output reflected

## WaypointController-NaviCtrl間の通信ペイロード

## ID一覧

ID|From|To|Function
--|----|--|--------
 1|NaviCtrl  |FlightCtrl|Send data for position control
 1|FlightCtrl|NaviCtrl  |Send data for logging
10|Drone Port|NaviCtrl  |Downlink request
10|NaviCtrl  |Drone Port|Downlink data
11|Drone Port|NaviCtrl  |Set navigation mode
11|NaviCtrl  |Drone Port|Set navigation mode response
12|Drone Port|NaviCtrl  |Set waypoint
12|NaviCtrl  |Drone Port|Set waypoint response

## ペイロード詳細

ID = 10, NaviCtrl -> FlightCtrl, Downlink request

Name|Type|Bytes|Meanings
----|----|-----|--------
-|-|0|-
||0|

ID = 10, FlightCtrl -> NaviCtrl, Downlink data

Name|Type|Bytes|Meanings
----|----|-----|--------
nav_mode|uint8_t|1|OFF: 0, HOLD: 1, AUTO: 2, HOME: 3
nav_status|uint8_t|1|000abcde
waypoint_status|uint8_t|1|TBD
gps_status|uint8_t|1|TBD
position|float[3]|12|position in meters
velocity|float[3]|12|velocity in m/s
quaternion|float[4]|16|attitude quaternion [q0,qx,qy,qz]
||44|

- a: NAV_STATUS_BIT_POSITION_RESET_REQUEST
- b: NAV_STATUS_BIT_LOW_PRECISION_VERTICAL
- c: NAV_STATUS_BIT_VELOCITY_DATA_OK
- d: NAV_STATUS_BIT_POSITION_DATA_OK
- e: NAV_STATUS_BIT_HEADING_DATA_OK"

ID = 11, Drone Port -> NaviCtrl, Set navigation mode

Name|Type|Bytes|Meanings
----|----|-----|--------
write_data|uint8_t|1|0: read-only, 1: read-write
nav_mode_request|uint8_t|1|OFF: 0, HOLD: 1, AUTO: 2, HOME: 3, TAKEOFF_TO_HOLD: 4, LAND: 5, ARM: 6
||2|

ID = 11, NaviCtrl -> Drone Port, Set navigation mode response

Name|Type|Bytes|Meanings
----|----|-----|--------
nav_mode|uint8_t|1|OFF: 0, HOLD: 1, AUTO: 2, HOME: 3, TAKEOFF_TO_HOLD: 4, LAND: 5, ARM: 6
unused|uint8_t|1|
||2|

ID = 12, Drone Port -> NaviCtrl, Set waypoint

Name|Type|Bytes|Meanings
----|----|-----|--------
write_data|uint8_t|1|0: read-only, 1: read-write
route_number|uint8_t|1|0,1,2,3
number_of_waypoints|uint8_t|1|
waypoint_number|uint8_t|1|
wait_ms|uint16_t|2|milliseconds to stay at waypoint
target_longitude|float|4|longitude in deg
target_latitude|float|4|latitude in deg
target_altitude|float|4|height from ground in meters (upward positive)
transit_speed|float|4|max transit speed to reach this waypoint in m/s
radius|float|4|max allowable position error in meters
target_heading|float|4|target heading in deg (clockwise from north, 0-360)
heading_rate|float|4|max heading rate in deg/s
heading_range|float|4|max allowable heading error in deg
||38|

ID = 12, NaviCtrl -> Drone Port, Set waypoint response

Name|Type|Bytes|Meanings
----|----|-----|--------
number_of_waypoints_missing|uint8_t[4]|4|number of waypoints missing for each of 4 routes
waypoint_number_missing|uint8_t[4]|4|smallest index of waypoint missing for each of 4 routes
||8|

## FlightCtrl-NaviCtrl間の通信ペイロード

FlightCtrl -> NaviCtrl (FromFlightCtrl / ToNaviCtrl)

```c
struct FromFlightCtrl {
  uint16_t timestamp;
  uint8_t nav_mode_request;
  uint8_t flightctrl_state;
  float accelerometer[3];
  float gyro[3];
  float quaternion[4];
  float pressure_alt;
} __attribute__((packed));
```

NaviCtrl -> FlightCtrl (ToFlightCtrl / FromNaviCtrl)

```c
struct ToFlightCtrl {
  uint16_t version;
  uint8_t nav_mode;
  uint8_t navigation_status;
  float position[3]; // meter
  float velocity[3];
  float quat0;
  float quatz;
  float target_position[3];
  float transit_vel;
  float target_heading;
  float heading_rate;
} __attribute__((packed));
```

## その他 (Serial)

```c
struct ToDronePort {
  uint8_t nav_mode;
  uint8_t nav_status;
  uint8_t waypoint_status;
  uint8_t gps_status;
  float position[3];
  float velocity[3];
  float quaternion[4];
} __attribute__((packed));
```

## その他　(TCP)

```c
struct FromMarker {
  uint32_t timestamp; // microseconds
  float position[3]; // meter
  float quaternion[3]; // x y z
  float r_var[3]; // meter^2
  uint8_t status; // 1 : detected, 0 : not detected
} __attribute__((packed));
```

```c
struct FromGPS {
  float position[3];
  float velocity[3];
  float r_var[3];
  float v_var[3];
  uint8_t status; // 0x01: pos OK 0x02: vel OK
} __attribute__((packed));
```

```c
struct FromLSM {
  float mag[3];
  uint8_t status; // 1: OK 0: unavailable
} __attribute__((packed));
```

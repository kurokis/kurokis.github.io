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

ID|From|To|Name|Type|Bytes|Meanings
--|----|--|----|----|-----|--------
10|DP|NC|-|-|0|-
|||||0|

ID|From|To|Name|Type|Bytes|Meanings
--|----|--|----|----|-----|--------
10|NC|DP|nav_mode|uint8_t|1|OFF: 0, HOLD: 1, AUTO: 2, HOME: 3
10|NC|DP|nav_status|uint8_t|1|000abcde
10|NC|DP|waypoint_status|uint8_t|1|TBD
10|NC|DP|gps_status|uint8_t|1|TBD
10|NC|DP|position|float[3]|12|position in meters
10|NC|DP|velocity|float[3]|12|velocity in m/s
10|NC|DP|quaternion|float[4]|16|attitude quaternion [q0,qx,qy,qz]
|||||44|

- a: NAV_STATUS_BIT_POSITION_RESET_REQUEST
- b: NAV_STATUS_BIT_LOW_PRECISION_VERTICAL
- c: NAV_STATUS_BIT_VELOCITY_DATA_OK
- d: NAV_STATUS_BIT_POSITION_DATA_OK
- e: NAV_STATUS_BIT_HEADING_DATA_OK"

ID|From|To|Name|Type|Bytes|Meanings
--|----|--|----|----|-----|--------
11|DP|NC|write_data|uint8_t|1|0: read-only, 1: read-write
11|DP|NC|nav_mode_request|uint8_t|1|OFF: 0, HOLD: 1, AUTO: 2, HOME: 3, TAKEOFF_TO_HOLD: 4, LAND: 5, ARM: 6
|||||2|

ID|From|To|Name|Type|Bytes|Meanings
--|----|--|----|----|-----|--------
11|NC|DP|nav_mode|uint8_t|1|OFF: 0, HOLD: 1, AUTO: 2, HOME: 3, TAKEOFF_TO_HOLD: 4, LAND: 5, ARM: 6
11|NC|DP|unused|uint8_t|1|
|||||2|

ID|From|To|Name|Type|Bytes|Meanings
--|----|--|----|----|-----|--------
12|DP|NC|write_data|uint8_t|1|0: read-only, 1: read-write
12|DP|NC|route_number|uint8_t|1|0,1,2,3
12|DP|NC|number_of_waypoints|uint8_t|1|
12|DP|NC|waypoint_number|uint8_t|1|
12|DP|NC|wait_ms|uint16_t|2|milliseconds to stay at waypoint
12|DP|NC|target_longitude|float|4|longitude in deg
12|DP|NC|target_latitude|float|4|latitude in deg
12|DP|NC|target_altitude|float|4|height from ground in meters (upward positive)
12|DP|NC|transit_speed|float|4|max transit speed to reach this waypoint in m/s
12|DP|NC|radius|float|4|max allowable position error in meters
12|DP|NC|target_heading|float|4|target heading in deg (clockwise from north, 0-360)
12|DP|NC|heading_rate|float|4|max heading rate in deg/s
12|DP|NC|heading_range|float|4|max allowable heading error in deg
|||||38|

ID|From|To|Name|Type|Bytes|Meanings
--|----|--|----|----|-----|--------
12|NC|DP|number_of_waypoints_missing|uint8_t[4]|4|number of waypoints missing for each of 4 routes
12|NC|DP|waypoint_number_missing|uint8_t[4]|4|smallest index of waypoint missing for each of 4 routes
|||||8|

## FlightCtrl-NaviCtrl間の通信ペイロード

TBD

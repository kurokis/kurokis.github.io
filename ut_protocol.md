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
Sequence number         |uint8_t |1|currently 0, to be implemented
Payload        |misc.   |N|
CRC            |uint16_t|2|CRC-16-CCITT with both input and output reflected

CRC-16-CCITTの計算方法は以下を参照(crc16.h)。

```c
#include <inttypes.h>

// This is the same CRC that is used in MAVLink. It uses a polynomial
// represented by 0x1021 with both the input and output reflected. The CRC
// should be initialized to 0xFFFF.
static inline uint16_t CRCUpdateCCITT(uint16_t crc, uint8_t data)
{
  data ^= (uint8_t)(crc & 0xFF);
  data ^= (data << 4);
  return (crc >> 8) ^ (data << 8) ^ (data << 3) ^ (data >> 4);
}

// -----------------------------------------------------------------------------
// This function computes the CRC for an array of bytes.
static inline uint16_t CRCCCITT(const uint8_t * array, size_t length)
{
  uint16_t crc = 0xFFFF;
  for (size_t i = length; i--; ) crc = CRCUpdateCCITT(crc, *(array++));
  return crc;
}
```


## WaypointController-NaviCtrl間の通信ペイロード

## ID一覧
---

ID|From|To|Function
--|----|--|--------
 1|NaviCtrl  |FlightCtrl|Send data for position control
 1|FlightCtrl|NaviCtrl  |Send data for logging
10|NaviCtrl  |Drone Port|Downlink
11|Drone Port|NaviCtrl  |Set drone port mode
11|NaviCtrl  |Drone Port|Set drone port mode response
12|Drone Port|NaviCtrl  |Set waypoint
12|NaviCtrl  |Drone Port|Set waypoint response

## 各種変数の意味

  - nav_mode / nav_mode_request

    ```c
    enum NavMode {
      NAV_MODE_OFF = 0,
      NAV_MODE_HOLD = 1,
      NAV_MODE_AUTO = 2,
      NAV_MODE_HOME = 3,
    };
    ```
  - drone_port_mode / drone_port_mode_request

    ```c
    enum DPMode {
      NCWaypoint = 0,
      Disarm = 1,
      Arm = 2,
      DPHold = 3,
      DPWaypoint = 4,
      Takeoff = 5, // Takeoff then hold
      Land = 6,
    };
    ```

  - nav_status

    ```c
    enum NavStatusBits {
      HeadingOK = 1<<0,
      PositionOK = 1<<1,
      VelocityOK = 1<<2,
      LOW_PRECISION_VERTICAL = 1<<3,
      POSITION_RESET_REQUEST = 1<<4,
      // TODO: ON_GROUND
    };
    ```

  - drone_port_status

    ```c
    enum DPStatus {
      DPStatusModeInProgress = 0,
      DPStatusEndOfMode = 1,
    };
    ```

## ペイロード詳細
---

ID = 10, NaviCtrl -> Drone Port, Downlink

リクエストに応答するのではなく、NaviCtrlから定期的に送信する方式に変更。送信レートは暫定的に2Hzとする（要検討）。

Name|Type|Bytes|Meanings
----|----|-----|--------
nav_mode|uint8_t|1|
drone_port_mode|uint8_t|1|
nav_status|uint8_t|1|
drone_port_status|uint8_t|1|
position|float[3]|12|position in meters
velocity|float[3]|12|velocity in m/s
quaternion|float[4]|16|attitude quaternion [q0,qx,qy,qz]
||44|


```c
struct ToDronePort {
  uint8_t nav_mode;
  uint8_t drone_port_mode;
  uint8_t nav_status;
  uint8_t drone_port_status;
  float position[3];
  float velocity[3];
  float quaternion[4];
} __attribute__((packed));
```

---

ID = 11, Drone Port -> NaviCtrl, Set drone port mode

Name|Type|Bytes|Meanings
----|----|-----|--------
write_data|uint8_t|1|0: read-only, 1: write
drone_port_mode_request|uint8_t|1|
||2|

```c
struct FromDPSetDronePortMode{
  uint8_t read_write; // 0: read-only, 1: write
  uint8_t drone_port_mode_request;
} __attribute__((packed));
```

---

ID = 11, NaviCtrl -> Drone Port, Set drone port mode response

Name|Type|Bytes|Meanings
----|----|-----|--------
drone_port_mode|uint8_t|1|
drone_port_status|uint8_t|1|
||2|

```c
struct ToDPSetDronePortMode {
  uint8_t drone_port_mode;
  uint8_t drone_port_status;
} __attribute__((packed));
```

---

ID = 12, Drone Port -> NaviCtrl, Set waypoint

Name|Type|Bytes|Meanings
----|----|-----|--------
write_data|uint8_t|1|0: read-only, 1: write
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

---

ID = 12, NaviCtrl -> Drone Port, Set waypoint response

Name|Type|Bytes|Meanings
----|----|-----|--------
number_of_waypoints_missing|uint8_t[4]|4|number of waypoints missing for each of 4 routes
waypoint_number_missing|uint8_t[4]|4|smallest index of waypoint missing for each of 4 routes
||8|

---

ID = 13, Drone Port -> NaviCtrl, Position

Name|Type|Bytes|Meanings
----|----|-----|--------
timestamp|uint32_t|4|timestamp in microseconds
position|float[4]|16|position in meters, North-East-Down coordinates
quaternion|float[3]|12|vector part of quaternion
r_var|float[3]|12|position variance in m^2
status|uint8_t|1|1: detected, 0: not detected
||45|

```c
struct FromDronePort {
  uint32_t timestamp; // microseconds
  float position[3]; // meter
  float quaternion[3]; // x y z
  float r_var[3]; // meter^2
  uint8_t status; // 1 : detected, 0 : not detected
} __attribute__((packed));
```


## FlightCtrl-NaviCtrl間の通信ペイロード
---

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

## その他　(TCP)
---


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
  int32_t longitude; // [10^-6 deg]
  int32_t latitude; // [10^-6 deg]
  float z; // height above sea level [m], downward positive
  float velocity[3]; // [m/s]
  uint8_t gps_status; // 3: pos & vel OK 2: only pos OK 1: only vel OK 0: unavailable
} __attribute__((packed));
```

```c
struct FromLSM {
  float mag[3];
  uint8_t status; // 1: OK 0: unavailable
} __attribute__((packed));
```

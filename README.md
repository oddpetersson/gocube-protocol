# GoCube BLE Protocol Documentation
GoCube is a bluetooth enabled Rubik's cube. This document contains a partial description of the GoCube BLE protocol.

This document describes the general format of the notifications sent from the cube and the specifics for a subset of these notifications.

**TODO:**
- Figure out how to turn on the backlight of the cube

# GATT Service
| Service             | UUID |
| ------------------- | ---- |
| Nordic UART Service | 6e400003-b5a3-f393-e0a9-e50e24dcca9e |

# Notifications
This section describes the format of the notification characteristic value.

## Common Message Format
| Byte Offset | Length (Bytes) | Format  | Name     | Description |
| ----------- | -------------- | ------- | -------- | ----------- |
| 0           | 1              | Uint8   | Prefix   | Fixed value: 0x2A (*) | 
| 1           | 1              | Uint8   | Length   | Length excluding Suffix |
| 2           | 1              | Uint8   | Type     | Message Type |
| 3           | Length-4       | Uint8[] | Message  | Message |
| Length-1    | 1              | Uint8   | Checksum | Sum of byte 0 .. (Length-2) % 256 |
| Length      | 2              | Uint8[] | Suffix   | Fixed value: [0x0D, 0x0A] (CRLF) |

> **Example:**
> | Byte Offset | Value | Description |
> | ----------- | ----- | ----------- |
> | 0           | 0x2A  | Prefix: asterisk (*) |
> | 1           | 0x05  | Length: 5 bytes |
> | 2           | 0x05  | Message Type: MsgBatteryLevel |
> | 3           | 0x38  | Message: Battery level: 56% |
> | 4           | 0x6C  | Checksum: (0x2A + 0x05 + 0x05 + 0x38) % 256 |
> | 5           | 0x0D  | Suffix: CR |
> | 6           | 0x0A  | Suffix: LF |

## Message Types
| Type | Name |
| ---- | ---- |
| 0x01 | MsgRotatingSide |
| 0x02 | MsgCubeColorAndDirectionState |
| 0x03 | MsgQuaternion |
| 0x04 | MsgQuaternionShort |
| 0x05 | MsgBatteryLevel |
| 0x06 | ? |
| 0x07 | MsgOffLineStats |
| 0x08 | MsgIsEdgeCube |
| 0x09 | ? |
| 0x0A | MsgCubeInfo |

## Message Type 0x05: MsgBatteryLevel
| Byte Offset | Length (Bytes) | Format  | Name         | Description |
| ----------- | -------------- | ------- | ------------ | ----------- |
| 0           | 1              | Uint8   | BatteryLevel | <p>Current battery level percentage.<br>Range: 0x00 - 0x64 (0 - 100)</p> |

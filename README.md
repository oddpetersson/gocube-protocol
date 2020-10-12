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

## Message Type 0x01: MsgRotatingSide
**Message Length:** 2 bytes

| Byte Offset | Length (Bytes) | Format  | Name          | Description |
| ----------- | -------------- | ------- | ------------- | ----------- |
| 0           | 1              | Uint8   | FaceRotation  | Face and rotation |
| 1           | 1              | Uint8   | Orientation   | Center piece orientation |

| FaceRotation | Binary Value  | Color   | Rotation |
| ------------ | ------------  | -----   | -------- |
| 0x00         | ``0000 0000`` | Blue    | Clockwise |
| 0x01         | ``0000 0001`` | Blue    | Counterclockwise |
| 0x02         | ``0000 0010`` | Green   | Clockwise |
| 0x03         | ``0000 0011`` | Green   | Counterclockwise |
| 0x04         | ``0000 0100`` | White   | Clockwise |
| 0x05         | ``0000 0101`` | White   | Counterclockwise |
| 0x06         | ``0000 0110`` | Yellow  | Clockwise |
| 0x07         | ``0000 0111`` | Yellow  | Counterclockwise |
| 0x08         | ``0000 1000`` | Red     | Clockwise |
| 0x09         | ``0000 1001`` | Red     | Counterclockwise |
| 0x0A         | ``0000 1010`` | Orange  | Clockwise |
| 0x0B         | ``0000 1011`` | Orange  | Counterclockwise |

| Orientation | Description |
| ----------- | ----------- |
| 0x00        | Twelve o'clock |
| 0x03        | Three o'clock |
| 0x06        | Six o'clock |
| 0x09        | Nine o'clock |

> **Example:**
> Prerequisite: Yellow side facing up, green side facing you
>
> | Notation | FaceRotation | Orientation | Description |
> | -------- | ------------ | ----------- | ----------- |
> | F        | 0x02         | 0x09        | Green face clockwise rotation, center piece orientation nine o'clock |
> | R        | 0x0A         | 0x00        | Orange face clockwise rotation, center piece orientation twelve o'clock |
> | U        | 0x06         | 0x09        | Yellow face clockwise rotation, center piece orientation nine o'clock |
> | L        | 0x08         | 0x00        | Red face clockwise rotation, center piece orientation twelve o'clock |
> | B        | 0x00         | 0x06        | Blue face clockwise rotation, center piece orientation six o'clock |
> | D        | 0x04         | 0x00        | Yellow face clockwise rotation, center piece orientation twelve o'clock |
> | F'       | 0x03         | 0x06        | Green face counterclockwise rotation, center piece orientation six o'clock |
> | R'       | 0x0B         | 0x09        | Orange face counterclockwise rotation, center piece orientation nine o'clock |
> | U'       | 0x07         | 0x06        | Yellow face counterclockwise rotation, center piece orientation six o'clock |
> | L'       | 0x09         | 0x09        | Red face counterclockwise rotation, center piece orientation nine o'clock |
> | B'       | 0x01         | 0x03        | Blue face counterclockwise rotation, center piece orientation three o'clock |
> | D'       | 0x05         | 0x09        | Yellow face counterclockwise rotation, center piece orientation nine o'clock |

## Message Type 0x05: MsgBatteryLevel
**Message Length:** 1 byte

| Byte Offset | Length (Bytes) | Format  | Name         | Description |
| ----------- | -------------- | ------- | ------------ | ----------- |
| 0           | 1              | Uint8   | BatteryLevel | <p>Current battery level percentage<br>Range: 0x00 - 0x64 (0 - 100)</p> |

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

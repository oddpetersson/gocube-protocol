# GoCube BLE Protocol Documentation
GoCube is a bluetooth enabled Rubik's cube. This document contains a partial description of the GoCube BLE protocol.

# Overview
Setup communication:
- Scan for BLE devices that supports the Primary Service
- Connect to the primary service
- Enable notifications for the TX characteristic service

The following notifications are sent without requests:
- 3D tracking: 15 notifications per second (MsgOrientation). These notifications can be disabled or enabled
- Rotations: one message each time a face is rotated (MsgRotation)

The following notifications can be requested:
- Current battery level (MsgBattery)
- Current state (MsgState)
- Offline stats (MsgStats)
- Cube type (MsgCubeType)

Additional requests:
- Reboot
- Toggle backlight
- Calibrate 3D tracking
- Reset cube to solved state

# GATT Services
| Service             | Properties | UUID |
| ------------------- | ---------- | ---- |
| Primary Service     |            | 6e400001-b5a3-f393-e0a9-e50e24dcca9e |
| RX Characterstic    | W, WNR     | 6e400002-b5a3-f393-e0a9-e50e24dcca9e |
| TX Characterstic    | N          | 6e400003-b5a3-f393-e0a9-e50e24dcca9e |

# Requests
Requests supported by the RX Characterstic service.

## Request messages
Enable or disable MsgOrientation (3D tracking):

| Value | Command            |
| ----- | ------------------ |
| 0x37  | DisableOrientation |
| 0x38  | EnableOrientation  |

Request additional information from the cube:

| Value | Command         | Message Type |
| ----- | --------------- | -----------  |
| 0x32  | GetBattery      | MsgBattery   |
| 0x33  | GetState        | MsgState     |
| 0x39  | GetStats        | MsgStats     |
| 0x56  | GetCubeType     | MsgCubeType  |

## Configuration

| Value | Command              | Description |
| ----- | -------------------- | ----------- |
| 0x34  | Reboot               | Reboot the cube |
| 0x35  | SetSolvedState       | Resets the cube to solved state |
| 0x41  | LedFlash             | Flash the backlight three times |
| 0x42  | LedToggleAnimation   | Enable or disable animated backlight |
| 0x43  | LedFlashSlow         | Slowly flashes the backlight three times |
| 0x44  | LedToggle            | Toggle backlight |
| 0x57  | CalibrateOrientation | Calibrate 3D tracking |

# Notifications
This section describes the format of the notifications.

## Common Message Format
| Byte Offset | Length (Bytes) | Name     | Description |
| ----------- | -------------- | -------- | ----------- |
| 0           | 1              | Prefix   | Fixed value: 0x2A (*) | 
| 1           | 1              | Length   | Length excluding Suffix |
| 2           | 1              | Type     | Message Type |
| 3           | Length-4       | Message  | Message |
| Length-1    | 1              | Checksum | Sum of byte 0 .. (Length-2) % 0x100 |
| Length      | 2              | Suffix   | Fixed value: [0x0D, 0x0A] (CRLF) |

> **Example:**
> | Byte Offset | Value | Description |
> | ----------- | ----- | ----------- |
> | 0           | 0x2A  | Prefix: asterisk (*) |
> | 1           | 0x05  | Length: 5 bytes |
> | 2           | 0x05  | Message Type: MsgBattery |
> | 3           | 0x38  | Message: Battery level: 56% |
> | 4           | 0x6C  | Checksum: (0x2A + 0x05 + 0x05 + 0x38) % 0x100 |
> | 5           | 0x0D  | Suffix: CR |
> | 6           | 0x0A  | Suffix: LF |

## Message Types
| Type | Name |
| ---- | ---- |
| 0x01 | MsgRotation |
| 0x02 | MsgState |
| 0x03 | MsgOrientation |
| 0x05 | MsgBattery |
| 0x07 | MsgOffLineStats |
| 0x08 | MsgCubeType |

## Message Type 0x01: MsgRotation
**Message Length:** n * 2 bytes (Length - 4) (n = number of rotations)

| Message Offset | Length (Bytes) | Name          | Description |
| -------------- | -------------- | ------------- | ----------- |
| n              | 1              | FaceRotation  | Face and rotation |
| n + 1          | 1              | Orientation   | Center piece orientation |

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
>
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

## Message Type 0x02: MsgState
**Message Length:** 60 bytes (Length - 4)

| Message Offset | Length (Bytes) | Name          | Description |
| -------------- | -------------- | ------------- | ----------- |
| 0              | 1              | BlueCenter    | Constant value: 0x00 (blue) |
| 1              | 8              | BlueFace      | The colors visable on the blue face, starting top left going clockwise |
| 9              | 1              | GreenCenter   | Constant value: 0x01 (green) |
| 10             | 8              | GreenFace     | The colors visable on the green face, starting top left going clockwise |
| 18             | 1              | WhiteCenter   | Constant value: 0x02 (white) |
| 19             | 8              | WhiteFace     | The colors visable on the white face, starting top left going clockwise |
| 27             | 1              | YellowCenter  | Constant value: 0x03 (yellow) |
| 28             | 8              | YellowFace    | The colors visable on the yellow face, starting top left going clockwise |
| 36             | 1              | RedCenter     | Constant value: 0x04 (red) |
| 37             | 8              | RedFace       | The colors visable on the red face, starting top left going clockwise |
| 45             | 1              | OrangeCenter  | Constant value: 0x05 (orange) |
| 46             | 8              | OrangeFace    | The colors visable on the orange face, starting top left going clockwise |
| 54             | 6              | Orientation   | Orientation for each center piece: 0, 3, 6 or 9 o'clock |

| Value | Color |
| ----- | ----- |
| 0x00  | Blue |
| 0x01  | Green |
| 0x02  | White |
| 0x03  | Yellow |
| 0x04  | Red |
| 0x05  | Orange |

Message offset map:

<table>
  <tbody>
    <tr>
      <td colspan="3"></td><td colspan="3">White Face</td><td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>19</td><td>20</td><td>21</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>26</td><td>18</td><td>22</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>25</td><td>24</td><td>23</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3">Green Face</td><td colspan="3">Red Face</td><td colspan="3">Blue Face</td><td colspan="3">Orange Face</td>
    </tr>
    <tr>
      <td>10</td><td>11</td><td>12</td>
      <td>37</td><td>38</td><td>39</td>
      <td>1</td><td>2</td><td>3</td>
      <td>46</td><td>47</td><td>48</td>
    </tr>
    <tr>
      <td>17</td><td>9</td><td>13</td>
      <td>44</td><td>36</td><td>40</td>
      <td>8</td><td>0</td><td>4</td>
      <td>53</td><td>45</td><td>49</td>
    </tr>
    <tr>
      <td>16</td><td>15</td><td>14</td>
      <td>43</td><td>42</td><td>41</td>
      <td>7</td><td>6</td><td>5</td>
      <td>52</td><td>51</td><td>50</td>
    </tr>
    <tr>
      <td colspan="3"></td><td colspan="3">Yellow Face</td><td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>28</td><td>29</td><td>30</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>35</td><td>27</td><td>31</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>34</td><td>33</td><td>32</td>
      <td colspan="6"></td>
    </tr>
  </tbody>
</table>

**Example message, hex values:**

| BlueCenter + BlueFace      | GreenCenter + GreenFace    | WhiteCenter + WhiteFace    | YellowCenter + YellowFace  | RedCenter + RedFace        | OrangeCenter + OrangeFace  | Orientation       |
| -------------------------- | -------------------------- | -------------------------- | -------------------------- | -------------------------- | -------------------------- | ----------------- |
| 00-02-05-00-04-00-05-05-05 | 01-03-03-05-04-02-05-05-00 | 02-04-02-03-02-04-04-01-01 | 03-01-04-03-01-05-01-02-03 | 04-03-03-00-00-00-01-04-02 | 05-04-00-01-03-01-02-02-00 | 03-00-03-03-06-00 |

<table>
  <tbody>
    <tr>
      <td colspan="3"></td><td colspan="3">White Face</td><td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>R</td><td>W</td><td>Y</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>G</td><td>W</td><td>W</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>G</td><td>R</td><td>R</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3">Green Face</td><td colspan="3">Red Face</td><td colspan="3">Blue Face</td><td colspan="3">Orange Face</td>
    </tr>
    <tr>
      <td>Y</td><td>Y</td><td>O</td>
      <td>Y</td><td>Y</td><td>B</td>
      <td>W</td><td>O</td><td>B</td>
      <td>R</td><td>B</td><td>G</td>
    </tr>
    <tr>
      <td>B</td><td>G</td><td>R</td>
      <td>W</td><td>R</td><td>B</td>
      <td>O</td><td>B</td><td>R</td>
      <td>B</td><td>O</td><td>Y</td>
    </tr>
    <tr>
      <td>O</td><td>O</td><td>W</td>
      <td>R</td><td>G</td><td>B</td>
      <td>O</td><td>O</td><td>B</td>
      <td>W</td><td>W</td><td>G</td>
    </tr>
    <tr>
      <td colspan="3"></td><td colspan="3">Yellow Face</td><td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>G</td><td>R</td><td>Y</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>Y</td><td>Y</td><td>G</td>
      <td colspan="6"></td>
    </tr>
    <tr>
      <td colspan="3"></td>
      <td>W</td><td>G</td><td>O</td>
      <td colspan="6"></td>
    </tr>
  </tbody>
</table>

## Message Type 0x03: MsgOrientation
**Message Length:** variable

Current orientation of the cube, reported as a quaternion. Numeric values represented as ascii strings. Used for 3D tracking.

Format: x#y#z#w

> **Example:**
>
> | x                   | delimeter | y              | delimeter | z                             | delimeter | w                   |
> | ------------------- | --------- | -------------- | --------- | ----------------------------- | --------- | ------------------- |
> | 0x39 0x31 0x37 0x35 | 0x23      | 0x33 0x34 0x36 | 0x23      | 0x2D 0x31 0x33 0x35 0x32 0x38 | 0x23      | 0x2D 0x34 0x35 0x39 |
> | 9175                | #         | 346            | #         | -13528                        | #         | -459                |

## Message Type 0x05: MsgBattery
**Message Length:** 1 byte (Length - 4)

| Message Offset | Length (Bytes) | Name         | Description |
| -------------- | -------------- | ------------ | ----------- |
| 0              | 1              | BatteryLevel | <p>Current battery level percentage<br>Range: 0x00 - 0x64 (0 - 100)</p> |

> **Example:**
> | Byte Offset | Value | Description |
> | ----------- | ----- | ----------- |
> | 0           | 0x2A  | Prefix: asterisk (*) |
> | 1           | 0x05  | Length: 5 bytes |
> | 2           | 0x05  | Message Type: MsgBattery |
> | 3           | 0x38  | Message: Battery level: 56% |
> | 4           | 0x6C  | Checksum: (0x2A + 0x05 + 0x05 + 0x38) % 0x100 |
> | 5           | 0x0D  | Suffix: CR |
> | 6           | 0x0A  | Suffix: LF |

## Message Type 0x07: MsgOffLineStats
**Message Length:** variable (Length - 4)

Format: moves#time#solves

## Message Type 0x08: MsgCubeType
**Message Length:** 1 byte (Length - 4)

| Message Offset | Length (Bytes) | Name         | Description |
| -------------- | -------------- | ------------ | ----------- |
| 0              | 1              | CubeType     | <p>0x00: Non edge cube<br>0x01: Edge cube</p> |

> **Example:**
> | Byte Offset | Value | Description |
> | ----------- | ----- | ----------- |
> | 0           | 0x2A  | Prefix: asterisk (*) |
> | 1           | 0x05  | Length: 5 bytes |
> | 2           | 0x08  | Message Type: MsgCubeType |
> | 3           | 0x01  | Message: Edge cube |
> | 4           | 0x38  | Checksum: (0x2A + 0x05 + 0x08 + 0x01) % 0x100 |
> | 5           | 0x0D  | Suffix: CR |
> | 6           | 0x0A  | Suffix: LF |

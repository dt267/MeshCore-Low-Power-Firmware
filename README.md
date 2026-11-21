# MeshCore Low Power Firmware
MeshCore firmware built for multi-day off-grid operation. Deep power optimization, a full on-device UI that connects over BLE, WiFi or USB — or runs standalone with no phone — plus advanced radio and network controls.

**Supported devices:** 
- Heltec WiFi LoRa 32 V3, WSL3
- Heltec WiFi LoRa 32 V4.2, V4.3 (with or without OLED display)
- Heltec Vision Master E213 (2.13" e-ink) — V1.0\*, V1.1
- Heltec Vision Master E290 (2.9" e-ink)\*
- Heltec Wireless Paper (2.13" e-ink) — V1.1, V1.1.1, V1.2\*
- Heltec Mesh Node T096
- Seeed Studio XIAO ESP32S3 & Wio-SX1262 Kit (with or without OLED display)
- RAK4631 WisBlock
- Waveshare ESP32-S3-LR1121-XF (dual-band sub-GHz + 2.4 GHz, with or without OLED display)
- EBYTE EoRa-HUB-xxxTB (dual-band sub-GHz + 2.4 GHz)
- Heltec HT-CT62 (ESP32-C3 + SX1262 module), and DIY ESP32-C3 + SX1262 boards wired the same way (with or without OLED display)
- ProMicro nRF52840 on a fakeTec board with a Heltec HT-RA62 or Ai-Thinker Ra-01SH radio module, and DIY ProMicro + SX1262 boards wired the same way (with or without OLED display)
- Seeed Wio Tracker L1 and L1 Pro (1.3" OLED), L1 Lite (no display)\*
- RAK3401 WisBlock with the RAK13302 1 W LoRa module, ported from the RAK4631\*
- ...

\* The developer has no unit of these devices: they are not tested or power-profiled by the developer.


[![GitHub Release](https://img.shields.io/github/v/release/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4)](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases/latest) [![GitHub Release Date](https://img.shields.io/github/release-date/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4)](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases/latest)

## Table of Contents
- [What's New](#whats-new)
- [Changelog](CHANGELOG.md)
- [Installation](#installation)
- [Idle Battery Life Estimation](#idle-battery-life-estimation-2000-mah-battery)
- [RX Duty Cycle](#rx-duty-cycle)
- [Hardware Wiring Guide](Hardware_Wiring_Guide.md) — I2C sensors, OLED, GPS, battery divider, DIY boards
- [Detailed Power Profiles by Device](Power-Profiles-Detailed.md)
- [Bypass External LNA on Heltec V4.2](Bypass-External-LNA-on-Heltec-V4.md)
- [Stock Antennas SWR Testing](Stock_Antennas_SWR_Testing.md)
- [Companion Display UI Guide](Companion_Display_Guide.md)
- [Companion TerminalCLI Commands](Companion_TerminalCLI_Commands.md)
- [Repeater / Room Server CLI Commands](Repeater_CLI_Commands.md)
- [License](#license)

## What's New

### v1.18_1121

- **New device: RAK3401 with the RAK13302 1 W LoRa module.** *(Companion, Repeater, Room Server)*

  `RAK_3401_*.uf2` images for the RAK3401 WisBlock Core with the RAK13302 in the IO slot; a RAK4631 with a RAK13302 runs them too. Flash like the other nRF52 boards (see [Installation](#installation)).

  **Not yet tested on hardware:** the port is built from the RAK4631 support.

- **Changed: RAK4631 user button on AIN1 only.** *(Companion)*

  The user button is AIN1 (P0.31) to GND, header J11 pin 1 on the RAK19007. A button on IO5 (P0.09) no longer works. See [wiring](Hardware_Wiring_Guide.md#optional-oled-display).

- **Changed: three lines of a message on OLED with the large font.** *(Companion)*

  The OLED `Large` message font goes from 13 px to 12 px, so a message shows three lines instead of two.

- **New: Brightness on OLED.** *(Companion)*

  Settings → Display → **Brightness** now works on OLED screens too: `100%`, `75%`, `50%`, `25%`.

- **New: Lock keys.** *(Companion)*

  With the joystick on or a CardKB attached, **Lock keys** at the top of the Home menu locks the keys, and a lock icon shows in the header. Any key then shows `Press Up to unlock`. See [Display Guide](Companion_Display_Guide.md#home-page).

- **New: Mute a contact, room server or channel.** *(Companion)*

  **Mute** in the contact or channel menu, or in the popup while reading a message, silences that conversation: its new messages make no sound or vibration, show no alert and do not turn the screen on. They are still kept and show as unread. **Unmute** turns it back on. See [Display Guide](Companion_Display_Guide.md#new-message-alert).

- **New: display on Vision Master E213 V1.0 and Wireless Paper V1.1.** *(Companion, Repeater, Room Server)*

  These units have an older e-ink panel, and the firmware ran on them without the display. It now detects the panel at boot and drives it.

### v1.18_1114

> **Important fix, update recommended for ESP32 boards on v1.18_1018, v1.18_1025, v1.18_1031 or v1.18_1107:** *(Companion, Repeater, Room Server)* after OTA or a connection mode change, the node could keep drawing high current until the RESET button was pressed. The WiFi / Bluetooth radio is now shut down properly when the node restarts. In those versions, a restart while WiFi or Bluetooth was on, such as the end of OTA, `reboot` during OTA, or a companion's switch from WiFi or BLE to USB, left the radio powered: a Heltec V4 repeater drew about 32 mA instead of 4.4 mA until the RESET button was pressed or the power removed. Updating over OTA from these versions clears it, no RESET needed.

- **New device: Seeed Wio Tracker L1, L1 Pro and L1 Lite.** *(Companion, Repeater, Room Server)*

  One set of `WioTrackerL1_*.uf2` images serves all three: the L1 and L1 Pro with their 1.3" OLED, and the L1 Lite, which has no display and runs without one. The joystick is built in and always on, with the Menu button as Back, and the Grove port takes I2C sensors or a CardKB. Flash like the other nRF52 boards (see [Installation](#installation)).

  **Not yet tested on hardware:** the port is built from the other devices this firmware supports.

- **New: 5-way joystick on Heltec V4, E213, E290 and Waveshare LR1121.** *(Companion)*

  The joystick and on-screen keyboard now work on these boards too. On the E213 and E290, the second button is Back while the joystick is on. See [wiring](Hardware_Wiring_Guide.md#5-way-joystick).

- **New: send emoji from the node.** *(Companion)*

  Turn on **Emoji** in Settings → Input Languages. In the message editor, Tab with the CardKB, or the `EMO` key on the on-screen keyboard, shows a grid of emoji: the top row types at once, the other keys open a row of similar ones. See [Display Guide](Companion_Display_Guide.md#on-screen-keyboard).

- **New: favourites on the Location Radar.** *(Companion)*

  **Source** → **Faves** shows only your favourite contacts, to keep an eye on a group. The top left shows how long ago the selected contact's position was received. See [Display Guide](Companion_Display_Guide.md#location-radar-screen).

- **New: direct messages from the node are retried.** *(Companion)*

  A direct message sent from the display is tried up to 5 times, the last time by flood, as the app does. The screen shows each try and whether it goes direct or by flood, e.g. `Retry 2/5 direct...`.

- **New: OTA over your own WiFi network.** *(Companion, Repeater, Room Server — ESP32 boards)*

  `start ota sta`, or **Start OTA STA** in the companion's Settings, runs the update on the WiFi network set with `wifi.ssid` / `wifi.pwd` instead of the `MeshCore-OTA` hotspot. The repeater's reply, or the companion's screen, gives the address to open. Both OTA modes also answer at a `.local` name: `meshcore.local` on the hotspot, `meshcore-xxxx.local` on your own network. On the companion, any button press leaves OTA. `wifi scan` lists the networks a repeater or room server hears. See [Repeater CLI](Repeater_CLI_Commands.md) and [Display Guide](Companion_Display_Guide.md).

- **Improved: lower idle current with BLE off.** *(Companion — ESP32 boards)*

  Turning **BLE** off in Settings now shuts Bluetooth down completely, so a node used on its own without the app lasts longer on battery. Idle current with BLE off, SX1262 Boosted Gain off and RX Duty Cycle on, previous release → this release: Heltec V4.3 8.06 mA → 4.48 mA, Heltec V3 4.06 mA → 3.83 mA.

- **New: page icons in the header.** *(Companion)*

  Each page shows its icon at the top left instead of its name, leaving more room in the header for status icons. A new icon shows when Bluetooth is off. See [Display Guide](Companion_Display_Guide.md#home-screen-pages).

- **New: set the time zone, date and time on the node.** *(Companion)*

  Settings → General has **Timezone**, **Date** and **Time**, so the clock can be set without the app or GPS. Pick the time zone first, then enter the local date and time. See [Display Guide](Companion_Display_Guide.md#date-and-time-screen).

### v1.18_1107

- **New: 5-way joystick and on-screen keyboard on ProMicro, Heltec V3, WSL3, T096 and EoRa-HUB.** *(Companion)*

  A joystick moves through the screens directly, and without a CardKB an on-screen keyboard types messages in every input language, with accented letters right on the screen. Turn it on with **Joystick** in Settings → General. See [Display Guide](Companion_Display_Guide.md#5-way-joystick) and [wiring](Hardware_Wiring_Guide.md#5-way-joystick).

- **New: path hash size and default region on the node.** *(Companion)*

  Settings → Radio has **PathHash** (`1 byte` → `2 bytes` → `3 bytes`) and **Region**; long press on the Radio page now opens the same list as Settings → Radio, Repeat included. Region opens a list: **None**, **Discover**, which asks the repeaters in direct range for their regions as the app's "Fetch regions from repeaters" does, and with the CardKB **Type name...**. See [Display Guide](Companion_Display_Guide.md#region-screen).

- **New: Location Radar.** *(Companion)*

  **Location Radar** at the top of the saved locations list shows your saved locations and contacts around you, so you can see where a contact is next to places you know. Long press picks the zoom, the source or **Trace**. Saved locations are now numbered from 0, as in `set loc.<N>`. See [Display Guide](Companion_Display_Guide.md#location-radar-screen).

- **Fix: compass mounted upside down.** *(Companion)*

  A compass module mounted upside down now gives the right heading: calibration detects it, so turn the device clockwise while calibrating, and calibrate once more after updating. Before, east and west were swapped and the GPS Trace arrow pointed the wrong way.

- **New: one `help` for every node.** *(Companion, Repeater, Room Server)*

  Companion, repeater and room server now share the same `help`, also over the companion's Remote CLI (**TerminalCLI** in a person's menu). `help` lists the sections, `help <section>` shows one, e.g. `help gps`, and `help <word>` finds commands by word, e.g. `help wifi`; this replaces `help all`. See [Companion TerminalCLI Commands](Companion_TerminalCLI_Commands.md).

- **New: Reply names the author.** *(Companion)*

  **Reply** on someone else's message in a channel or room now mentions its author, as the apps do, and messages show mentions as `@name`. With the CardKB the mention is already typed in and you can edit it; with presets the title shows `@name`.

- **New: emoji in messages and names.** *(Companion)*

  Emoji from the app now show on TFT and e-ink displays and with the OLED large font. With the OLED normal font the common ones show as text, e.g. 😜 as `;P`.

- **Fix: overlapping text in messages with a line break.** *(Companion)*

  A line break in a message now starts a new line on TFT and e-ink displays and with the OLED large font. Before, the text after it was drawn over by the next line on TFT and e-ink, and ran into the previous sentence on OLED.

See [CHANGELOG.md](CHANGELOG.md) for all earlier releases.

## Installation

### ESP32-S3 / ESP32-C3

> **Unified binary:** A single firmware file runs on both OLED and no-display hardware variants — no separate build required. The display is detected automatically at boot via I2C probe. The device name shown in the MeshCore app reflects the actual hardware detected (e.g. *Heltec V4.3 No Display* vs *Heltec V4.3 OLED*).

Two binary formats are provided for each firmware variant:

| File | Contents | Use case |
| :--- | :--- | :--- |
| `<name>.bin` | Application only | Update / OTA — preserves existing partitions |
| `<name>_merged.bin` | Bootloader + partition table + application | First-time install or full recovery |

> **Custom partition table on 4 MB boards — flash the merged image first.** Waveshare ESP32-S3-LR1121-XF, EBYTE EoRa-HUB and Heltec HT-CT62 use this firmware's own partition table:
>
> | Partition | Offset | Size |
> | :--- | :--- | :--- |
> | nvs | `0x9000` | 20 KB |
> | otadata | `0xE000` | 8 KB |
> | app0 (OTA slot 0) | `0x10000` | 1.6875 MB |
> | app1 (OTA slot 1) | `0x1C0000` | 1.6875 MB |
> | spiffs (settings, contacts) | `0x370000` | 512 KB |
> | coredump | `0x3F0000` | 64 KB |
>
> It differs from other firmware for these boards, so the first time you install this firmware — on a new board, or one coming from any other firmware — flash the `_merged.bin` image (Option 1). It writes the bootloader, this partition table and the application in one go. Option 2 and Wi-Fi OTA only work once this partition table is on the board.

**Option 1: Full flash (merged binary) — recommended for first-time install**

Flash the `_merged.bin` file starting at address `0x0`. This is self-contained and requires no prior MeshCore installation.
```
python -m esptool --chip esp32s3 write_flash 0x0 <name>_merged.bin
```
For Heltec HT-CT62 (ESP32-C3) use `--chip esp32c3` here and in Option 2. If esptool cannot connect, hold **BOOT** (GPIO9) while pressing **RESET** to enter download mode.

> **Note:** Full flash erases the NVS partition, which stores BLE pairing keys — you will need to re-pair BLE devices after flashing. Settings stored in the SPIFFS filesystem partition are beyond the merged binary range and are **not** affected.

**Option 2: Application update via esptool**

Flash the plain `<name>.bin` file at `0x10000`. The existing bootloader and partition table are preserved.
```
python -m esptool --chip esp32s3 write_flash 0x10000 <name>.bin
```

**Option 3: Wi-Fi OTA** *(requires v1.14_0320 or later)*
Type `start ota` via TerminalCLI (Companion) or Command Line (Repeater / Room Server) → connect to `MeshCore-OTA` Wi-Fi → go to `192.168.4.1/update`. Upload the plain `<name>.bin` file only — the merged binary is **not** compatible with OTA.

### nRF52840

**Option 1: UF2 drag-and-drop**
Double-tap the Reset button → a USB drive appears → copy the `.uf2` file onto it. The device reboots automatically when done.
- **RAK4631:** drive is named `RAK4631`
- **Heltec T096:** drive is typically named `HT-n5262G`
- **ProMicro:** drive is typically named `NICENANO`. If the first line of `INFO_UF2.TXT` on the drive reads `UF2 Bootloader 0.6.0`, the Companion `.uf2` does not install: after the copy the LED keeps blinking fast and the board never reboots (Repeater and Room Server install fine). Update the bootloader first: copy `update-nice_nano_bootloader-0.11.0_nosd.uf2` from the [Adafruit nRF52 Bootloader releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases/tag/0.11.0) onto the drive, wait until the drive comes back with `0.11.0` in `INFO_UF2.TXT`, then copy the firmware again. The bootloader update erases the installed firmware.

**Option 2: BLE DFU** *(requires v1.14_0320 or later)*
Enter DFU mode, then use the **nRF Device Firmware Update** app (iOS/Android) to upload the `.zip` DFU package.

Ways to enter DFU mode:
- **RAK4631, RAK3401:** Type `start ota` via TerminalCLI (Companion) or Command Line (Repeater / Room Server)
- **Heltec T096:** Hold the user button while pressing Reset — or type `start ota` via TerminalCLI / Command Line — or on Companion: **Settings → Start OTA**


## Idle Battery Life Estimation (2000 mAh battery)

| Device | Idle Current (mA) | Estimated Idle Runtime (Hours) | Estimated Idle Runtime (Days) | Firmware |
| :--- | :--- | :---: | :---: | :---: |
| Heltec T096 Companion BLE, KCT8103L LNA off, SX1262 Boosted Gain off, RX Duty Cycle on | 3.68 | 462.0 | 19.25 | v1.18_1018 |
| Heltec T096 Repeater, KCT8103L LNA off, SX1262 Boosted Gain off, RX Duty Cycle on | 3.64 | 467.0 | 19.46 | v1.18_1018 |
| Heltec V4.3 Companion BLE, KCT8103L LNA off, SX1262 Boosted Gain off, RX Duty Cycle on | 11 | 154.5 | 6.4 | v1.18_1018 |
| Heltec V4.3 Companion BLE, KCT8103L LNA on, SX1262 Boosted Gain off, RX Duty Cycle on | 19.4 | 87.6 | 3.7 | v1.18_1018 |
| Heltec V4.3 Companion, BLE off, KCT8103L LNA off, SX1262 Boosted Gain off, RX Duty Cycle on | 4.48 | 379.5 | 15.81 | v1.18_1114 |
| Heltec V4.3 Repeater, KCT8103L LNA off, SX1262 Boosted Gain off, RX Duty Cycle on | 4.52 | 376.1 | 15.7 | v1.18_1018 |
| Heltec V4.3 Repeater, KCT8103L LNA on, SX1262 Boosted Gain off, RX Duty Cycle on | 12.78 | 133.0 | 5.5 | v1.18_1018 |
| Heltec V3 Companion BLE, SX1262 Boosted Gain off, RX Duty Cycle on | 6.15 | 276.4 | 11.52 | v1.18_1018 |
| Heltec V3 Companion, BLE off, SX1262 Boosted Gain off, RX Duty Cycle on | 3.83 | 443.9 | 18.49 | v1.18_1114 |
| Heltec V3 Repeater, SX1262 Boosted Gain off, RX Duty Cycle on | 3.82 | 445.0 | 18.54 | v1.18_1018 |
| RAK4631 Companion BLE, SX1262 Boosted Gain off, RX Duty Cycle on | 2.25 | 755.6 | 31.48 | v1.18_1018 |
| RAK4631 Repeater, SX1262 Boosted Gain off, RX Duty Cycle on | 2.21 | 769.2 | 32.05 | v1.18_1018 |
| ProMicro fakeTec Companion BLE, SX1262 Boosted Gain off, RX Duty Cycle on | 2.7 | 629.6 | 26.23 | v1.18_1018 |
| ProMicro fakeTec Repeater, SX1262 Boosted Gain off, RX Duty Cycle on | 2.64 | 643.9 | 26.83 | v1.18_1018 |
| DIY ESP32-C3 Super Mini + SX1262 Companion BLE, 32.768 kHz crystal, SX1262 Boosted Gain off, RX Duty Cycle on | 4.7 | 361.7 | 15.07 | v1.18_1018 |
| DIY ESP32-C3 Super Mini + SX1262 Repeater, 32.768 kHz crystal, SX1262 Boosted Gain off, RX Duty Cycle on | 3.33 | 510.5 | 21.27 | v1.18_1018 |

## RX Duty Cycle

RX duty cycle (`set rx.duty on`, or **RxDuty** on the Companion display) lets the receiver sleep between short listening windows. It runs only with some SF and bandwidth combinations. The table shows where it runs and the share of the time the receiver still listens, the figure `get rx.duty` shows (lower saves more):

| SF | BW 62.5 kHz | BW 125 kHz | BW 250 kHz | BW 500 kHz |
|---|---|---|---|---|
| SF5 | – | – | – | – |
| SF6 | 55% | – | – | – |
| SF7 | 47% | 55% | – | – |
| SF8 | 41% | 47% | 55% | – |
| SF9 | 72% | 77% | – | – |
| SF10 | 72% | 72% | 77% | – |
| SF11 | 72% | 72% | 72% | 77% |
| SF12 | 72% | 72% | 72% | 72% |

– = does not run, the receiver listens all the time.

---

## License

The files in this repository are licensed under the [MIT License](LICENSE).

# MeshCore - Low power firmware for Heltec Lora 32 V3, V4 & WSL3, Seeed Studio Xiao S3 Wio, RAK4631
Optimized MeshCore firmware, engineered for low power consumption and extended off-grid battery life for multi-day operation.


[![GitHub Release](https://img.shields.io/github/v/release/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4)](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases) [![GitHub Release Date](https://img.shields.io/github/release-date/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4)](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases)

## Table of Contents
- [What's New](#whats-new)
- [Installation](#installation)
- [Idle Battery Life Estimation](#idle-battery-life-estimation-2000-mah-battery)
- [Power Profiles: Heltec Lora 32 V3](#heltec-lora-32-v3)
- [Power Profiles: Heltec Lora 32 V4.2](#heltec-lora-32-v42)
- [Power Profiles: Seeed Studio XIAO ESP32S3 & Wio-SX1262](#seeed-studio-xiao-esp32s3--wio-sx1262)
- [Power Profiles: RAK4631 (RAK19003)](#rak4631-rak19003)
- [Bypass External LNA on Heltec V4.2](#bypass-external-lna-on-heltec-v42)
- [Companion Display UI Guide](Companion_Display_Guide.md)
- [Companion TerminalCLI Commands](Companion_TerminalCLI_Commands.md)
- [License](#license)

## What's New

### v1.14_0417

- **Hybrid RSSI + hardware CAD channel sensing (all node types).**

  `isChannelActive()` now performs a two-stage check before transmitting:
  1. RSSI check (fast, single SPI register read) — defers if signal is above `noise_floor + int.thresh`.
  2. Hardware CAD (`scanChannel()`) — if RSSI misses, performs LoRa chirp correlation to detect signals below the noise floor (~16ms blocking scan on SX126x).

  RSSI also detects any in-band signal (interference, jamming), while CAD only correlates LoRa chirp patterns and ignores non-LoRa noise entirely. With hybrid, RSSI acts as the first guard — CAD only runs when the channel appears clear to RSSI.

  `int.thresh=0` disables both RSSI and CAD. `int.thresh=1` enables full hybrid at maximum sensitivity.

  **On repeaters** (single source sending): results depend on topology. Field tests with 4 repeaters (SF8/BW62.5kHz, **`txdelay=2`**, 100 messages):

  - **Repeaters close together / strong inter-repeater signal:** RSSI handles detection well, CAD rarely fires. Example: int.thresh=1 → **9% collision rate**.
  - **Spread-out repeaters, some pairs below noise floor:** RSSI misses sub-NF pairs; CAD fills the gap. Example: hybrid/CAD **8%** vs RSSI-only **17%**.
  - **Many hidden node pairs:** neither RSSI nor CAD helps. **Only `txdelay` reduces the floor.** Example: ~20–24% regardless of sensing method.

  **On companions** (multiple sources sending concurrently): channel sensing still helps, but with diminishing returns. Tested with 2 concurrent companions plus a third node sending long messages every 5s (SF8/BW62.5kHz):

  - `int.thresh=3`: **53–64%** of messages successfully relayed by all 4 repeaters (confirmed by hearing each relay back)
  - `int.thresh=0` (no sensing): **0–1%** relayed by all 4; most messages are relayed by 0–1 repeaters only — collisions occur at two levels: concurrent companion transmissions corrupt each other at the repeater, and the resulting relay transmissions from multiple repeaters collide on the way back

  Channel sensing — even imperfect — is far better than none. The remaining loss at int.thresh=3 is a fundamental ALOHA-style limitation: uncoordinated LoRa nodes cannot eliminate simultaneous transmission without a shared scheduling mechanism that does not exist in this protocol.

- **Companion: `get/set txdelay`, `get/set direct.txdelay`, `get/set int.thresh` via [TerminalCLI](Companion_TerminalCLI_Commands.md).**

  Relay timing and interference threshold are now configurable without reflashing.

- **Companion: "Heard N Repeats" alert after Quick Send.**

  After sending from the Quick Send screen, the display shows how many repeaters have relayed the message (e.g. "Heard 3 Repeats"). The counter updates in real time as each relay is heard.

- **Companion: local time and date on the display.**

  All pages now show the current time (`HH:MM`) in the header, between the page title and the battery icon. The Home page also shows the full date at the bottom (e.g. `14 Apr 2026`).

  Time is sourced from the device RTC, which is synchronized upon app connection or GPS fix. Configure your local timezone offset once via [TerminalCLI](Companion_TerminalCLI_Commands.md):
  ```
  set tz.offset 7    # UTC+7
  set tz.offset -5   # UTC-5
  get tz.offset
  ```
  Offset is saved to flash. All internal timestamps remain UTC — the offset is applied only for display.

- **Companion: Metric / Imperial units.**

  A new **Units** item in the Settings page toggles between Metric and Imperial. Setting is saved to flash and persists after reboot.

  | Display | Metric | Imperial |
  |---|---|---|
  | GPS Trace distance | `150m` / `1.2km` | `492ft` / `0.7mi` |
  | GPS page altitude | `245m` | `804ft` |
  | Home page date | `14 Apr 2026` | `Apr 14 2026` |

- **Companion: GPS Privacy mode.**

  A new **GPS Privacy** item in the Settings page lets you stop GPS coordinates from being attached to Quick Send messages. When enabled, the Quick Send bottom line shows `GPS: Private` as a reminder. Toggle it off to resume sharing coordinates. Setting is saved to flash.

- **Companion: ESP32 BLE now connects reliably on Windows 11.**
- **Companion: ESP32 & nRF52 BLE random disconnect issue is fixed.**
- **Included 'default-scope'**

### v1.14_0410

- **Message preview: scroll long messages & see all 256 buffered messages.**

  The message preview screen is rebuilt from the ground up. All 256 buffered messages are now navigable — previously capped at 32. Long messages that overflow the screen can be scrolled line by line.

  #### Button controls in message preview

  | Action | Effect |
  |---|---|
  | **Single click** | Scroll text down (3 lines); advances to next older message at end of text |
  | **Double click** | Scroll text up (3 lines); goes to next newer message at top; at newest → home |
  | **Long press** | Open menu: **Save location** *(if message has GPS coords)* / **Home** |

  #### Counter and unread tracking

  ```
  ┌──────────────────────────────┐
  │ 5/19                    42s  │
  │──────────────────────────────│
  │ (2) Alien:                   │
  │ Hello everyone, just wanted  │
  │ to check in. We made it to   │
  │ base camp safe and sound.    │
  │                           ▼  │
  └──────────────────────────────┘
  ```

  `5/19` = viewing message #5 (newest = 19, oldest = 1). `▼` = more text below. `42s` = time since received. The counter tracks unread messages — when you close preview and return, only new messages since last session are counted.

- **Saved Locations: save GPS coordinates from messages to flash.**

  When viewing a message with GPS coordinates, long press opens a menu. Choose **Save location**, then pick one of 10 slots to save into. Saved locations persist in flash memory — they survive reboot.

  Navigate to the **SAVED LOCS** page on the home screen to browse your saved locations and open the **GPS Trace screen** for any of them.

  ```
  ┌──────────────────────────────┐
  │ SAVED  2/10                  │
  │──────────────────────────────│
  │ > Alien: I need help         │
  │   Big Boy: Heading home      │
  │                              │
  │                              │
  └──────────────────────────────┘
  ```

  Each entry shows **sender + message snippet** so you can identify entries even when multiple locations from the same person are saved.

  | Press | Effect |
  |---|---|
  | Single click | Move highlight to next entry |
  | Long press | Open **GPS Trace screen** for that location |
  | Double click | Return to home |

- **GPS Trace screen: live distance & bearing to a saved location.**

  ```
  ┌──────────────────────────────┐
  │ Alien: I need help       5m  │
  │──────────────────────────────│
  │      10.7769  106.7009       │
  │                              │
  │            1.2km             │
  │                              │
  │          247°  WSW           │
  └──────────────────────────────┘
  ```

  The timer in the top-right corner shows how long you have been on this Trace screen. Requires own GPS fix for distance/bearing. Raw coordinates are always shown. Any button returns to the Saved Locations list.

- **Saved locations CLI commands ([TerminalCLI](Companion_TerminalCLI_Commands.md)).**

  Manage saved locations from the terminal without touching the display:

  | Command | Effect |
  |---|---|
  | `get loc` | List all occupied slots (`N:lat,lon:name`, N is 0-based) |
  | `set loc.<N> <name> <lat> <lon>` | Save to slot N (0-based; display shows 1–10) |
  | `del loc.<N>` | Clear slot N (0-based) |
  | `del loc.all` | Clear all slots |

### v1.14_0404

- **Quick Send and Settings — control your Companion without a phone.**

  Two new pages are added to the Companion's display, accessible without a phone or app.

  #### Button controls

  - **Single click / Double click** — navigate between pages (next / previous)
  - **Long press on FIRST page** — reopen unread message preview (up to 32 messages buffered)
  - **Long press** on Quick Send or Settings — enter the page; active item highlights
    - Single click = next item · Double click = exit · Long press = confirm

  #### Quick Send

  Send a short status message directly over LoRa to the public channel — no typing, no phone needed. Useful when your phone is dead or unavailable.

  GPS coordinates are automatically appended if available (e.g. `I'm OK @10.7769,106.7009`). If GPS has no current fix, last known coordinates are used with a `?` prefix so you know before sending.

  **10 built-in presets:**
  - I'm OK
  - On my way
  - I need help
  - Everyone OK here
  - Wait for me
  - Heading home
  - Running late
  - Lost contact, call me
  - Battery low, signing off
  - All clear

  **Customize via [TerminalCLI](Companion_TerminalCLI_Commands.md)** — changes are saved to flash and persist after reboot:
  ```
  get quick                        list all current presets
  set quick.0 Arrived at camp      set preset at index 0
  set quick.reset                  restore all 10 built-in defaults
  ```

  #### Settings

  A scrollable list of device settings, controlled directly from the button:

  | Item | Action |
  |---|---|
  | BLE | Toggle Bluetooth on/off (shows connection state) |
  | Repeat | Toggle repeat mode on/off |
  | RxGain | Cycle RxGain mode: OFF → ON → AUTO *(AUTO: V4.2 only)* |
  | Buzzer | Toggle buzzer on/muted *(boards with buzzer only)* |
  | Send Advert | Broadcast your presence to nearby nodes |
  | Start OTA | Start OTA update mode — connect to `MeshCore-OTA` WiFi and go to `192.168.4.1/update` |
  | Shutdown | Power off the device |

- **Heltec V4.3 support (KCT8103L FEM).**
  Firmware automatically detects V4.2 / V4.3 at boot — no configuration required. V4.3 replaces the GC1109 FEM with KCT8103L, which supports explicit LNA/bypass RX mode selection via `radio.rxgain`.

  | Mode | Description |
  |---|---|
  | `on` *(default)* | FEM LNA active — best sensitivity |
  | `off` | FEM bypass — SX1262 boosted gain compensates, better in high-interference environments near strong transmitters |

  > V4.3 does **not** support `auto` mode — use `on` or `off`.

  **[TerminalCLI](Companion_TerminalCLI_Commands.md)** (Companion app), **Command Line** (Repeater / Room Server):
  ```
  set radio.rxgain on
  set radio.rxgain off
  get radio.rxgain
  ```
  
  Setting is saved and restored after reboot.

- **Unified firmware for "No Display" hardware variants.**

  A single firmware binary now runs on both OLED and no-display hardware — no separate build required. The display is detected automatically at boot via I2C probe.

  | Hardware | Detected as |
  |---|---|
  | Heltec V3 with OLED | Heltec V3 |
  | Heltec WSL3 (no OLED) | Heltec WSL3 |
  | Heltec V4.2 with OLED | Heltec V4.2 OLED |
  | Heltec V4.2 without OLED | Heltec V4.2 No Display |
  | Heltec V4.3 with OLED | Heltec V4.3 OLED |
  | Heltec V4.3 without OLED | Heltec V4.3 No Display |

  The device name shown in the MeshCore app reflects the actual hardware detected. On no-display hardware, the user button has no effect (previously it could trigger unintended I2C writes).

### v1.1414_0327
- **Bidirectional clock sync on Repeater and Room Server — no more `clkreboot` needed.**
  `clock sync` now works in both directions — no manual `clkreboot` + re-sync required.

- **Repeat mode on Companion now supports custom frequencies.**
  Useful for off-grid or emergency deployments where no public MeshCore network is available and a private frequency is used. You can add your operating frequency to the allowed list via [TerminalCLI](Companion_TerminalCLI_Commands.md) — for example `add repeat.freq 915`.

  | Command | Parameters | Notes |
  |---|---|---|
  | `get repeat.freqs` | — | List all frequencies allowed for repeat mode (MHz) |
  | `add repeat.freq <MHz>` | `MHz`: frequency in MHz, e.g. `915` | Add a frequency to the repeat allowed list (max 5). Setting is retained after reboot |
  | `del repeat.freq <MHz>` | `MHz`: frequency in MHz | Remove a frequency from the repeat allowed list |

- **Low-battery protection and battery voltage reading now available on Xiao S3 Companion** (previously Repeater/Room Server only).
  Use `get adc.multiplier` / `set adc.multiplier <value>` in [TerminalCLI](Companion_TerminalCLI_Commands.md) — same commands and circuit as documented in the [Earlier](#earlier) section below. Setting is retained after reboot and power cycle.

- **BLE random disconnects may be fixed.**
  Early testing shows 100h+ continuous connection stability in the background — feedback welcome.

### v1.14_0322
- **Node names and messages in non-English languages now display correctly on Companion's screen.**
  Characters from Bulgarian, Catalan, Croatian, Czech, Danish, Dutch, Estonian, Finnish, French, German, Hungarian, Icelandic, Italian, Latvian, Lithuanian, Macedonian, Maltese, Norwegian, Polish, Portuguese, Romanian, Russian, Serbian, Slovak, Slovenian, Spanish, Swedish, Turkish, Ukrainian, Vietnamese, and Welsh are automatically converted to their closest English equivalents — so text stays readable on standard OLED/LCD screens without any layout changes.

### v1.14_0320
- **Command Line Interface for Companion.**
  Setup: In the MeshCore app, create a channel named "TerminalCLI". It will now act as a Terminal CLI for Companion; everything typed here is a command. Supported CLI for Companion:

  | Command | Parameters | Notes |
  |---|---|---|
  | `stats` | — | Display battery (mV), uptime (s), noise floor, last RSSI/SNR, RX/TX/error packet counters |
  | `reboot` | — | Reboot the device |
  | `poweroff` | — | Power off the device |
  | `gps` | — | Show GPS status |
  | `gps on` | — | Enable GPS |
  | `gps off` | — | Disable GPS |
  | `reg read <addr>` | `addr`: hex register address | Read 1 byte from radio register. Example: `reg read 08B5` |
  | `reg write <addr> <val>` | `addr`, `val`: hex | Write 1 byte to radio register. Example: `reg write 08B5 04` |
  | `get radio.rxgain` | — | Show current RX gain mode: `off`, `on`, or `auto` |
  | `set radio.rxgain <mode>` | `off` \| `on` \| `auto` | Set RX gain mode. `auto` is for Heltec V4.2 only. |
  | `start ota` | — | Start Wi-Fi OTA firmware update: connect to `MeshCore-OTA`, go to `192.168.4.1/update` |

- **Wi-Fi OTA Update for Companion.**
  Type `start ota` in "TerminalCLI" (above) and use it just like the Wi-Fi OTA Update on the Repeater.  

  <img height="600" alt="Screenshot 2026-03-19 at 9 38 13 PM" src="https://github.com/user-attachments/assets/c1df229f-5eed-43b8-abdb-906c3c864a62" />

### v1.14_0315
- **Synchronizing GPS usage with low-power mode on Heltec V4.2.**
  GPS power is kept on only as needed for frame acquisition; update intervals scale dynamically (10-30s) based on signal quality. GPS power profile on repeater: 
   - Mean: 32.39 mA. 
   - Estimated battery life: ~52 h (2000 mAh battery)
  
  <img alt="v4-repeater-gps-power-usage" src="https://github.com/user-attachments/assets/cf934adc-ca11-41c3-99a9-7a8d7e1f0857" />

### v1.14_0307
- **Adaptive Rx Boosted Gain on Heltec V4.2.**
  A new algorithm for acquiring and calculating ambient noise floor that accurately tracks environmental fluctuations. This enables the Heltec V4.2 to autonomously toggle the Rx Boosted Gain mode based on real-time noise floor conditions.
- **'poweroff' CLI command for repeater and room server.**

### v1.13_0301
- **Read/write SX1262 register CLI for repeater and room server.**
  Usage:
  ```
  reg read <address> : read 1 byte from the register.
  reg write <address> <value> : write 1 byte to the register.
  ```
  Examples:
  ```
  reg read 08AC ; read Whitening Initial Value (0x08AC).
  reg read 0x0740 ; read Sync Word (supports '0x' prefix).
  reg write 0740 1424 ; write 0x14, 0x24 — set private LoRa sync word.
  reg write 08AC 00 ; write 0x00 to 0x08AC.
  ```
  Example Output:
  ```
  reg[0x08AC] = 0xFF (255)
  OK - wrote 0x14 to reg[0x0740]
  ```
  Note: Register values will revert to defaults after reboot. Use at your own discretion.

### Earlier
- **Low-battery protection:** automated deep sleep at 3.4V and system recovery at 3.5V, allowing stable re-activation after recharging. With a deep sleep current below 0.5mA, a remaining 200mAh battery can provide 400 hours (~16.7 days) of standby time.
- **Supported battery monitoring for Xiao S3 Wio.** Use cli command `set adc.multiplier 2.04` or `set adc.multiplier 0` to enable/disable battery voltage measurement and low-battery protection feature on repeater/room server. The factor `2.04` must be adjusted accordingly if different resistor values are utilized in the voltage divider circuit. Measure battery voltage circuit:  
  <img height="100" alt="xiao-measure-bat" src="https://github.com/user-attachments/assets/d073d1f0-44a8-41b7-8f30-631816615716" />
- Improved battery measurement and management.
- No clock drift problem on repeater and room server firmware.
- Serial port will be deactivated after 30 seconds idle.

## Installation

### Heltec V3 / V4.2 / WSL3 · XIAO S3 Wio (ESP32)

The provided firmware is an application-only binary (non-merged). It does not include the bootloader or partition table, designed for seamless integration with existing MeshCore partitions.

**Option 1: Flash via esptool**
```
python -m esptool --chip esp32s3 write-flash 0x10000 <firmware.bin>
```

**Option 2: Wi-Fi OTA** *(requires v1.14_0320 or later)*
Type `start ota` via TerminalCLI (Companion) or Command Line (Repeater / Room Server) → connect to `MeshCore-OTA` Wi-Fi → go to `192.168.4.1/update`.

### RAK4631 (nRF52840)

**Option 1: UF2 drag-and-drop**
Double-tap the Reset button → a USB drive named `RAK4631` appears → copy the `.uf2` file onto it. The device reboots automatically when done.

**Option 2: BLE DFU** *(requires v1.14_0320 or later)*
Type `start ota` via TerminalCLI (Companion) or Command Line (Repeater / Room Server) → the device enters DFU mode → use the **nRF Device Firmware Update** app (iOS/Android) to upload the `.zip` DFU package.


## Idle Battery Life Estimation (2000 mAh battery)

| Device | Idle Current (mA) | Estimated Idle Runtime (Hours) | Estimated Idle Runtime (Days) |
| :--- | :--- | :---: | :---: |
| Heltec V3 Companion BLE | 10 | 170.0 | 7.08 |
| Heltec V3 Repeater | 7.8 | 217.9 | 9.08 |
| Heltec V3 Room Server | 8.0 | 212.5 | 8.85 |
| Heltec WSL3 Companion BLE | 10 | 170.0 | 7.08 |
| Heltec WSL3 Repeater | 7.7 | 220.8 | 9.20 |
| Heltec WSL3 Room Server | 7.9 | 215.2 | 8.97 | 
| Heltec V4.2 Companion BLE | 20 | 85.0 | 3.54 |
| Heltec V4.2 Repeater | 13.3 | 127.8 | 5.33 | 
| Heltec V4.2 Room Server | 13.4 | 126.9 | 5.29 |
| XIAO S3 Wio Companion BLE | 11 | 154.5 | 6.44 |
| XIAO S3 Wio Repeater | 8.7 | 195.4 | 8.14 |
| XIAO S3 Wio Room Server | 8.7 | 195.4 | 8.14 |
| RAK4631 Companion BLE | 6.61 | 257.18 | 10.71 |
| RAK4631 Repeater | 5.79 | 293.6 | 12.23 |

---

## Heltec Lora 32 V3 

<img alt="helec-v3" src="https://github.com/user-attachments/assets/2a4b7faf-0420-4d31-bcbe-47b970810f22" />


### *Typical power profile of Heltec V3 BLE companion 1.13.dev, 5 LoRa messages in 30 seconds (14,400 messages a day):*
* Maximum: 241.48 mA
* Minimum: 4.99 mA
* Mean: 31.37 mA

**Estimated ~54.19 h (2.57 days) with 2000mAh battery.**

<img alt="dt267-v3-113-companion-h" src="https://github.com/user-attachments/assets/3e79cd31-87a9-4366-9576-2013b8ae7469" />


### *Typical power profile of Heltec V3 BLE companion 1.13.dev in idle:*
* Maximum: 102.97 mA
* Minimum: 5.72 mA
* Mean: 9.22 mA

**Estimated ~184.38 h (7.68 days) with 2000mAh battery.**

<img alt="dt267-v3-113-companion-l" src="https://github.com/user-attachments/assets/d0c5bb68-4136-481c-80b7-b1adfe515687" />

### *Typical power profile of Heltec V3 repeater 1.13.dev in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*
* Maximum: 160.49 mA
* Minimum: 2.01 mA
* Mean: 27.74 mA
  
**Estimated ~61.28 h (2.55 days) with 2000mAh battery.**

<img alt="dt267-v3-113-repeater-h" src="https://github.com/user-attachments/assets/54f806e7-b197-47fa-a054-19e9d72cad98" />

### *Typical power profile of Heltec V3 repeater 1.13.dev in idle:*
* Maximum: 44.28 mA
* Minimum: 5.51 mA
* Mean: 7.25 mA
  
**Estimated ~234.48 h (9.77 days) with 2000mAh battery.**

<img alt="dt267-v3-113-repeater-l" src="https://github.com/user-attachments/assets/9e8511de-53c9-40e2-89db-eb517d3dcf46" />

---

## Heltec Lora 32 V4.2

<img alt="v4001" src="https://github.com/user-attachments/assets/658c4b3a-edca-451f-b4d9-d4a75e927d7c" />

### *Typical power profile of Heltec V4.2 BLE companion 1.13.dev, 5 LoRa messages in 30 seconds (14,400 messages a day):*
* Maximum: 734.11 mA
* Minimum: 14.31 mA
* Mean: 96.64 mA

**Estimated ~17.59 h (0.73 days) with 2000mAh battery.**

<img alt="dt267-v4.2-companion-h" src="https://github.com/user-attachments/assets/80f1746f-9cde-4b69-a8db-3dd9056dc87c" />

### *Typical power profile of Heltec V4.2 BLE companion 1.13.dev in idle:*
* Maximum: 107.92 mA
* Minimum: 14.93 mA
* Mean: 19.85 mA

**Estimated ~85.64 h (3.56 days) with 2000mAh battery.**

<img alt="dt267-v4.2-companion-l" src="https://github.com/user-attachments/assets/c042c2d9-2940-433f-a51c-0f901fa9ecaa" />

### *Typical power profile of Heltec V4.2 repeater 1.13.dev in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*
* Maximum: 681.08 mA
* Minimum: 11.07 mA
* Mean: 108.18 mA
  
**Estimated ~15.7 h (0.65 days) with 2000mAh battery.**

<img alt="dt267-v4.2-repeater-h" src="https://github.com/user-attachments/assets/73b0e1a5-e5fe-451c-ad1f-86396202e040" />

### *Typical power profile of Heltec V4.2 repeater 1.13.dev in idle:*
* Maximum: 53.55 mA
* Minimum: 11.74 mA
* Mean: 13.57 mA
  
**Estimated ~125.27 h (5.2 days) with 2000mAh battery.**

<img alt="dt267-v4.2-repeater-l" src="https://github.com/user-attachments/assets/3fe07ea7-d733-4df6-aaa0-7132c9307936" />

---

## Seeed Studio XIAO ESP32S3 & Wio-SX1262

![wio-sx1262-with-xiao-esp32s3](https://github.com/user-attachments/assets/615d1c65-fdb9-4769-acf4-5e9680d1a009)

### *Typical power profile of XIAO S3 Wio companion BLE 1.12.dev, 5 LoRa messages in 30 seconds (14,400 messages a day):*
* Maximum: 274 mA
* Minimum: 4.07 mA
* Mean: 35.53 mA

**Estimated ~47.9 h (1.99 days) with 2000mAh battery.**

<img alt="xiao-c-h" src="https://github.com/user-attachments/assets/da6005c2-1afa-4418-83d2-da65b6d7b371" />

### *Typical power profile of XIAO S3 Wio companion BLE 1.12.dev in idle:*
* Maximum: 127.38 mA
* Minimum: 9.47 mA
* Mean: 15.34 mA

**Estimated ~110.8 h (4.62 days) with 2000mAh battery.**

<img alt="xiao-c-l" src="https://github.com/user-attachments/assets/291cf991-2cee-47c1-96c1-c55ec0368545" />

### *Typical power profile of XIAO S3 Wio repeater 1.12.dev in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*
* Maximum: 141.1 mA
* Minimum: 0.51 mA
* Mean: 24.57 mA
  
**Estimated ~69.2 h (2.88 days) with 2000mAh battery.**

<img alt="xiao-r-h" src="https://github.com/user-attachments/assets/c859735b-a7ea-4d85-8d42-df7ef25249d9" />

### *Typical power profile of XIAO S3 Wio repeater 1.12.dev in idle:*
* Maximum: 25.59 mA
* Minimum: 4.53 mA
* Mean: 7.07 mA
  
**Estimated ~240.5 h (10.02 days) with 2000mAh battery.**

<img alt="xiao-r-l" src="https://github.com/user-attachments/assets/280c8b70-604b-4ea3-b1c4-76f21efc2c24" />

---

## RAK4631 (RAK19003)

![rak19003](https://github.com/user-attachments/assets/e95d138e-c4c4-4727-bfbb-860b077af8d3)

### *Typical power profile of RAK4631 companion BLE 1.14.dev, 5 LoRa messages in 30 seconds (14,400 messages a day):*
* Maximum: 104.16 mA
* Minimum: 6.7 mA
* Mean: 19.2 mA

**Estimated ~88.5 h (3.69 days) with 2000mAh battery.**

<img alt="rak4631-companion-h" src="https://github.com/user-attachments/assets/0c03c8ae-8104-49b5-9984-0b08b09643c2" />

### *Typical power profile of RAK4631 companion BLE 1.14.dev in idle:*
* Maximum: 10.92 mA
* Minimum: 5.25 mA
* Mean: 6.61 mA

**Estimated ~257.18 h (10.71 days) with 2000mAh battery.**

<img alt="rak4631-companion-l" src="https://github.com/user-attachments/assets/c18b00fd-a722-4e01-a852-c6bcda8d6ff6" />

### *Typical power profile of RAK4631 repeater 1.14.dev in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*
* Maximum: 100.3 mA
* Minimum: 0.2 mA
* Mean: 20.23 mA
  
**Estimated ~84.03 h (3.5 days) with 2000mAh battery.**

<img alt="ra4631-repeater-h" src="https://github.com/user-attachments/assets/2934f559-93c2-439c-aca7-3b6705fc7576" />

### *Typical power profile of RAK4631 repeater 1.14.dev in idle:*
* Maximum: 7.88 mA
* Minimum: 4.53 mA
* Mean: 5.79 mA
  
**Estimated ~293.6 h (12.23 days) with 2000mAh battery.**

<img alt="ra4631-repeater-l" src="https://github.com/user-attachments/assets/37951209-5fa6-4a37-9ff3-d91ee58fd820" />

---

## Note:
- Heltec V3's LoRa Tx is 22dBm into dummy load.
- Heltec V4.2's LoRa Tx is 28dBm into dummy load.
- T_hours = 2000 * 0.85 / I_mean
- [Power Profiles of the Original MeshCore Firmware for Heltec V3, V4.2](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/blob/main/Power-Profiles-of-the-Original-MeshCore-Firmware-for-Heltec-V3.md)

---

## Bypass External LNA on Heltec V4.2
If you encounter poor RX sensitivity or an abnormally high noise floor on the Heltec V4.2, please perform the following mod to bypass the external LNA as shown in the images below. This modification only affects the RX path, the GC1109 Power Amplifier remains fully functional.


<img width="781" height="413" alt="Screenshot 2025-12-11 160253" src="https://github.com/user-attachments/assets/4ecc6acb-9944-4ba5-b756-e156f4be1dd2" />

![IMG_6926](https://github.com/user-attachments/assets/3eb8a0a4-f4c4-4ecd-bb6a-45ec6e796533)

Compare Heltec V3 and Heltec V4.2 receive sensitivity after the mod:

<img width="522" height="404" alt="bypassed_external_lna" src="https://github.com/user-attachments/assets/a53c0f4d-eea5-416a-a8c9-9d8f50135dbf" />

---

## License

The files in this repository are licensed under the [MIT License](LICENSE).

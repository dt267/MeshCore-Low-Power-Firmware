# MeshCore Low Power Firmware
MeshCore firmware with deep power optimization, a full companion display UI, and advanced radio and network controls — built for multi-day off-grid operation.

**Supported devices:** 
- Heltec WiFi LoRa 32 V3, WSL3
- Heltec WiFi LoRa 32 V4.2, V4.3 (with or without OLED display)
- Heltec Vision Master E213 (2.13" e-ink)
- Heltec Vision Master E290 (2.9" e-ink)
- Heltec Wireless Paper (2.13" e-ink)
- Heltec Mesh Node T096
- Seeed Studio XIAO ESP32S3 & Wio-SX1262 Kit
- RAK4631 WisBlock
- ...


[![GitHub Release](https://img.shields.io/github/v/release/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4)](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases/latest) [![GitHub Release Date](https://img.shields.io/github/release-date/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4)](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases/latest)

## Table of Contents
- [What's New](#whats-new)
- [Installation](#installation)
- [Idle Battery Life Estimation](#idle-battery-life-estimation-2000-mah-battery)
- [Power Profiles: Heltec T096](#heltec-t096)
- [Power Profiles: Heltec V4.3](#heltec-v43)
- [Power Profiles: Heltec Lora 32 V3](#heltec-lora-32-v3)
- [Power Profiles: Heltec Lora 32 V4.2](#heltec-lora-32-v42)
- [Power Profiles: Seeed Studio XIAO ESP32S3 & Wio-SX1262](#seeed-studio-xiao-esp32s3--wio-sx1262)
- [Power Profiles: RAK4631 (RAK19003)](#rak4631-rak19003)
- [Bypass External LNA on Heltec V4.2](#bypass-external-lna-on-heltec-v42)
- [Stock Antennas SWR Testing](Stock_Antennas_SWR_Testing.md)
- [Companion Display UI Guide](Companion_Display_Guide.md)
- [Companion TerminalCLI Commands](Companion_TerminalCLI_Commands.md)
- [Repeater / Room Server CLI Commands](Repeater_CLI_Commands.md)
- [License](#license)

## What's New

### v1.16_0614

- **New devices: Heltec Vision Master E213, Wireless Paper, and Vision Master E290 — full e-ink companion support (Companion, Repeater, Room Server).**

  Three e-ink boards are now fully supported with the complete companion UI — Quick Send, Contacts, Settings, Saved Locations, GPS Trace, and message preview. Repeater and Room Server firmware are provided for all three.

  - **Heltec Vision Master E213** — ESP32-S3 with a 2.13" e-ink display (250×122 px), SX1262 LoRa, and a QuickLink I2C port. Has a second user button (GPIO21): press to scroll up in any list, or to go back — faster than double clicking the main button.
  - **Heltec Wireless Paper** — ESP32-S3 with the same 2.13" e-ink panel as the E213 in a more compact form factor. Shares the same companion UI and firmware variants as the E213. Single button only.
  - **Heltec Vision Master E290** — ESP32-S3R8 with a larger 2.9" e-ink display (296×128 px), SX1262 LoRa, and a QuickLink I2C port. Also has a second button (GPIO21) like the E213. The companion UI adapts automatically to the wider panel — more message lines and a larger clock on the Home screen.

  **Common characteristics across all three e-ink boards:**
  - **Always-on display** — e-ink retains content indefinitely without power; no screen timeout.
  - **Native multilingual text** — Latin, Cyrillic, and Greek scripts render natively.
  - **Font Weight setting** — choose between **Thin** and **Bold** via **Settings → Font Weight**. Preference is saved to flash.
  - **I2C sensor support** — environment sensors can be connected via the QuickLink I2C port.

- **Companion: unified BLE / USB / WiFi connection mode for all ESP32-S3 boards.**

  All ESP32-S3 companion builds (Heltec V3, V4, E213, Wireless Paper, E290, XIAO S3) now ship as a single unified firmware image that supports all three connection transports: **BLE**, **USB serial**, and **WiFi TCP**. The active mode is saved to flash and selected at boot — no per-mode build is needed.

  Switching is done via **Settings → Connection Mode**, which opens a direct selection screen listing all three modes with the current one marked `*`. Navigate to the desired mode and confirm — the device reboots into the new mode.

  Alternatively, switch via [TerminalCLI](Companion_TerminalCLI_Commands.md): `set conn.mode wifi|ble|usb`. WiFi credentials are configured with `set wifi.ssid` / `set wifi.password`, or from the **OTA update page** which now includes a WiFi credentials form alongside the firmware upload button.

  In WiFi mode, the node connects as a STA to your router and the Home screen shows the IP address and port. All three modes meet the low-power criteria of this repo.  

- **Companion UI: Home screen always shows a large clock; message count shown in the header.**

  The center of the Home screen now always shows the current time (`HH:MM`) in a large font — `MSG: N` is gone. If there are messages stored in memory, their count and a small envelope icon appear in the header (where the small clock used to be); the header is left empty when count is zero. The header clock is visible again on all other pages as before.

  On e-ink displays (E213, Wireless Paper, E290), the clock is rendered in a large font; pairing pin or connection status appears at the bottom-left, and the date at the bottom-right. On OLED and T096, the layout is: large clock → date → connection status, stacked top to bottom.

  Before the clock is synchronized with the app or GPS, the display shows uptime counting up from `00:00` since boot, consistent with the small header clock.

- **Repeater/Room Server: flood hop limits now correctly ignore leading-zero path padding.**

  Some senders limit how far a packet propagates by pre-filling the path with zero entries — a TTL trick used by custom firmware, and by companions in this firmware via `ch.hops`. Previously, repeaters counted these zeros as real relay hops, causing packets to be dropped too early or assigned incorrect retransmit priority when any of the flood hop limits (`flood.max`, `flood.max.unscoped`, `advert.hops.max`, `group.hops.max`) was in use. All policy checks now count only actual relay hops, ignoring the leading-zero prefix. In addition, `flood.max.unscoped` is now automatically capped when `flood.max` is reduced or `path.hash.mode` is changed, consistent with the other hop limits.

- **Note on filenames:** Previously, companion firmware was released as separate per-transport builds — the BLE build had `_ble` in its filename (e.g. `Heltec_v3_companion_radio_ble_v1.16.dev_0607.bin`, `RAK_4631_companion_radio_ble_v1.16.dev_0607.uf2`). Since BLE + USB + WiFi are now unified into a single build for ESP32-S3, and BLE + USB for nRF52, the `_ble` suffix is gone. Companion filenames are now simply `Heltec_v3_companion_radio_v1.16.dev_0614.bin` and `RAK_4631_companion_radio_v1.16.dev_0614.uf2`.

### v1.16_0607

- **Companion UI: Native multilingual display on Heltec T096 (ST7735S)**

  Node names and messages now render natively on the T096 color TFT — characters are displayed as-is instead of being converted to ASCII equivalents. Supported languages include all 31 from v1.14 plus additional scripts:

  **Latin-based:** Catalan, Croatian, Czech, Danish, Dutch, Estonian, Finnish, French, German, Hungarian, Icelandic, Italian, Latvian, Lithuanian, Maltese, Norwegian, Polish, Portuguese, Romanian, Slovak, Slovenian, Spanish, Swedish, Turkish, Vietnamese, Welsh

  **Cyrillic:** Belarusian, Bulgarian, Macedonian, Russian, Serbian, Ukrainian

  **Greek:** Greek

- **Companion UI: Contacts screen now includes room servers; request telemetry from any contact.**

  Room servers appear in the contact list tagged `[R]`. Selecting any contact — chat node or room server — and long pressing opens an action menu: **Send message** or **Request telemetry**. Requesting telemetry displays the node's battery voltage and GPS coordinates (if present); long press to open the GPS Trace screen for that location.

  GPS coordinates follow the **Pos. Format** selected in Settings (DD, UTM, or MGRS).

  The message inbox group list now tags group/channel entries with `[G]` and room server entries with `[R]` for consistent visual distinction. Each room message shows the original author's name. When viewing a room message, long press to open the popup and select **Reply** to post back to the room (visible to all subscribers).

  > **Private room servers:** messaging requires a prior login. Public rooms (no password) work without any login. For private rooms, log in once via the MeshCore app — if your account has admin rights on that room server, the session persists across reboots. Regular user sessions are not saved to flash and will require re-login after the room server reboots.

- **Companion: switch the app connection between BLE and USB serial without reflashing.**

  In USB mode the node behaves like a standard `usb` build — the PC app connects over the USB serial port directly. Toggle via **Settings → Connection Mode** on the display, or via TerminalCLI: `set conn.mode usb` / `set conn.mode ble`. The setting persists across reboots. BLE toggle is hidden in the Settings menu while USB mode is active.

- **Repeater, Room Sever: hold user button 5 seconds to power off.**

  Hold the user button for 5 seconds to power off the node — faster than reaching for the CLI when you're standing next to it. The LED blinks 5 times as a warning before shutdown.

- **Repeater, Room Server: `advert.hops.max` default changed to 8; room server support added; `flood.max.advert` alias.**

  `advert.hops.max` now defaults to `8` instead of `flood.max` — advert relay limiting is active out of the box without any configuration. Room servers now also enforce this limit (previously repeater-only). The command `flood.max.advert` is accepted as an alias for compatibility with upstream firmware.

### v1.15_0531

- **Full support for Heltec T096 — companion, repeater, and room server**

  The Heltec T096 is now fully supported across all node types. The hardware is an nRF52840-based board with an SX1262 radio, KCT8103L FEM, UC6580 GPS, and a 0.96" 160×80 color TFT display (ST7735S).

  The complete companion UI — Quick Send, Contacts, Settings, GPS, Saved Locations, GPS Trace, and message preview — runs on the color TFT.

  **Differences from other supported boards:**

  - **Color TFT display.** All companion pages render in color on the 160×80 ST7735S — unlike other supported boards which use a monochrome OLED.

  - **Brightness control.** A **Brightness** item in the Settings page adjusts the TFT backlight intensity: `25` → `50` → `75` → `100`. Setting is saved to flash.

  - **KCT8103L FEM — same as Heltec V4.3.** `set radio.rxgain on` / `off` works identically.

  - **UC6580 GPS.** `gps.interval` works the same as on Heltec V4. Constellation selection (`gps.mode`) uses a different set of options:

    | Value | Constellations |
    |---|---|
    | `1` | GPS L1 only |
    | `2` | All-system L1 (GPS+BDS+GLO+GAL) |
    | `3` | All-system + QZSS dual-band *(default)* |

- **AGC auto-reset improvements (all node types).**

  Coordination between AGC auto-reset and channel busy detection has been improved. While the noise floor baseline is being re-established after a reset, channel sensing falls back to hardware CAD only.

- **Fix: excessive flash writes on nRF52 Companion (T096, RAK4631).**

  Each received advertisement previously triggered an immediate flash write — both to the advert blob store and to the contact list. In areas with high advert traffic this caused unnecessary flash wear, and a malicious node spamming adverts could wear out ExtraFS in hours. Advert blobs are now buffered in RAM and flushed to flash at most once every 10 minutes. Auto-discovered contacts use the same 10-minute pattern instead of a short debounce timer.

### v1.15_0524

- **Fix: radio deafness recovery in noisy environments (all node types).**

  When strong in-band interference causes the SX1262 AGC to become overwhelmed, the radio can go deaf — severely degraded in receive sensitivity. The firmware now detects this condition automatically and triggers an immediate hardware recalibration, restoring sensitivity without waiting for the scheduled `agc.reset.interval`. The `agc.reset.interval` setting remains available but is now unnecessary.

  Two new commands let you monitor and reset the AGC reset counter:

  | Command | Effect |
  |---------|--------|
  | `get agc.resets` | Show how many times the AGC has been auto-reset since boot or last `clear` |
  | `clear agc.resets` | Reset the counter to zero |

  **Real-world validation:** A repeater installed near a periodic in-band interference source (confirmed via RTL-SDR) accumulated 918 auto-resets over 8.5 hours (~1.8/min), matching the observed interference sweep cycle. A second repeater at a clean location recorded 0 resets over the same period, confirming no false positives. Without this feature, the first repeater was unreachable remotely due to persistent deafness.

- **GPS update interval configurable at runtime on Heltec V4 (all node types).**

  A new `gps.interval` setting controls the sleep time between GPS position updates. After each fix, GPS powers down for the configured interval, then wakes and acquires a new fix. Set it to `0` to keep GPS always on. Default is 10 seconds.

  Available on **Companion** via [TerminalCLI](Companion_TerminalCLI_Commands.md) and on **Repeater / Room Server** via the Command Line:

  | Command | Effect |
  |---|---|
  | `get gps.interval` | Show current interval (`always on` if 0) |
  | `set gps.interval 0` | GPS always on — maximum accuracy, highest power draw |
  | `set gps.interval 30` | GPS sleeps 30s after each fix, then re-acquires |
  | `set gps.interval 300` | GPS sleeps 5 minutes after each fix |

  Setting is saved to flash and takes effect immediately — no reboot needed.

- **GPS constellation selection for L78K on Heltec V4 (`gps.mode`) (all node types).**

  Choose which satellite constellations the L78K module tracks. Average current draw is essentially the same across all configurations in duty cycle mode with `gps.interval` greater than 10 (~23 mA mean current measured on Heltec V4.3 Companion with `gps.mode = 4` and `gps.interval = 10`). Leave at the default `4` for the most robust fix; adjust only if you have a specific coverage reason.

  | Value | Constellations |
  |---|---|
  | `1` | GPS only |
  | `2` | GPS + BeiDou |
  | `3` | GPS + GLONASS |
  | `4` | GPS + BeiDou + GLONASS *(default)* |

  Available on **Companion** via [TerminalCLI](Companion_TerminalCLI_Commands.md) and on **Repeater / Room Server** via the Command Line:

  | Command | Effect |
  |---|---|
  | `get gps.mode` | Show current constellation selection |
  | `set gps.mode 4` | GPS + BeiDou + GLONASS (default) |
  | `set gps.mode 1` | GPS only |

  Setting is saved to flash. Change takes effect on next GPS on.

- **Companion: GPS screen shows interval and constellation when GPS is off.**

  When GPS is off, the GPS page now shows the configured update interval (`intv`) and constellation selection (`mode`, Heltec V4 only), so you can verify settings before enabling GPS.

### v1.15_0517

- **Companion: per-channel outgoing hop limit (`ch.hops`).**

  Limit how far your outgoing messages travel on a specific channel, without affecting any other channel or any other node in the network. Useful for keeping a private or local-area channel confined to a small radius — for example, a home-to-nearby-relay link — without touching the rest of the city mesh.

  Configure via **TerminalCLI**:

  | Command | Effect |
  |---|---|
  | `set ch.hops <channel> <N>` | Limit outgoing messages on `<channel>` to at most N hops |
  | `set ch.hops <channel> off` | Remove the limit |
  | `get ch.hops <channel>` | Show current limit |
  | `ch.hops status` | List all channels with an active limit |
  | `ch.hops clear` | Remove all limits |

  ```
  set ch.hops Public 2         # Public channel messages travel at most 2 hops
  set ch.hops My Local Net 3   # My Local Net channel travel at most 3 hops
  set ch.hops Public off       # remove the limit
  ch.hops status
  ```


  The limit applies to **your outgoing messages only** — other nodes sending on the same channel are unaffected. No repeater configuration or network coordination required.

  The repeater's `group.hops.max` algorithm has been updated to correctly account for sender-side pre-fill, so `ch.hops` and `group.hops.max` work together without conflict — update repeater firmware to get full compatibility.

  Settings are stored in flash and persist after reboot.

- **Companion: toggle RxGain directly from the Radio screen.**

  Long press on the **Radio** page cycles through RxGain modes (OFF → ON → Auto) and shows a popup confirming the new mode. The current mode is now also displayed on the Radio page itself (`RxG: OFF` / `RxG: ON` / `RxG: Auto`), so you can check it at a glance without going into Settings. *(`Auto` is available on Heltec V4.2 only.)*

### v1.15_0510

- **Repeater: per-type relay hop cap (`advert.hops.max` / `group.hops.max`).**

  Two new settings let you independently limit how far **advertisement packets** and **group messages** are relayed across the mesh, without affecting direct messages, ACKs, or path discovery.

  | Setting | Controls | Default |
  |---|---|---|
  | `advert.hops.max` | Max hops to relay node advertisement (ADVERT) packets | = `flood.max` |
  | `group.hops.max` | Max hops to relay group messages (GRP_TXT / GRP_DATA) | = `flood.max` |

  These are **repeater-only** settings. Configure via the **Command Line** in the MeshCore App:
  ```
  set advert.hops.max 3    # relay adverts at most 3 hops from sender
  set group.hops.max 5     # relay group messages at most 5 hops
  get advert.hops.max
  get group.hops.max
  ```

  Setting either value to `0` completely suppresses that packet type — no relay at all, while everything else (DM, ACK, path) continues to work normally.

  **Why this matters — advert storm reduction:**

  Each repeater periodically broadcasts an advertisement that every other repeater in range relays, up to `flood.max` times (default 64). In a network of 35 repeaters and 35 companions, advert traffic alone consumes roughly 4% of airtime. Limiting to 3 hops brings that down to under 0.5% — a 7× reduction — with no loss of communication capability.  

  **No coordination required.** Each repeater applies its own cap independently. You do not need all repeaters to agree on the same value — even a partial deployment reduces airtime. No region configuration, no App UI changes needed.

  **Recommended profiles:**

  | Profile | `advert.hops.max` | `group.hops.max` | Airtime (35R / 35C) | Use case |
  |---|---|---|---|---|
  | Default | = `flood.max` | = `flood.max` | ~4% | No change |
  | Optimized | 3 | 5 | ~0.5% | Most deployments |
  | DM-focused | 0 | 3 | ~0.2% | Prioritise direct messages |
  | EU compliance | 0 | 3 | ~0.2% | Large EU networks (legal requirement) |

  > **Relationship with `flood.max` and path hash size:** `flood.max` is the master hop cap for **all** flood payloads. Its default of 64 is calibrated for 1-byte path hash mode. With larger hashes the packet fills up sooner — 2-byte hash caps at 32 hops, 3-byte hash caps at 21 hops — so `flood.max` is typically set to match. `advert.hops.max` and `group.hops.max` are automatically clamped to `flood.max` and work correctly with all hash sizes. If you want to suppress adverts entirely while keeping everything else unrestricted, set `advert.hops.max 0` — leave `flood.max` untouched.

- **Repeater: combining hop caps with region scope for full community isolation.**

  `advert.hops.max` and `group.hops.max` work standalone — no coordination needed. For operators who also want to restrict which community's group messages get relayed, combine them with region rules and the companion's scope settings:

  1. Each companion sets a region scope — either per-channel via the burger menu, or globally via **Settings → Experimental → Default Region Scope**. All outgoing flood packets (including DM path discovery) are tagged with the community name.
  2. Each repeater: `denyf *` + `allowf <community>` — only relay community-tagged traffic.
  3. Add `set advert.hops.max 0` to also suppress advert relay.

  With a global Default Region Scope, DM path discovery is tagged and passes through `denyf *` repeaters — direct messaging works normally inside the community.

  > **Default Region Scope** is in the App's Experimental Settings and is opt-in. Per-channel scope (burger menu) only tags group messages for that channel, not advertisements or DM path discovery.

- **Companion: GPS coordinate display formats — DD / UTM / MGRS.**

  A new **Pos. Format** item in the Settings page lets you choose how GPS coordinates are shown on the GPS page, GPS Trace screen, and Quick Send status bar.

  | Format | Example |
  |---|---|
  | DD *(default)* — Decimal Degrees | `10.7769  106.7009` |
  | UTM — Universal Transverse Mercator | `48P 681978E 1190975N` |
  | MGRS — Military Grid Reference System | `48PXS8197890975` |

  Setting is saved to flash and persists after reboot.

  > **Sharing coordinates with others.** Coordinates attached to messages are always sent in DD form, no matter which Pos. Format you chose. Each receiver sees them in the format they set on their own device — so you can read MGRS while your friend reads DD on the same shared point. Same applies to saved locations and to the TerminalCLI `loc` commands: input and storage stay in DD; only the on-screen display changes.

### v1.15_0426

- **Companion: redesigned message preview.**

  Messages are now grouped by sender or channel so you can read a conversation in one place instead of hunting through a mixed list.

  From the **group list** you can scroll back through older messages from any sender or channel, including ones you've already read. Long press a group to open it.

  When new messages arrive, you go straight to a **new messages view** that shows only the unread ones — from all senders and channels — one by one. Once you've read them all, a single click takes you home.

  Long messages scroll one full page at a time with no overlap. Small arrows (`▲` / `▼`) appear at the corner of the screen to let you know there is more content above or below.

- **Companion: reply directly from message preview.**

  While reading any message, long press to open the menu. A new **Reply** option lets you send a preset message back without leaving the screen:

  - If the message came from a contact, the reply goes to that contact as a private direct message.
  - If the message came from a channel (Public, #SOS, or any other), the reply goes back to that same channel.

  > **Note:** The **Quick Send** page always sends to the Public channel. Use Reply from the message preview when you want to respond to a specific contact or a non-Public channel.

- **Companion: Contacts page and direct messaging.**

  A new **Contacts** page sits between Quick Send and Saved Locations. Long press it to open the contact list, only chat-capable nodes are listed (repeaters, room servers and sensors are excluded — they cannot receive direct messages). Select a contact and long press to send them a direct message using your Quick Send presets.

- **Companion: configurable Screen Off timeout in Settings.**

  A new **Screen Off** item in the Settings page lets you choose how long the display stays on after the last button press: `15s` → `3min` → `Never`. Setting is saved to flash and persists after reboot.

- **Companion: Flip Screen setting in Settings.**

  A new **Flip Screen** toggle in the Settings page rotates the display 180°. Setting is saved to flash and persists after reboot.

### v1.15_0419

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

> **Unified binary:** A single firmware file runs on both OLED and no-display hardware variants — no separate build required. The display is detected automatically at boot via I2C probe. The device name shown in the MeshCore app reflects the actual hardware detected (e.g. *Heltec V4.3 No Display* vs *Heltec V4.3 OLED*).

Two binary formats are provided for each firmware variant:

| File | Contents | Use case |
| :--- | :--- | :--- |
| `<name>.bin` | Application only | Update / OTA — preserves existing partitions |
| `<name>_merged.bin` | Bootloader + partition table + application | First-time install or full recovery |

**Option 1: Full flash (merged binary) — recommended for first-time install**

Flash the `_merged.bin` file starting at address `0x0`. This is self-contained and requires no prior MeshCore installation.
```
python -m esptool --chip esp32s3 write_flash 0x0 <name>_merged.bin
```
> **Note:** Full flash erases the NVS partition, which stores BLE pairing keys — you will need to re-pair BLE devices after flashing. Settings stored in the SPIFFS filesystem partition are beyond the merged binary range and are **not** affected.

**Option 2: Application update via esptool**

Flash the plain `<name>.bin` file at `0x10000`. The existing bootloader and partition table are preserved.
```
python -m esptool --chip esp32s3 write_flash 0x10000 <name>.bin
```

**Option 3: Wi-Fi OTA** *(requires v1.14_0320 or later)*
Type `start ota` via TerminalCLI (Companion) or Command Line (Repeater / Room Server) → connect to `MeshCore-OTA` Wi-Fi → go to `192.168.4.1/update`. Upload the plain `<name>.bin` file only — the merged binary is **not** compatible with OTA.

### RAK4631 / Heltec T096 (nRF52840)

**Option 1: UF2 drag-and-drop**
Double-tap the Reset button → a USB drive appears → copy the `.uf2` file onto it. The device reboots automatically when done.
- **RAK4631:** drive is named `RAK4631`
- **Heltec T096:** drive is typically named `HT-n5262G`

**Option 2: BLE DFU** *(requires v1.14_0320 or later)*
Enter DFU mode, then use the **nRF Device Firmware Update** app (iOS/Android) to upload the `.zip` DFU package.

Ways to enter DFU mode:
- **RAK4631:** Type `start ota` via TerminalCLI (Companion) or Command Line (Repeater / Room Server)
- **Heltec T096:** Hold the user button while pressing Reset — or type `start ota` via TerminalCLI / Command Line — or on Companion: **Settings → Start OTA**


## Idle Battery Life Estimation (2000 mAh battery)

| Device | Idle Current (mA) | Estimated Idle Runtime (Hours) | Estimated Idle Runtime (Days) |
| :--- | :--- | :---: | :---: |
| Heltec T096 Companion BLE, KCT8103L LNA off, SX1262 Boosted Gain on | 7.4 | 229.7 | 9.6 |
| Heltec T096 Repeater, KCT8103L LNA off, SX1262 Boosted Gain on | 7.4 | 229.7 | 9.6 |
| Heltec V4.3 Companion BLE, KCT8103L LNA on, SX1262 Boosted Gain off | 22.9 | 74.2 | 3.1 |
| Heltec V4.3 Companion BLE, KCT8103L LNA off, SX1262 Boosted Gain on | 15.4 | 110.4 | 4.6 |
| Heltec V4.3 Companion, BLE off, KCT8103L LNA off, SX1262 Boosted Gain on | 11.5 | 147.8 | 6.2 |
| Heltec V4.3 Repeater, KCT8103L LNA on, SX1262 Boosted Gain off | 16.1 | 105.6 | 4.4 |
| Heltec V4.3 Repeater, KCT8103L LNA off, SX1262 Boosted Gain on | 8 | 212.5 | 8.6 |
| Heltec V3 Companion BLE, SX1262 Boosted Gain on | 10 | 170.0 | 7.08 |
| Heltec V3 Repeater, SX1262 Boosted Gain on | 7.8 | 217.9 | 9.08 |
| Heltec V3 Room Server, SX1262 Boosted Gain on | 8.0 | 212.5 | 8.85 |
| Heltec WSL3 Companion BLE, SX1262 Boosted Gain on | 10 | 170.0 | 7.08 |
| Heltec WSL3 Repeater, SX1262 Boosted Gain on | 7.7 | 220.8 | 9.20 |
| Heltec WSL3 Room Server, SX1262 Boosted Gain on | 7.9 | 215.2 | 8.97 | 
| Heltec V4.2 Companion BLE, SX1262 Boosted Gain off | 20 | 85.0 | 3.54 |
| Heltec V4.2 Repeater, SX1262 Boosted Gain off | 13.3 | 127.8 | 5.33 | 
| Heltec V4.2 Room Server, SX1262 Boosted Gain off | 13.4 | 126.9 | 5.29 |
| XIAO S3 Wio Companion BLE, SX1262 Boosted Gain on | 11 | 154.5 | 6.44 |
| XIAO S3 Wio Repeater, SX1262 Boosted Gain on | 8.7 | 195.4 | 8.14 |
| XIAO S3 Wio Room Server, SX1262 Boosted Gain on | 8.7 | 195.4 | 8.14 |
| RAK4631 Companion BLE, SX1262 Boosted Gain on | 6.61 | 257.18 | 10.71 |
| RAK4631 Repeater, SX1262 Boosted Gain on | 5.79 | 293.6 | 12.23 |

---

## Heltec T096

### *Typical power profile of Heltec T096 BLE companion, 5 LoRa messages in 30 seconds (14,400 messages a day):*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 621.9 mA
* Minimum: - mA
* Mean: 80.2 mA

**Estimated ~21.2 h (0.9 days) with 2000mAh battery.**

<img alt="t096_com_h" src="https://github.com/user-attachments/assets/e37f0a8a-2b9a-4325-83e0-3b0560621e77" />

### *Typical power profile of Heltec T096 BLE companion in idle:*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 13.6 mA
* Minimum: 5.8 mA
* Mean: 7.4 mA

**Estimated ~229.7 h (9.6 days) with 2000mAh battery.**

<img alt="t096_com_l" src="https://github.com/user-attachments/assets/40177031-c213-443c-a327-9fb3492f8c6e" />

### *Typical power profile of Heltec T096 repeater in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 637.5 mA
* Minimum: - mA
* Mean: 85.1 mA

**Estimated ~20 h (0.8 days) with 2000mAh battery.**

<img alt="t096_rep_h" src="https://github.com/user-attachments/assets/46f5f721-758f-4859-bca8-ef0016cfb511" />

### *Typical power profile of Heltec T096 repeater in idle:*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 10 mA
* Minimum: 5 mA
* Mean: 7.4 mA
  
**Estimated ~229.7 h (9.6 days) with 2000mAh battery.**

<img alt="t096_rep_l" src="https://github.com/user-attachments/assets/fa14d784-322d-4294-9f90-476661cd1a6e" />

## Heltec V4.3

### *Typical power profile of Heltec V4.3 BLE companion, 5 LoRa messages in 30 seconds (14,400 messages a day):*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 632.5 mA
* Minimum: 9.2 mA
* Mean: 76.2 mA

**Estimated ~22.3 h (0.93 days) with 2000mAh battery.**

<img alt="v43_companion_ble_on_lna_off_high" src="https://github.com/user-attachments/assets/e4433853-d9df-4366-b56f-8b1930f8aa1b" />

### *Typical power profile of Heltec V4.3 BLE companion in idle:*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 105.7 mA
* Minimum: 9.7 mA
* Mean: 15.4 mA

**Estimated ~110.4 h (4.6 days) with 2000mAh battery.**

<img alt="v43_companion_ble_on_lna_off_idle" src="https://github.com/user-attachments/assets/2334bd0f-fce1-41ba-a49f-ee049985463f" />

### *Typical power profile of Heltec V4.3 repeater in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 598.3 mA
* Minimum: 5.8 mA
* Mean: 82.2 mA
  
**Estimated ~20.7 h (0.86 days) with 2000mAh battery.**

<img alt="v43_repeater_lna_off_high" src="https://github.com/user-attachments/assets/89b7e995-1b9b-4647-8611-7987dfed54b4" />

### *Typical power profile of Heltec V4.3 repeater in idle:*

Firmware 1.15.dev, KCT8103L LNA off, SX1262 Boosted Gain on.

* Maximum: 47 mA
* Minimum: 6.3 mA
* Mean: 8 mA
  
**Estimated ~212.5 h (8.9 days) with 2000mAh battery.**

<img alt="v43_repeater_lna_off_idle" src="https://github.com/user-attachments/assets/62eb1d6a-4706-4828-b66c-4cf3808412b0" />

## Heltec Lora 32 V3 

<img alt="helec-v3" src="https://github.com/user-attachments/assets/2a4b7faf-0420-4d31-bcbe-47b970810f22" />


### *Typical power profile of Heltec V3 BLE companion, 5 LoRa messages in 30 seconds (14,400 messages a day):*

Firmware 1.13.dev, SX1262 Boosted Gain on.

* Maximum: 241.48 mA
* Minimum: 4.99 mA
* Mean: 31.37 mA

**Estimated ~54.19 h (2.57 days) with 2000mAh battery.**

<img alt="dt267-v3-113-companion-h" src="https://github.com/user-attachments/assets/3e79cd31-87a9-4366-9576-2013b8ae7469" />


### *Typical power profile of Heltec V3 BLE companion in idle:*

Firmware 1.13.dev, SX1262 Boosted Gain on.

* Maximum: 102.97 mA
* Minimum: 5.72 mA
* Mean: 9.22 mA

**Estimated ~184.38 h (7.68 days) with 2000mAh battery.**

<img alt="dt267-v3-113-companion-l" src="https://github.com/user-attachments/assets/d0c5bb68-4136-481c-80b7-b1adfe515687" />

### *Typical power profile of Heltec V3 repeater in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*

Firmware 1.13.dev, SX1262 Boosted Gain on.

* Maximum: 160.49 mA
* Minimum: 2.01 mA
* Mean: 27.74 mA
  
**Estimated ~61.28 h (2.55 days) with 2000mAh battery.**

<img alt="dt267-v3-113-repeater-h" src="https://github.com/user-attachments/assets/54f806e7-b197-47fa-a054-19e9d72cad98" />

### *Typical power profile of Heltec V3 repeater in idle:*

Firmware 1.13.dev, SX1262 Boosted Gain on.

* Maximum: 44.28 mA
* Minimum: 5.51 mA
* Mean: 7.25 mA
  
**Estimated ~234.48 h (9.77 days) with 2000mAh battery.**

<img alt="dt267-v3-113-repeater-l" src="https://github.com/user-attachments/assets/9e8511de-53c9-40e2-89db-eb517d3dcf46" />

---

## Heltec Lora 32 V4.2

<img alt="v4001" src="https://github.com/user-attachments/assets/658c4b3a-edca-451f-b4d9-d4a75e927d7c" />

### *Typical power profile of Heltec V4.2 BLE companion, 5 LoRa messages in 30 seconds (14,400 messages a day):*

Firmware 1.13.dev, SX1262 Boosted Gain off.

* Maximum: 734.11 mA
* Minimum: 14.31 mA
* Mean: 96.64 mA

**Estimated ~17.59 h (0.73 days) with 2000mAh battery.**

<img alt="dt267-v4.2-companion-h" src="https://github.com/user-attachments/assets/80f1746f-9cde-4b69-a8db-3dd9056dc87c" />

### *Typical power profile of Heltec V4.2 BLE companion in idle:*

Firmware 1.13.dev, SX1262 Boosted Gain off.

* Maximum: 107.92 mA
* Minimum: 14.93 mA
* Mean: 19.85 mA

**Estimated ~85.64 h (3.56 days) with 2000mAh battery.**

<img alt="dt267-v4.2-companion-l" src="https://github.com/user-attachments/assets/c042c2d9-2940-433f-a51c-0f901fa9ecaa" />

### *Typical power profile of Heltec V4.2 repeater in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*

Firmware 1.13.dev, SX1262 Boosted Gain off.

* Maximum: 681.08 mA
* Minimum: 11.07 mA
* Mean: 108.18 mA
  
**Estimated ~15.7 h (0.65 days) with 2000mAh battery.**

<img alt="dt267-v4.2-repeater-h" src="https://github.com/user-attachments/assets/73b0e1a5-e5fe-451c-ad1f-86396202e040" />

### *Typical power profile of Heltec V4.2 repeater in idle:*

Firmware 1.13.dev, SX1262 Boosted Gain off.

* Maximum: 53.55 mA
* Minimum: 11.74 mA
* Mean: 13.57 mA
  
**Estimated ~125.27 h (5.2 days) with 2000mAh battery.**

<img alt="dt267-v4.2-repeater-l" src="https://github.com/user-attachments/assets/3fe07ea7-d733-4df6-aaa0-7132c9307936" />

---

## Seeed Studio XIAO ESP32S3 & Wio-SX1262

![wio-sx1262-with-xiao-esp32s3](https://github.com/user-attachments/assets/615d1c65-fdb9-4769-acf4-5e9680d1a009)

### *Typical power profile of XIAO S3 Wio BLE companion, 5 LoRa messages in 30 seconds (14,400 messages a day):*

Firmware 1.12.dev, SX1262 Boosted Gain on.

* Maximum: 274 mA
* Minimum: 4.07 mA
* Mean: 35.53 mA

**Estimated ~47.9 h (1.99 days) with 2000mAh battery.**

<img alt="xiao-c-h" src="https://github.com/user-attachments/assets/da6005c2-1afa-4418-83d2-da65b6d7b371" />

### *Typical power profile of XIAO S3 Wio BLE companion in idle:*

Firmware 1.12.dev, SX1262 Boosted Gain on.

* Maximum: 127.38 mA
* Minimum: 9.47 mA
* Mean: 15.34 mA

**Estimated ~110.8 h (4.62 days) with 2000mAh battery.**

<img alt="xiao-c-l" src="https://github.com/user-attachments/assets/291cf991-2cee-47c1-96c1-c55ec0368545" />

### *Typical power profile of XIAO S3 Wio repeater in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*

Firmware 1.12.dev, SX1262 Boosted Gain on.

* Maximum: 141.1 mA
* Minimum: 0.51 mA
* Mean: 24.57 mA
  
**Estimated ~69.2 h (2.88 days) with 2000mAh battery.**

<img alt="xiao-r-h" src="https://github.com/user-attachments/assets/c859735b-a7ea-4d85-8d42-df7ef25249d9" />

### *Typical power profile of XIAO S3 Wio repeater in idle:*

Firmware 1.12.dev, SX1262 Boosted Gain on.

* Maximum: 25.59 mA
* Minimum: 4.53 mA
* Mean: 7.07 mA
  
**Estimated ~240.5 h (10.02 days) with 2000mAh battery.**

<img alt="xiao-r-l" src="https://github.com/user-attachments/assets/280c8b70-604b-4ea3-b1c4-76f21efc2c24" />

---

## RAK4631 (RAK19003)

![rak19003](https://github.com/user-attachments/assets/e95d138e-c4c4-4727-bfbb-860b077af8d3)

### *Typical power profile of RAK4631 BLE companion, 5 LoRa messages in 30 seconds (14,400 messages a day):*

Firmware 1.14.dev, SX1262 Boosted Gain on.

* Maximum: 104.16 mA
* Minimum: 6.7 mA
* Mean: 19.2 mA

**Estimated ~88.5 h (3.69 days) with 2000mAh battery.**

<img alt="rak4631-companion-h" src="https://github.com/user-attachments/assets/0c03c8ae-8104-49b5-9984-0b08b09643c2" />

### *Typical power profile of RAK4631 companion BLE in idle:*

Firmware 1.14.dev, SX1262 Boosted Gain on.

* Maximum: 10.92 mA
* Minimum: 5.25 mA
* Mean: 6.61 mA

**Estimated ~257.18 h (10.71 days) with 2000mAh battery.**

<img alt="rak4631-companion-l" src="https://github.com/user-attachments/assets/c18b00fd-a722-4e01-a852-c6bcda8d6ff6" />

### *Typical power profile of RAK4631 repeater in high LoRa traffic, 6 LoRa messages in 30 seconds (17,280 messages a day):*

Firmware 1.14.dev, SX1262 Boosted Gain on.

* Maximum: 100.3 mA
* Minimum: 0.2 mA
* Mean: 20.23 mA
  
**Estimated ~84.03 h (3.5 days) with 2000mAh battery.**

<img alt="ra4631-repeater-h" src="https://github.com/user-attachments/assets/2934f559-93c2-439c-aca7-3b6705fc7576" />

### *Typical power profile of RAK4631 repeater in idle:*

Firmware 1.14.dev, SX1262 Boosted Gain on.

* Maximum: 7.88 mA
* Minimum: 4.53 mA
* Mean: 5.79 mA
  
**Estimated ~293.6 h (12.23 days) with 2000mAh battery.**

<img alt="ra4631-repeater-l" src="https://github.com/user-attachments/assets/37951209-5fa6-4a37-9ff3-d91ee58fd820" />

---

## Note:
- Heltec V3's LoRa Tx is 22dBm into dummy load.
- Heltec V4's LoRa Tx is 28dBm into dummy load.
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

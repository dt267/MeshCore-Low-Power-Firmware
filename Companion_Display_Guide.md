# Companion Radio — Display & Button Guide

How to use the OLED display and button on your Companion Radio node — no phone required.

---

## Button basics

One button does everything:

| Press | Action |
|---|---|
| **Single click** | Next page / next item / scroll down |
| **Double click** | Previous page / exit sub-level |
| **Long press** | Open message preview (Home) · Enter sub-level (Quick Send / Settings) · Toggle GPS (GPS page) |

---

## Home screen pages

Single click and double click cycle through pages forward and backward.

| Page | What it shows |
|---|---|
| **Home** | Message count, connection status, battery |
| **Quick Send** | Send a preset message over LoRa |
| **Saved Locations** | Preview of saved map locations |
| **Recent Advert** | Last 4 nodes heard over LoRa |
| **Radio** | LoRa radio parameters and noise floor |
| **GPS** | GPS status and coordinates *(if GPS hardware present)* |
| **Settings** | Device settings |

---

## Home page

```
┌──────────────────────────────┐
│ MyNodeName        14:32 [=]  │
│           · • · · · ·        │
│                              │
│          MSG: 5              │
│                              │
│       < Connected >          │
│         14 Apr 2026          │
└──────────────────────────────┘
```

- Node name at top-left; `HH:MM` and battery icon packed together at top-right
- Page indicator dots in the middle row (filled = current page)
- **MSG: N** — number of messages stored in memory (up to 256)
- Connection status line:
  - `< Connected >` — app connected via BLE
  - `BLE: OFF` — Bluetooth disabled
  - `Pin: 1234` — waiting for BLE pairing
- Date at the bottom (`DD Mon YYYY`)
- **Long press** → open message preview

> **Time and date** show local time once the timezone offset is set. Default is UTC. Configure once via TerminalCLI: `set tz.offset 7` (for UTC+7). Time is synced from the app on connect, or from GPS if hardware is present.

---

## Quick Send page

Send a preset message to the public channel without typing.

```
┌──────────────────────────────┐
│ QUICK SEND        14:32 [=]  │
│           · · • · · ·        │
│ I'm OK                       │
│ On my way                    │
│ I need help                  │
│ GPS:10.7769,106.7009         │
└──────────────────────────────┘
```

- 3 presets visible at a time (4 if no GPS); scrolls as you cycle
- Bottom line shows current GPS status:
  - `GPS:lat,lon` — valid fix, coordinates will be attached when sending
  - `?GPS:lat,lon` — no fix / using last known coords from app
  - `GPS: Private` — GPS Privacy mode is ON; coordinates will **not** be attached
- **Long press** to enter sub-level (active item highlights)

**In sub-level:**

| Press | Action |
|---|---|
| Single click | Next preset |
| Double click | Exit sub-level |
| Long press | Send highlighted preset → alert "Sent!" or "Send failed" |

GPS coordinates are appended automatically when sending (e.g. `I'm OK @10.7769,106.7009`), unless **GPS Privacy** is enabled in Settings.

**Customize presets via [TerminalCLI](Companion_TerminalCLI_Commands.md):**
```
get quick                        list all presets
set quick.0 Arrived at camp      change preset at index 0
set quick.reset                  restore all defaults
```

---

## Recent Advert page

Shows the last 4 nodes that advertised over LoRa, with time since last heard. Read-only, no interaction.

```
┌──────────────────────────────┐
│ RECENT ADVERT     14:32 [==] │
│           · · · • · ·        │
│ Alien                    42s │
│ Base Camp                 5m │
│ Repeater-1               12m │
│ Repeater-2                1h │
└──────────────────────────────┘
```

---

## Radio page

LoRa radio parameters — read-only.

```
┌──────────────────────────────┐
│ RADIO             14:32 [==] │
│           · · · · • ·        │
│ FQ: 915.000   SF: 10         │
│ BW: 250.00    CR: 5          │
│ TX: 20dBm                    │
│ Noise floor: -95             │
└──────────────────────────────┘
```

| Field | Meaning |
|---|---|
| FQ | Frequency in MHz |
| SF | Spreading factor |
| BW | Bandwidth in kHz |
| CR | Coding rate |
| TX | Transmit power in dBm |
| Noise floor | Ambient noise level in dBm |

---

## GPS page *(optional)*

Only shown if GPS hardware is present.

```
┌──────────────────────────────┐
│ GPS               14:32 [==] │
│         · · · · · • ·        │
│ gps on                   fix │
│ sat                        8 │
│ pos         10.7769 106.7009 │
│ alt                    12.50 │
└──────────────────────────────┘
```

- **Long press** — toggle GPS module on/off
- `gps on` / `gps off` — current software state
- `fix` / `no fix` — whether a valid position is available
- `sat` — number of satellites in view
- `pos` — current latitude and longitude
- `alt` — altitude

---

## Settings page

```
┌──────────────────────────────┐
│ SETTINGS          14:32 [==] │
│           · · · · · •        │
│ BLE                       ON │
│ Repeat                   OFF │
│ RxGain                    ON │
│ Units                 Metric │
└──────────────────────────────┘
```

- **Long press** to enter sub-level (active item highlights)

**In sub-level:**

| Press | Action |
|---|---|
| Single click | Next item (scrolls if more than 4) |
| Double click | Exit sub-level |
| Long press | Activate item |

| Item | Action |
|---|---|
| **BLE** | Toggle Bluetooth on/off; shows `ON, Connected` when app is connected |
| **Repeat** | Toggle packet repeat on/off |
| **RxGain** | Cycle RxGain: OFF → ON → AUTO *(AUTO: V4.2 only)* |
| **Units** | Toggles between Metric and Imperial |
| **GPS Privacy** | Toggle ON to hide GPS coords from Quick Send messages; Quick Send bottom line shows `GPS: Private` as reminder |
| **Send Advert** | Broadcast your presence → alert "Advert sent!" |
| **Start OTA** | OTA update mode — connect to WiFi `MeshCore-OTA`, go to `192.168.4.1/update` |
| **Shutdown** | Power off the device |

---

## Reading messages

Long press on the **Home** page opens message preview.

```
┌──────────────────────────────┐
│ 19/19                   42s  │
│──────────────────────────────│
│ (2) Alien:                   │
│ I need help                  │
│ @10.7769,106.7009            │
└──────────────────────────────┘
```

- **`19/19`** — viewing message #19 (newest) of 19 total; `1/19` = oldest
- **`42s`** — time since this node received the message
- **`(2) Alien:`** — from Alien, relayed through 2 hops
- **`▼`** — more text below, scroll with single click

| Press | Action |
|---|---|
| Single click | Scroll text down 3 lines; at end of message → next older message |
| Double click | Scroll text up 3 lines; at top → go to next newer message; at newest → home |
| Long press | Open menu: **Save location** (if GPS in message) / **Home** |

Up to **256 messages** are stored in memory (lost on reboot).

---

## Saving a GPS location

When viewing a message that contains GPS coordinates, long press opens a small menu:

```
┌──────────────────────────────┐
│ 19/19                   42s  │
│──────────────────────────────│
│ (2) Alien:                   │
│ I need help                  │
│ ┌────────────────────────┐   │
│ │ > Save location        │   │
│ │   Home                 │   │
│ └────────────────────────┘   │
└──────────────────────────────┘
```

Select **Save location**, then choose which slot (1–10) to save into:

```
┌──────────────────────────────┐
│ Save to slot:                │
│──────────────────────────────│
│ > 1. [empty]                 │
│   2. Base camp               │
│   3. [empty]                 │
│   4. [empty]                 │
└──────────────────────────────┘
```

| Press | Action |
|---|---|
| Single click | Move to next slot |
| Long press | Save to selected slot |
| Double click | Cancel |

Saved locations persist in flash memory — they survive reboot.

> **Tip for SOS situations**: save the location immediately. Even if the message is later overwritten by new incoming messages, your saved location remains accessible from the **Saved Locs** page.

---

## Saved locations

The **SAVED LOCS** page (navigate to it with single click from the home page) shows your saved GPS points.

```
┌──────────────────────────────┐
│ SAVED LOCS         ●         │
│──────────────────────────────│
│ Alien: I need help           │
│ Big Boy: Heading home        │
│                              │
│                              │
└──────────────────────────────┘
```

Long press the **SAVED LOCS** page to open the full list, then navigate and open a location:

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

Each entry shows **sender name + message snippet** so you can tell entries apart even when multiple locations are saved from the same person.

| Press | Action |
|---|---|
| Single click | Move highlight to next entry |
| Long press | Open GPS trace screen for highlighted entry |
| Double click | Return to home |

---

## GPS Trace screen

The GPS Trace screen shows live distance and bearing from your current position to a saved location. Open it by long-pressing an entry in the Saved Locations list.

```
┌──────────────────────────────┐
│ Alien: I need help     14:32 │
│──────────────────────────────│
│      10.7769  106.7009       │
│                              │
│            1.2km             │
│                              │
│          247°  WSW           │
└──────────────────────────────┘
```

- **Top line**: location label + `HH:MM`
- **Coordinates**: the saved GPS point (always shown)
- **Distance**: straight-line distance to the target — requires your own GPS fix
- **Bearing / cardinal direction**: direction to head toward the target

If your node has no GPS fix, only the coordinates are shown and distance/bearing are hidden.

| Press | Action |
|---|---|
| Any button | Return to Saved Locations list |

---

## Message origin labels

| Label | Meaning |
|---|---|
| `(0) Alien:` | Alien's message received with 0 relay hops (arrived directly) |
| `(2) Alien:` | Alien's message relayed through 2 nodes before arriving |
| `(D) Alien:` | Sent via a pre-discovered route (not a flood broadcast); Alien sent it specifically to this node |
| `(1) Public:` | Channel "Public" broadcast, relayed through 1 node |

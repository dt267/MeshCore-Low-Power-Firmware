# Companion Radio — Display & Button Guide

How to use the display and button on your Companion Radio node. Applies to all supported hardware — OLED (Heltec V3, V4, and the optional 0.96" SSD1306 / 1.3" SH1106 OLED on HT-CT62, XIAO ESP32S3 & Wio-SX1262 Kit and Waveshare ESP32-S3-LR1121-XF), color TFT (Heltec T096), and e-ink (Heltec E213, Wireless Paper, E290).

---

## Button basics

One button does everything:

| Press | Action |
|---|---|
| **Single click** | Next page / next item / scroll down |
| **Double click** | Previous page / back |
| **Triple click** | Previous item; where there is none, the same as double click |
| **Long press** | Open / select: the page's list or menu, the selected item; on the GPS page, turn GPS on or off |

> **Heltec E213 and E290 — second button (GPIO21):** works like a triple click. The Wireless Paper has only one button.

---

## Home screen pages

Single click cycles forward; double click and triple click both cycle backward.

| Page | What it shows |
|---|---|
| **Home** | Current time (large), date, connection status, battery |
| **Chats** | Most recent conversations |
| **Quick Send** | Send a preset message over LoRa |
| **Contacts** | List of known contacts for direct messaging |
| **Saved Locations** | Preview of saved map locations |
| **Recent** | Most recently heard nodes over LoRa, or found via Node Discovery |
| **Radio** | LoRa radio parameters and noise floor |
| **GPS** | GPS status and coordinates *(if GPS hardware present)* |
| **Sensors** | Battery history chart, plus attached environment sensor readings and a compass dial *(if any)* |
| **Settings** | Device settings |

---

## Home page

**OLED / TFT:**
```
┌──────────────────────────────┐
│ MyNodeName        5✉ [=]    │
│         · • · · · · ·        │
│                              │
│           14:32              │
│                              │
│         14 Apr 2026          │
│        < Connected >         │
└──────────────────────────────┘
```

**E-ink (E213 / Wireless Paper / E290):**
```
┌──────────────────────────────────────────────────────────┐
│ MyNodeName                                  5✉  [===]   │
│                · · · • · · · ·                           │
│                                                          │
│                       14:32                              │
│                                                          │
│                                                          │
│ Pin: 477370                             14 Apr 2026      │
└──────────────────────────────────────────────────────────┘
```

- Node name at top-left, battery at top-right
- **Battery icon**: the fill shows the charge.
  - A lightning bolt while charging *(V3, Wireless Paper, T096)*.
  - A USB plug while Connection Mode is USB and the cable is connected *(Heltec V4, E213, E290, T096; on T096 only once an app has opened the connection)*.
- **Envelope** (`5✉`): unread messages
- **GPS icon**: shown while GPS is on, so you don't forget it *(on OLED only on this page)*
- **Muted icon**: shown while the buzzer is muted *(ProMicro fakeTec with a buzzer)*
- Dots below the header: which page you are on
- **Large clock**: local time once the time is set; before that, time since start-up
- **OLED / TFT**: date and connection status at the bottom. In WiFi mode the node's network name replaces the date, e.g. `meshcore-d267.local`.
- **E-ink**: connection status or pairing PIN at the bottom-left, date at the bottom-right. In WiFi mode, before an app connects, the bottom row shows the IP address and the network name, e.g. `192.168.1.83` and `meshcore-d267`.
- **Long press** → **Home menu**. Single click moves down, long press runs the item, double click leaves.
  - **Sync to app**: sends all messages on the node to the connected app again (`Syncing N msgs`), e.g. after reinstalling the app or for a second app over USB or WiFi. Shows `Sync (no app)` when no app is connected.
  - **Save messages**: saves all messages now (`Saved N msgs`), so they survive a power cut or the RESET button.
  - **Reboot**: restarts the node.
  - **Shutdown**: turns the node off, the same as **Settings → Shutdown**.

Connection status:
- `< BLE Connected >` — app connected over Bluetooth
- `Pin: 123456` — waiting for Bluetooth pairing; enter this PIN in the app
- `BLE: OFF` — Bluetooth is off
- `< USB >` — USB mode
- `< WiFi Connected >` — app connected over WiFi
- `IP:192.168.1.x` — WiFi mode, waiting for the app to connect (port 5000)
- `WiFi: connecting...` — WiFi mode, not yet connected to the router

> **Finding the node on WiFi**: the IP address can change. The network name `meshcore-xxxx.local` stays the same and can be used instead, on devices that support `.local` names.

> **Time and date**: set your time zone once with `set tz.offset 7` (for UTC+7, or `set tz.offset 5:30` for UTC+5:30, see [TerminalCLI](Companion_TerminalCLI_Commands.md)); the default is UTC. The time comes from the app when it connects, or from GPS.

---

## Chats page

```
┌──────────────────────────────┐
│ CHATS          3✉ 14:32 [=]  │
│         · • · · · · ·        │
│ ▐☻▌ Alien            *2 · 2m │
│ ▐#▌ Public           *1 · 1h │
│ ▐▤▌ Dev               1 · 3d │
│                              │
└──────────────────────────────┘
```

The most recent conversations, newest first: contacts (person icon), channels (`#` icon) and room servers (speech bubble icon), each with its number of messages (`*2` = 2 unread) and the age of the newest one. A channel set to priority `none` shows `blocked`. The header shows the unread count.

Messages stay on the node after your app has synced them, and the messages you send are kept with them. The node keeps up to 256 messages (128 on the HT-CT62); when it is full, the oldest are deleted first. Messages are kept through a restart, an OTA update, **Power off** and the low-battery sleep. On ESP32 boards, a power cut or the RESET button deletes them, unless you used **Save messages** in the Home menu before.

- **Long press** → the full conversation list ([Level 0 — Group list](#level-0--group-list))

---

## Quick Send page

Send a preset message to a channel without typing. It goes to the Public channel unless you pick another in Settings → **Qk.Ch.** (see [Quick Send Channel screen](#quick-send-channel-screen)).

```
┌──────────────────────────────┐
│ QUICK SEND        14:32 [=]  │
│         · · • · · · ·        │
│ I'm OK                       │
│ On my way                    │
│ I need help                  │
│ GPS:10.7769,106.7009         │
└──────────────────────────────┘
```

- The bottom line shows your position:
  - `GPS:lat,lon` — GPS has a fix; the position is added to the message
  - `?GPS:lat,lon` — no GPS fix; the last known position is used
  - `GPS: Private` — GPS Privacy is on; no position is added
- **Long press** to choose a preset

**Choosing a preset:**

| Press | Action |
|---|---|
| Single click | Next preset |
| Double click | Back |
| Triple click | Previous preset |
| Long press | Send it → `Sent!` or `Send failed` |

After sending, the display counts the repeaters heard passing your message on (e.g. `Heard 3 Repeats`). The position is added to the text, e.g. `I'm OK @10.7769,106.7009`.

**Change the presets and the channel with [TerminalCLI](Companion_TerminalCLI_Commands.md):**
```
get quick                        list all presets
set quick.0 Arrived at camp      change preset at index 0
set quick.reset                  restore all defaults
get quick.channel                show current target channel
set quick.channel MyGroup        change target channel
```

---

## Quick Send Channel screen

Opened from **Qk.Ch.** in Settings: pick the channel Quick Send sends to.

```
┌──────────────────────────────┐
│ QK.CHANNEL 2/12              │
│──────────────────────────────│
│   Public                   * │
│ > Base Camp                  │
│   #SOS                       │
│   Valley Net                 │
└──────────────────────────────┘
```

- `*` marks the channel in use

| Press | Action |
|---|---|
| Single click | Move arrow to next channel |
| Double click | Back, nothing changed |
| Triple click | Move arrow to previous channel |
| Long press | Use the selected channel |

---

## Contacts page

```
┌──────────────────────────────┐
│ CONTACTS          14:32 [=]  │
│         · · · • · · ·        │
│ Alien                     2m │
│ Big Boy                   3d │
│                              │
│                              │
└──────────────────────────────┘
```

The contacts heard most recently, with how long ago on the right. **Long press** opens the contact list.

---

## Contact List screen

```
┌──────────────────────────────┐
│ CONTACTS  4                  │
│──────────────────────────────│
│ > ▐☻▌ Alien               2m │
│   ▐☻▌ Big Boy             3d │
│   ▐▤▌ Base Camp Room      5d │
│   ▐≋▌ Hilltop Repeater    1h │
└──────────────────────────────┘
```

Your contacts, most recently heard first, with how long ago on the right. Contacts are marked with a person icon, room servers with a speech bubble icon, repeaters with a repeater icon.

| Press | Action |
|---|---|
| Single click | Move arrow to next contact |
| Double click | Return to home |
| Triple click | Move arrow to previous contact |
| Long press | Open action menu for selected contact |

---

## Contact Action screen

Long press on a contact opens its menu:

```
┌──────────────────────────────┐
│ ▐☻▌ Alien                    │
│──────────────────────────────│
│ > Chat (3)                   │
│   Send message               │
│   Request telemetry          │
│                              │
└──────────────────────────────┘
```

| Press | Action |
|---|---|
| Single click | Move arrow to next item |
| Double click | Return to contact list |
| Triple click | Move arrow to previous item |
| Long press | Run the selected item |

- **Chat (n)** — the conversation with this contact, when there is one (see [Level 1 — Group detail](#level-1--group-detail)). Double click returns to this menu.
- **Send message** — send a direct message (Quick Reply screen)
- **Trace position** — show the contact's position on the [GPS Trace screen](#gps-trace-screen), when the contact shares one
- **Request telemetry** — ask the contact for its battery, sensors and position
- **TerminalCLI** — run commands on a repeater or room server, with the CardKB attached (see [TerminalCLI on a repeater or room server](#terminalcli-on-a-repeater-or-room-server))

A repeater's menu:

```
┌──────────────────────────────┐
│ ▐≋▌ Hilltop Repeater         │
│──────────────────────────────│
│ > Trace position             │
│   Request telemetry          │
│   TerminalCLI                │
│                              │
└──────────────────────────────┘
```

---

## Telemetry Result screen

Shows what the contact sends back: `VBAT` (its battery), its sensors (`#2`, `#3`...) and its GPS position if it has one.

On a repeater, `Logging in...` is shown before `Requesting...`.

**While waiting:**
```
┌──────────────────────────────┐
│ Alien                        │
│──────────────────────────────│
│ Requesting...                │
│                              │
│                              │
│                              │
└──────────────────────────────┘
```

**On success:**
```
┌──────────────────────────────┐
│ Alien                        │
│──────────────────────────────│
│ VBAT                   3.82V │
│ #2                    24.5°C │
│ GPS                          │
│ 10.7769  106.7009            │
└──────────────────────────────┘
```

**On timeout:**
```
┌──────────────────────────────┐
│ Alien                        │
│──────────────────────────────│
│ No response                  │
│                              │
│                              │
│                              │
└──────────────────────────────┘
```

| State | Press | Action |
|---|---|---|
| Waiting | Double click | Cancel request, return to the contact's menu |
| Result | Single click / Triple click | Scroll down / up, if the list is longer than the screen |
| Result | Long press | Open GPS Trace screen *(only when the reply has a GPS position)* |
| Result / Failed | Double click | Return to the contact's menu (single click too, if there is nothing to scroll) |

The position is shown in the format chosen in Settings → **Pos. Format**.

Other results: `Login failed` or `Login: no response` — the repeater did not let you in, or did not answer; `No response` — the contact did not answer; `Send failed` — the request could not be sent.

---

## Reading messages

Long press on the **Chats** page opens the conversation list. A message counts as read once you have seen it, or once your app has received it.

### New message alert

When new messages arrive, an alert shows which conversations they are in:

```
┌──────────────────────────────┐
│ ┌──────────────────────────┐ │
│ │    New messages from:    │ │
│ │      Alien, Public       │ │
│ │         Dev +2           │ │
│ └──────────────────────────┘ │
└──────────────────────────────┘
```

| Press | Action |
|---|---|
| Single click | Open the conversation list |
| Double click | Close the alert; the messages stay unread |

A new message in the conversation you are reading shows there instead, without an alert (see [Level 1 — Group detail](#level-1--group-detail)).

### Level 0 — Group list

Long press on the **Chats** page, or a single click on the new message alert, opens the list of all your conversations.

```
┌──────────────────────────────┐
│ CHATS          3✉     12/256 │
│──────────────────────────────│
│ > ▐☻▌ Alien          *2 · 2m │
│   ▐#▌ Public         *1 · 1h │
│   ▐▤▌ Dev             1 · 3d │
│   ▐#▌ #SOS            8 · 5d │
└──────────────────────────────┘
```

- Newest conversation first; on the right, the number of messages (`*2` = 2 unread) and the age of the newest one
- Top right: the unread count, then messages stored / the most the node keeps
- `#` icon = channel, speech bubble icon = room server, person icon = contact
- A channel set to priority `none` shows `blocked`. When it has no messages left, it moves to the end of the list, where long press lets you change its priority back.

| Press | Action |
|---|---|
| Single click | Move arrow to next conversation |
| Double click | Return to home |
| Triple click | Move arrow to previous conversation |
| Long press | Open the conversation; on a `blocked` row, open its [priority](#channel-priority) |

### Level 1 — Group detail

One conversation, like a chat, oldest message at the top. It opens on the first unread message.

```
┌──────────────────────────────┐
│ ▐#▌ Public · Bob · 3m    2/3 │
│──────────────────────────────│
│ ▐Alien▌ where are you?       │
│ ▐Bob▌ at the gate, come     ┆│
│ over                        ┆│
│ ▐Me▌ on my way               │
└──────────────────────────────┘
```

- Each message starts with the sender's name; yours start with **Me**.
- The dashed line on the right marks the selected message; the top row shows who sent it and when.
- When the newest message is on the screen, a new one scrolls into view as it arrives.
- While you read older messages, the top right shows how many newer ones are unread (`▼ 2 new`) instead of the message number.

| Press | Action |
|---|---|
| Single click | Next message (from the newest back to the oldest) |
| Double click | Return to the group list |
| Triple click | Previous message |
| Long press | Open popup menu |

---

## Popup menu (while reading a message)

Long press any message to open the popup:

```
┌──────────────────────────────┐
│ Alien · 3h               5/5 │
│──────────────────────────────│
│ I need help @10.77,106.70    │
│ ┌──────────────────────┐     │
│ │ > Reply              │     │
│ │   Save location      │     │
│ │   Delete             │     │
│ │   Delete all         │     │
│ │   Home               │     │
│ └──────────────────────┘     │
└──────────────────────────────┘
```

| Press | Action |
|---|---|
| Single click | Move to next item |
| Triple click | Move to previous item |
| Long press | Confirm selected item |
| Double click | Dismiss menu (cancel) |

- **Reply** — reply to the contact, the channel or the room server (everyone in the room sees it)
- **Save location** — save the position in the message, when it has one
- **Delete** — delete this message
- **Delete all** — delete the whole conversation
- **Home** — go to the Home page
- **Priority** — the channel's [priority](#channel-priority) (channel messages only)

### Channel priority

Opened from **Priority** in the popup, or from a `blocked` row in the conversation list. `*` marks the current priority.

```
┌──────────────────────────────┐
│ ▐#▌ Public          Priority │
│──────────────────────────────│
│ > none                       │
│   low                        │
│   normal                   * │
│   high                       │
└──────────────────────────────┘
```

| Priority | Effect |
|---|---|
| `none` | New messages from the channel are ignored: not kept, not sent to the app, no alert. Messages already on the node stay. |
| `low` | Only the last 12 messages are kept (6 on the HT-CT62). |
| `normal` | Default. When the node is full, the oldest channel messages are deleted first. |
| `high` | Never deleted to make room, like direct messages. |

Long press sets the priority, double click goes back without a change. It is the same setting as `set ch.msgs` in [TerminalCLI](Companion_TerminalCLI_Commands.md).

### Confirming a delete

Delete asks first, with **No** selected:

```
┌──────────────────────────────┐
│ Alien · 3h               5/5 │
│──────────────────────────────│
│ I need help @10.77,106.70    │
│ ┌──────────────────────┐     │
│ │ Delete 5 msgs?       │     │
│ │ > No                 │     │
│ │   Yes                │     │
│ └──────────────────────┘     │
└──────────────────────────────┘
```

Single click moves between **No** and **Yes**, long press confirms, double click cancels.

> A deleted message is gone for good, also for an app that has not synced it yet.

---

## Quick Reply screen

Reply with a preset, without typing. The presets are the same as on the Quick Send page.

```
┌──────────────────────────────┐
│ To: Alien                    │
│──────────────────────────────│
│ > I'm on my way              │
│   Meet at base               │
│   Need supplies              │
│                              │
└──────────────────────────────┘
```

The header shows who it goes to: `To: Alien` for a contact, `To: [Public]` for a channel.

| Press | Action |
|---|---|
| Single click | Next preset |
| Double click | Back without sending |
| Triple click | Previous preset |
| Long press | Send the selected preset |

An alert shows whether it was sent. A reply from a conversation then takes you back to its end, where your reply shows.

---

## CardKB keyboard

An M5Stack Unit CardKB v1.1 on the node's I2C sensor pins (SDA, SCL, 3.3 V, GND) lets you type messages. You can plug it in and out at any time; **Keyboard ready** shows when it is found.

**Navigating.** The keys work together with the button:

| Key | Same as |
|---|---|
| Down | Single click: next item in a list or menu, next message |
| Up | Triple click: previous item, previous message |
| Right / Left | Next / previous Home screen page |
| Left (deeper screens) | Double click (back) |
| Enter | Long press (select / send) |
| Esc | Double click (back) |
| Fn + 1 … 9, Fn + 0 | Jump to that Home page from any screen: 1 Home, 2 Chats, 3 Quick Send, 4 Contacts, 5 Saved Locations, 6 Recent, 7 Radio, then GPS *(when there is one)*, Sensors and Settings |

- **On a Home page**, Up or Down opens the page's list, like Enter. On Sensors they switch between the views.
- **In a list opened from a page**, Right or Left goes to the next or previous page.
- **On other screens**, Left goes back, like Esc.

While the display is off, the first key only turns it on.

**Typing a message.** With the keyboard attached, **Reply** on a message and **Send message** on a contact open the text editor instead of the Quick Reply screen. On an open Quick Reply screen, `>> Type...` at the end of the list opens the editor.

```
┌──────────────────────────────┐
│ To: Alien                    │
│──────────────────────────────│
│ On my way, ETA 10 min|       │
│                              │
│                              │
└──────────────────────────────┘
```

The header shows who it goes to; `|` is the cursor.

| Key | Action |
|---|---|
| Letters, digits, symbols, Space | Insert at the cursor |
| Del | Delete the character before the cursor |
| Left / Right | Move the cursor |
| Up / Down | Jump to the start / end |
| Enter | Send |
| Esc | Leave without sending |

A message can be up to 151 characters. Shift gives capitals and Sym gives symbols; with the [low-power CardKB firmware](https://github.com/dt267/MeshCore-Low-Power-Firmware/releases/download/MeshCore-low-power-v1.17.dev_1011/M5Stack_CardKB_v1.1_low_power.hex), tap Shift or Sym once to lock it and again to unlock. A direct message shows **Sending...**, then **Delivered!** or **Not delivered**. A channel message shows **Sent!**.

### TerminalCLI on the node

While the CardKB is attached, the Home menu (long press on the Home page) ends with **TerminalCLI**. It runs the node's own [TerminalCLI commands](Companion_TerminalCLI_Commands.md).

```
┌──────────────────────────────┐
│ TerminalCLI                  │
│──────────────────────────────│
│ $ get tx                     │
│ 22                           │
│                              │
└──────────────────────────────┘
```

| Key | Action |
|---|---|
| Letters, digits, symbols, Space | Type the command after the `$` prompt |
| Enter | Run it; the reply appears under the command |
| Up / Down (on a reply) | Scroll a long reply |
| Enter (on a reply) | Next page, while `[Enter] more` is shown |
| Any other key (on a reply) | Back to the prompt; typing starts the next command |
| Up (empty prompt) | Bring back the last command |
| Esc (at the prompt) | Back to the Home page |

### TerminalCLI on a repeater or room server

**TerminalCLI** in the menu of a repeater or room server ([Contact Action screen](#contact-action-screen)) runs commands on it. First enter its admin password:

```
┌──────────────────────────────┐
│ ▐≋▌ CLI: Hilltop Repeater    │
│──────────────────────────────│
│ Password: ********|          │
│                              │
└──────────────────────────────┘
```

Press Enter to log in. With the admin password, the `$` prompt appears. Otherwise you see one of these and can try again:

- `Logged in as Guest`, `Read-only` or `Read-write`: logged in, but without admin rights
- `Login refused`: the server refused you
- `No reply`: no answer (also after a wrong password)

Type a command and press Enter: the reply appears under it, or `(no reply)`. Up / Down scroll a long reply; Up on an empty prompt brings back the last command. Esc goes back to the server's menu. The replies are shown only on the node, not in the app. See [Repeater CLI Commands](Repeater_CLI_Commands.md).

---

## Saving a GPS location

On a message with a GPS position, long press and choose **Save location**:

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
| Triple click | Move to previous slot |
| Long press | Save to selected slot |
| Double click | Cancel |

> **Tip for SOS situations**: save the location right away. It stays on the **Saved Locs** page even after the message is gone.

---

## Saved locations

The **SAVED LOCS** page shows your saved GPS points.

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

Long press the **SAVED LOCS** page to open the full list:

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

Each entry shows the sender and the start of the message.

| Press | Action |
|---|---|
| Single click | Move highlight to next entry |
| Triple click | Move highlight to previous entry |
| Long press | Open GPS trace screen for highlighted entry |
| Double click | Return to home |

---

## GPS Trace screen

Shows live distance and bearing from your current position to a saved location, a contact's advertised position (**Trace position**) or the GPS position in a telemetry reply (**Trace GPS**).

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

- **Top line**: the target's name and the time
- **Coordinates**: the target's position (format: Settings → **Pos. Format**)
- **Distance**: in a straight line, needs your own GPS fix
- **Bearing**: the direction to go

With a compass attached, an arrow on the left points to the target; distance and bearing move to the right:

```
┌──────────────────────────────┐
│ Alien: I need help     14:32 │
│──────────────────────────────│
│      10.7769  106.7009       │
│    ╭─▾─╮                     │
│   │  ↗  │        1.2km       │
│    ╰───╯      247°  WSW      │
└──────────────────────────────┘
```

If your node has no GPS fix, only the coordinates are shown.

| Press | Action |
|---|---|
| Any button | Go back |

---

## Recent page

Shows the most recently heard nodes — real adverts, and repeaters found by **Node Discovery**.

```
┌──────────────────────────────┐
│ RECENT            14:32 [==] │
│         · · · · • · ·        │
│ Alien                    42s │
│ Base Camp                 5m │
│ Repeater-1          3.5>-2.0 │
│ a1b2c3d4            1.0>-5.5 │
└──────────────────────────────┘
```

Each row shows how long ago the node was heard. A repeater found by Node Discovery shows the signal strength (SNR) both ways instead: how it heard you, then how you heard it. One that is not in your contacts shows a code instead of a name.

**Long press** opens a menu:

- **Send Zero Hop Advert** — announce yourself to the nodes in direct range (`Advert sent!`). Useful when two people standing together want to find each other at once.
- **Node Discovery** — look for repeaters in range (`Discovering...`); they appear on this page. They are not added to your contacts.

| Press | Action |
|---|---|
| Single click | Next item |
| Double click | Back |
| Triple click | Previous item |
| Long press | Run the item |

---

## Radio page

LoRa radio settings. Long press to change **RxGain** and **RxDuty**.

```
┌──────────────────────────────┐
│ RADIO             14:32 [==] │
│         · · · · · • ·        │
│ FQ: 915.000          SF: 10  │
│ BW: 250.00            CR: 5  │
│ TX: 20dBm          RxG: OFF  │
│ NF: -95            RxD: OFF  │
└──────────────────────────────┘
```

| Field | Meaning |
|---|---|
| FQ | Frequency in MHz |
| SF | Spreading factor |
| BW | Bandwidth in kHz |
| CR | Coding rate |
| TX | Transmit power in dBm |
| RxG | RX gain: `OFF` / `ON` (Heltec V4.3 / T096: `OFF` / `INT` / `EXT`) |
| NF | Noise floor: background noise in dBm |
| RxD | RX duty cycle: `OFF` / `ON` |

```
┌──────────────────────────────┐
│ RADIO             14:32 [==] │
│         · · · · · • ·        │
│ RxGain                    ON │
│ RxDuty                   OFF │
│ RSSI: -87          SNR: 7.5  │
│ RX: 1234             TX: 567 │
└──────────────────────────────┘
```

The bottom rows show the signal of the last packet received (RSSI, SNR) and the packets received / sent since start-up, so you can see the effect of a change.

| Press | Action |
|---|---|
| Single click | Next item |
| Double click | Back |
| Triple click | Previous item |
| Long press | Change it |

| Item | Action |
|---|---|
| **RxGain** | RX gain: OFF → ON (Heltec V4.3 / T096: OFF → INT → EXT) |
| **RxDuty** | RX duty cycle on/off: lets the receiver sleep, saving 2-3 mA. Works only at SF5-SF8. |

Both are also in [Settings](#settings-page).

---

## GPS page *(optional)*

Only shown when a GPS is found. On E213, V3 and E290, connect the GPS before switching the node on.

**GPS on:**
```
┌──────────────────────────────┐
│ GPS               14:32 [==] │
│         · · · · · · •        │
│ gps on                   fix │
│ sat / hdop           8 / 1.2 │
│ pos         10.7769 106.7009 │
│ alt                      12m │
└──────────────────────────────┘
```

**GPS off:**
```
┌──────────────────────────────┐
│ GPS               14:32 [==] │
│         · · · · · · •        │
│ gps off                      │
│ intv                  always │
│ mode             GPS+BDS+GLO │
│ AT6558R-5N-32-1C580901       │
└──────────────────────────────┘
```
*(mode line: Heltec V4 / T096 / E213 / V3 / E290 only)*

- **Long press** — turn GPS on or off
- `fix` / `no fix` — whether GPS has a position
- `sat / hdop` — satellites in use and accuracy (lower HDOP is better); `gps.minsat` / `gps.hdop` in [TerminalCLI Commands](Companion_TerminalCLI_Commands.md) set what counts as a fix
- `intv` — how often GPS updates: `always`, or every so many seconds
- `mode` — which satellite systems are used (see `gps.mode` in [TerminalCLI Commands](Companion_TerminalCLI_Commands.md))
- Bottom line (GPS off) — the GPS chip model
- `pos` — your position (format: Settings → **Pos. Format**)
- `alt` — altitude

---

## Sensors page

**Battery** first, then one view per sensor; long press switches view. With sensors attached, a scrollbar on the right shows which view you are on.

**Battery — 14-day history:**
```
┌──────────────────────────────┐
│ BATTERY 14D            3.82V │
│         · · · · · · •        │
│ ▂▃▄▅▆▇█▇▆▅▄▃▂▁       │
│ Uptime 3d 04h                │
└──────────────────────────────┘
```

- Battery level over the last 14 days, one bar per day
- Dashed lines: 100% and 50%
- Top right: battery voltage
- Bottom: time since start-up

**Attached sensor readings** *(only shown if an environment sensor is detected)*:
```
┌──────────────────────────────┐
│ SENSOR                BME280 │
│         · · · · · · •        │
│ 24.3°C          61%          │
│ 1013hPa         42m          │
└──────────────────────────────┘
```

- Top right: the sensor's name (e.g. `BME280`)
- Up to 6 readings, each with its unit (`°C`, `%`, `hPa`...)
- With several sensors, the views are numbered `SENSOR 1`, `SENSOR 2`...

**Compass** *(only shown if an HMC5883L or QMC5883L is detected)*:
```
┌──────────────────────────────┐
│ SENSOR              HMC5883L │
│         · · · · · · •        │
│      ▾                       │
│   W  ↑  E          123°      │
│      S              SE       │
└──────────────────────────────┘
```

- The notch at the top marks where the device points; the needle points to magnetic north
- Right: the heading in degrees and its direction (`N`, `NE`, `E`...)
- Hold the device level
- `No data` when there is no reading, e.g. with a magnet nearby
- Calibration: Settings → [Compass Calib.](#compass-calibration)

At startup, a **Detected:** alert lists the GPS and the sensors found.

| Press | Action |
|---|---|
| Long press | Next view |

---

## Settings page

```
┌──────────────────────────────┐
│ SETTINGS          14:32 [==] │
│         · · · · · · •        │
| Connection Mode          BLE |
│ BLE                       ON │
│ Repeat                   OFF │
│ RxGain                    ON │
└──────────────────────────────┘
```

- **Long press** to change settings

| Press | Action |
|---|---|
| Single click | Next item |
| Double click | Back |
| Triple click | Previous item |
| Long press | Change it |

| Item | Action |
|---|---|
| **BLE** | Bluetooth on/off; `ON, Connected` while an app is connected. *Not shown in USB or WiFi mode.* |
| **Connection Mode** | Choose `BLE`, `USB` or `WiFi`; the node restarts in that mode. `*` marks the current one. |
| **Repeat** | Pass on other nodes' packets, on/off |
| **RxGain** | RX gain: OFF → ON (Heltec V4.3 / T096: OFF → INT → EXT) |
| **RxDuty** | RX duty cycle on/off: lets the receiver sleep, saving 2-3 mA. Works only at SF5-SF8. |
| **Brightness** | Backlight: `25` → `50` → `75` → `100` *(Heltec T096)* |
| **Theme** | `Dark` / `Light`. Dark is the default. *(Heltec T096)* |
| **Buzzer** | `ON` / `MUTED`; it beeps when turned back on *(ProMicro fakeTec with a buzzer)* |
| **Units** | Metric / Imperial |
| **Pos. Format** | How positions are shown: `DD` → `UTM` → `MGRS` |
| **GPS Privacy** | ON: Quick Send messages go without your position |
| **Compass Calib.** | Calibrate the [compass](#compass-calibration) *(when there is one)* |
| **Qk.Ch.** | The channel Quick Send uses; opens the [channel list](#quick-send-channel-screen) |
| **Screen Off** | Screen timeout: `15s` → `3min` → `Never` |
| **Flip Screen** | Turn the display upside down |
| **Font Weight** | `Thin` / `Bold` *(e-ink only)* |
| **Message Font** | Message text size: `Normal` / `Large` |
| **Start OTA** | Update the firmware. On ESP32: connect to the WiFi network `MeshCore-OTA` and open `192.168.4.1`; the page also lets you set the WiFi network for WiFi mode. On nRF52 (T096 / RAK4631): use the **nRF Device Firmware Update** app with the `.zip` file. |
| **Shutdown** | Turn the node off |
| **About** | Node name, firmware version and build date |

---

## Compass calibration

Opened from Settings → **Compass Calib.**

Hold the device level and turn it slowly through a full circle. The dial is split into 12 slices, which fill in as you turn through them; the right side shows how many are done and the seconds left.

- When all 12 are done, **Compass calibrated** is shown and you are back in Settings.
- After 30 seconds without all 12, **Calibration failed** is shown.
- Double click cancels.

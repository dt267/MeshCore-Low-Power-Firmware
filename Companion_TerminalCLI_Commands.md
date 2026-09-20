# MeshCore Companion Radio TerminalCLI Commands

## Command Line Interface for Companion.
Setup: In the MeshCore app, create a channel named "TerminalCLI". It will now act as a Terminal CLI for Companion; everything typed here is a command.

<img height="600" alt="Screenshot 2026-03-19 at 9 38 13 PM" src="https://github.com/user-attachments/assets/c1df229f-5eed-43b8-abdb-906c3c864a62" />


## Command Reference

| Command | Parameters | Notes |
  |---|---|---|
  | `help` | — | List every command this board supports, grouped into sections. Only works over the app link; a remote CLI request replies `help is local-only`. |
  | `stats` | — | Show battery voltage (mV), uptime (s), noise floor, last RSSI/SNR, and RX/TX/error packet counts |
  | `reboot` | — | Reboot the device |
  | `poweroff` | — | Power off the device |
  | `ver` | — | Show firmware version and build date |
  | `board` | — | Show the board/manufacturer name this firmware was built for |
  | `get name` | — | Show the node name |
  | `set name <text>` | `text`: max 31 chars | Set the node name. The characters `[` `]` `\` `:` `,` `?` `*` are rejected. Saved to flash. |
  | `set pin <n>` | `n`: 6-digit BLE pairing pin, or `0` for the built-in default | Set the BLE pairing pin. Saved to flash; takes effect on the next reboot. `0` restores the compiled-in default, which on a board with a display is a new random pin each session. |
  | `clock` | — | Show the current UTC date and time |
  | `clock sync` | — | Set the clock from the timestamp of the command itself, i.e. the clock of the phone that sent it. Refused if that clock is older than the firmware build date. |
  | `gps` | — | Show GPS status: enabled/disabled, fix status, satellite count |
  | `gps on` | — | Enable GPS |
  | `gps off` | — | Disable GPS |
  | `get gps.interval` | — | Show GPS update interval (`always on` if `0`) |
  | `set gps.interval <s>` | `s`: seconds `1..86400`, or `0` = GPS always on (no sleep) | Set how often the GPS wakes up to update location. Applied immediately and saved. Default: `10`. |
  | `get gps.minsat` | — | Show how many satellites GPS needs before it reports a valid fix |
  | `set gps.minsat <n>` | `n`: `4..24` | Set how many satellites GPS needs before reporting a valid fix. Higher = more reliable, but may take longer to get one. Applied immediately and saved. Default: `6`. |
  | `get gps.hdop` | — | Show the accuracy threshold (HDOP ×10 — lower number means stricter/more accurate) GPS requires before reporting a valid fix |
  | `set gps.hdop <n>` | `n`: `5..250` (HDOP×10, e.g. `20` = HDOP 2.0) | Set how accurate a position must be before GPS reports a valid fix. Lower = stricter/more accurate but may take longer; higher = looser/faster but less precise. Applied immediately and saved. Default: `20` (HDOP 2.0). |
  | `get gps.mode` | — | Show current GNSS constellation selection. *(Heltec V4 / T096 / E213 / V3 / E290 only)* |
  | `set gps.mode <n>` | **Heltec V4:** `1`=GPS `2`=GPS+BDS `3`=GPS+GLO `4`=GPS+BDS+GLO (default `4`) · **T096:** `1`=GPS-L1 `2`=All-sys-L1 `3`=All-sys+QZSS-dual (default `3`) · **E213 / V3 / E290:** `1`=GPS `2`=GPS+BDS `3`=GPS+BDS+GLO+GAL `4`=GPS+BDS+GLO+GAL+QZSS (default `4`; only takes effect on a confirmed ATGM336H-6N module, tested via M5Stack's Unit GPS v1.1) | Select GNSS constellation preset. Saved to flash; takes effect on next GPS on. |
  | `get radio` | — | Show current radio parameters as `freq,bw,sf,cr` — frequency (MHz), bandwidth (kHz), spreading factor, coding rate |
  | `set radio <freq>,<bw>,<sf>,<cr>` | `freq`: MHz `150..2500`; `bw`: kHz, exact chip step only — **SX1262:** `7.8` `10.4` `15.6` `20.8` `31.25` `41.7` `62.5` `125` `250` `500` · **LR1121:** `62.5` `125` `250` `500`, plus `203.125` `406.25` `812.5` above 1 GHz; `sf`: `5..12`; `cr`: `5..8` | Set all four at once, **comma-separated** (no spaces). Saved and applied without a reboot. Example: `set radio 869.525,250,10,5` |
  | `get radio.rxgain` | — | Show the RX gain state, worded like the `set` reply, e.g. `> RX gain on` or `> External FEM LNA on` |
  | `set radio.rxgain <mode>` | `off` \| `on` · Heltec V4.3 / T096: `off` \| `int` \| `ext` | `off` = SX1262 Rx power saving gain (lowest RX current). `on` = SX1262 Rx boosted gain. Heltec V4.3 / T096: `off` also bypasses the KCT8103L LNA, `int` = SX1262 Rx boosted gain only, `ext` = KCT8103L LNA only; `on` is another spelling of `int`. Saved and applied immediately. Default: `off` on Heltec V4.3 / T096, `on` elsewhere. |
  | `get rx.duty` | — | Show whether RX duty cycle is on, and the listening windows in use |
  | `set rx.duty <on\|off>` | `on` \| `off` | Sleep the receiver between short listening windows to cut idle current by 2-3 mA. `on` uses the windows computed for the spreading factor in use; if none fit the current SF/BW the request is refused and duty cycling stays off. Saved; applied immediately. Default: `off`. |
  | `get agc.resets` | — | Show how many times the AGC has been auto-reset since boot or last `clear agc.resets`. Returns `n/a (not supported on LR1121)` on LR1121 boards. |
  | `clear agc.resets` | — | Reset the AGC auto-reset counter to zero. No-op on LR1121 boards (replies `not applicable on LR1121`). |
  | `get tx` | — | Show current transmit power (dBm) |
  | `set tx <dBm>` | `dBm`: integer | Set transmit power in dBm. Saved and applied immediately. |  
  | `start ota` | — | Start OTA firmware update. On ESP32: connect to Wi-Fi `MeshCore-OTA`, then go to `192.168.4.1` (opens the config portal). On nRF52: enters BLE DFU mode. |
  | `get repeat` | — | Show whether repeat mode is `on` or `off`, and the current frequency |
  | `set repeat on` | — | Enable repeat mode |
  | `set repeat off` | — | Disable repeat mode |
  | `get msg.persist` | — | Show whether unsynced messages are saved to flash: `on` or `off` (default `off`) |
  | `set msg.persist <mode>` | `on` \| `off` | Persist unsynced messages to flash so they survive a crash, reboot, or dead battery before the app syncs them. Turning off immediately clears any already-persisted copy from flash. |
  | `get repeat.freqs` | — | List all frequencies (MHz) allowed for repeat mode |
  | `add repeat.freq <MHz>` | `MHz`: e.g. `915` or `915.125` | Add a frequency to the repeat allowed list (max 5). Saved after reboot. |
  | `del repeat.freq <MHz>` | `MHz`: frequency to remove | Remove a frequency from the repeat allowed list |
  | `get adc.multiplier` | — | Show the battery voltage calibration multiplier |
  | `set adc.multiplier <value>` | `value`: decimal, e.g. `2.000` | Set battery voltage calibration multiplier. Use `0` to reset to default. |
  | `get bat.cfg` | — | Show the battery thresholds in effect as `<cutoff>,<max>` in volts. |
  | `set bat.cfg <cutoff>,<max>` | two values in volts (decimal point, e.g. `3.4`), **comma-separated**; `0` in either slot = board default | `cutoff`: the node deep-sleeps below it (default `3.4` on most boards, keeps ~10 % of the charge in reserve so deep sleep can keep checking the battery; settable down to `2.9` for headroom on other battery chemistries) and wakes on its own once the battery has recovered above it; `max`: the pack's full-charge voltage, up to `5.5`. Batteries other than Li-ion / LiPo are used at your own risk. Example for 2S LTO: `set bat.cfg 4.4,5.4`. |
  | `get txdelay` | — | Show flood relay jitter window scale factor (default `0.50`) |
  | `set txdelay <value>` | `value`: decimal `0..10`, e.g. `2.0` | Set flood relay jitter window scale. Higher = wider window = fewer collisions but higher latency. |
  | `get direct.txdelay` | — | Show direct relay jitter window scale factor (default `0.20`) |
  | `set direct.txdelay <value>` | `value`: decimal `0..10` | Set direct relay jitter window scale. |
  | `get int.thresh` | — | Show RSSI interference threshold in dB above noise floor (`0` = disabled) |
  | `set int.thresh <dB>` | `dB`: integer `0..200` | Set RSSI interference threshold. `0` disables the check. |
  | `get tz.offset` | — | Show UTC offset in hours used for display (`0` = UTC) |
  | `set tz.offset <hours>` | `hours`: integer `-12..14` | Set local timezone offset. Example: `set tz.offset 7` for UTC+7. Applied to the clock and date on the display only — all internal timestamps remain UTC. |
  | `get quick` | — | List all Quick Send presets with their index numbers |
  | `set quick.<N> <text>` | `N`: 0–9, `text`: message | Set preset at index N. Example: `set quick.2 Meet me at the park` |
  | `set quick.reset` | — | Restore all 10 presets to built-in defaults |
  | `get quick.channel` | — | Show the channel Quick Send currently targets |
  | `set quick.channel <channel>` | `channel`: channel name (spaces allowed) | Set the channel Quick Send sends to. Must be an existing, named channel. |
  | `get loc` | — | List occupied saved location slots, one per line: `N:lat,lon:name` (N is 0-based) |
  | `set loc.<N> <name> <lat> <lon>` | `N`: 0–9; name may contain spaces; lat/lon come last | Save a GPS location to slot N (0-based, display shows 1–10). Example: `set loc.0 Base camp 10.7769 106.7009` |
  | `del loc.<N>` | `N`: 0–9 | Clear saved location at slot N (0-based) |
  | `del loc.all` | — | Clear all saved location slots |
  | `set ch.hops <channel> <N>` | `channel`: channel name; `N`: 1..max | Limit outgoing messages on the channel to at most N hops. Channel names with spaces are supported. |
  | `set ch.hops <channel> off` | `channel`: channel name | Remove the hop limit for the channel |
  | `get ch.hops <channel>` | `channel`: channel name (spaces allowed) | Show current hop limit for the channel |
  | `ch.hops status` | — | List all channels that have an active hop limit |
  | `ch.hops clear` | — | Remove all channel hop limits |
  | `set ch.msgs <channel> <tier>` | `channel`: channel name (spaces allowed); `tier`: `none` \| `low` \| `normal` \| `high` | Set how many unsynced offline messages the channel is allowed to keep. `none` = completely blocked — never delivered to the app, never shown on-screen, no notification of any kind. `low` = capped at 12 unsynced offline messages, oldest trimmed first once over the cap — except the message currently on screen, which is never trimmed. `normal` (default) = unlimited, unchanged behavior. `high` = unlimited and never dropped to make room — same protected standing as a direct message. |
  | `get ch.msgs <channel>` | `channel`: channel name (spaces allowed) | Show current message priority tier for the channel |
  | `ch.msgs status` | — | List all channels not set to `normal` (the default) |
  | `ch.msgs clear` | — | Reset all channels to `normal` |
  | `get conn.mode` | — | Show current connection transport: `ble`, `usb`, or `wifi` |
  | `set conn.mode <mode>` | `ble` \| `usb` \| `wifi` | Switch transport mode. Saves to flash and reboots immediately. In USB mode the node connects via USB serial; BLE is not started. In WiFi mode the node connects as a STA to the configured network and listens for TCP connections on port 5000. |
  | `get wifi.ssid` | — | Show the WiFi SSID configured for WiFi mode |
  | `set wifi.ssid <ssid>` | `ssid`: network name (max 32 chars) | Set the WiFi SSID for WiFi mode. Saved to flash. |
  | `get wifi.password` | — | Returns `***` (password is write-only) |
  | `set wifi.password <password>` | `password`: max 64 chars | Set the WiFi password for WiFi mode. Saved to flash. Leave blank or omit for open networks. |
  | `get wifi.ip` | — | Show static IP and subnet configured for WiFi mode, or `DHCP` if none is set |
  | `set wifi.ip <ip> [subnet]` | `ip`: e.g. `192.168.1.100`; `subnet`: e.g. `255.255.255.0` (optional, default `255.255.255.0`) | Set a static IP for WiFi mode. Saved to flash; takes effect after next reboot into WiFi mode. If subnet is omitted, the previously saved subnet is kept. Example: `set wifi.ip 192.168.1.100` or `set wifi.ip 192.168.1.100 255.255.255.0` |
  | `clear wifi.ip` | — | Remove static IP; the node will use DHCP on next boot in WiFi mode. |
  | `set portal.password <password>` | `password`: max 32 chars | Set the config portal login password. When set, the browser must log in before accessing the portal. Saved to flash. |
  | `clear portal.password` | — | Remove the portal password; the portal becomes accessible without login. |

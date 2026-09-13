# MeshCore Repeater / Room Server / Sensor CLI Commands

Custom CLI commands added in this firmware build, beyond the upstream MeshCore defaults.
For the full upstream command reference see [docs/cli_commands.md](https://github.com/meshcore-dev/MeshCore/blob/main/docs/cli_commands.md).

## Command Reference

| Command | Parameters | Notes |
|---|---|---|
| `get advert.hops.max` | — | Show max hops for relaying advertisement (ADVERT) packets. Default: `8`. Alias: `flood.max.advert` |
| `set advert.hops.max <N>` | `N`: `0..flood.max` | Limit how far ADVERT packets are relayed. `0` = suppress all advert relay. Clamped to `flood.max`. Repeater and room server. |
| `get group.hops.max` | — | Show max hops for relaying group messages (GRP_TXT / GRP_DATA) |
| `set group.hops.max <N>` | `N`: `0..flood.max` | Limit how far group messages are relayed. `0` = suppress all group relay. Clamped to `flood.max`. **Repeater only.** |
| `get radio` | — | Show current radio parameters as `freq,bw,sf,cr` — frequency (MHz), bandwidth (kHz), spreading factor, coding rate |
| `set radio <freq>,<bw>,<sf>,<cr>` | `freq`: MHz `150..2500`; `bw`: kHz, exact chip step only — **SX1262:** `7.8` `10.4` `15.6` `20.8` `31.25` `41.7` `62.5` `125` `250` `500` · **LR1121:** `62.5` `125` `250` `500`, plus `203.125` `406.25` `812.5` above 1 GHz; `sf`: `5..12`; `cr`: `5..8` | Set all four at once, **comma-separated** (no spaces). Saved and applied without a reboot. Example: `set radio 869.525,250,10,5` |
| `get radio.rxgain` | — | Show the RX gain state, worded like the `set` reply, e.g. `> RX gain on` or `> External FEM LNA on` |
| `set radio.rxgain <mode>` | `off` \| `on` · Heltec V4.3 / T096: `off` \| `int` \| `ext` | `off` = SX1262 Rx power saving gain (lowest RX current). `on` = SX1262 Rx boosted gain. Heltec V4.3 / T096: `off` also bypasses the KCT8103L LNA, `int` = SX1262 Rx boosted gain only, `ext` = KCT8103L LNA only; `on` is another spelling of `int`. Saved and applied immediately. Default: `off` on Heltec V4.3 / T096, `on` elsewhere. |
| `tempradio <freq>,<bw>,<sf>,<cr>,<mins>` | same as `set radio`, plus `mins`: minutes `>0` | Apply radio params temporarily, then revert automatically after `mins`. Not saved to flash. Example: `tempradio 869.525,250,10,5,30` |
| `get rx.duty` | — | Show whether RX duty cycle is on, and the listening windows in use |
| `set rx.duty <on\|off>` | `on` \| `off` | Sleep the receiver between short listening windows to cut idle current by 2-3 mA. `on` uses the windows computed for the spreading factor in use; if none fit the current SF/BW the request is refused and duty cycling stays off. Saved; applied immediately. Default: `off`. |
| `get agc.resets` | — | Show how many times the AGC has been auto-reset since boot or last `clear agc.resets`. Returns `n/a (not supported on LR1121)` on LR1121 boards. |
| `clear agc.resets` | — | Reset the AGC auto-reset counter to zero. No-op on LR1121 boards (replies `not applicable on LR1121`). |
| `get gps.interval` | — | Show GPS update interval. Returns `always on` if `0`. |
| `set gps.interval <s>` | `s`: seconds `1..86400`, or `0` = GPS always on (no sleep) | Set how often the GPS wakes up to update location. Applied immediately and saved. Default: `10`. |
| `get gps.minsat` | — | Show how many satellites GPS needs before it reports a valid fix |
| `set gps.minsat <n>` | `n`: `4..24` | Set how many satellites GPS needs before reporting a valid fix. Higher = more reliable, but may take longer to get one. Applied immediately and saved. Default: `6`. |
| `get gps.hdop` | — | Show the accuracy threshold (HDOP ×10 — lower number means stricter/more accurate) GPS requires before reporting a valid fix |
| `set gps.hdop <n>` | `n`: `5..250` (HDOP×10, e.g. `20` = HDOP 2.0) | Set how accurate a position must be before GPS reports a valid fix. Lower = stricter/more accurate but may take longer; higher = looser/faster but less precise. Applied immediately and saved. Default: `20` (HDOP 2.0). |
| `get gps.mode` | — | Show current GNSS constellation selection. *(Heltec V4 / T096 / E213 / V3 / E290 only)* |
| `set gps.mode <n>` | **Heltec V4:** `1`=GPS `2`=GPS+BDS `3`=GPS+GLO `4`=GPS+BDS+GLO (default `4`) · **T096:** `1`=GPS-L1 `2`=All-sys-L1 `3`=All-sys+QZSS-dual (default `3`) · **E213 / V3 / E290:** `1`=GPS `2`=GPS+BDS `3`=GPS+BDS+GLO+GAL `4`=GPS+BDS+GLO+GAL+QZSS (default `4`; only takes effect on a confirmed ATGM336H-6N module, tested via M5Stack's Unit GPS v1.1) | Select GNSS constellation preset. Saved to flash; takes effect on next GPS on. |
| `get bridge.type` | — | Show which bridge type this build was compiled with. Returns `espnow` on the ESP32 builds in this release. |
| `get bridge.enabled` | — | Show whether the bridge is currently enabled |
| `set bridge.enabled <on\|off>` | `on` \| `off` | Enable/disable the bridge. Takes effect immediately — no reboot. |
| `get bridge.peer` | — | Show this repeater's own address and the one it paired with. The peer shown here must be the self shown on the other repeater — otherwise they are not talking to each other. |
| `get bridge.channel` | — | Show the WiFi channel the bridge uses |
| `set bridge.channel <n>` | `n`: `1..14` | Set the WiFi channel. Must match on both sides of the bridge. Default: `1`. |
| `get bridge.secret` | — | Show the shared secret used to obfuscate bridge traffic |
| `set bridge.secret <text>` | `text` | Set the shared secret used to obfuscate bridge traffic. Must match on both sides of the bridge. |
| `get bridge.tx` | — | Show the ESP-NOW bridge 2.4 GHz TX power in dBm, with the accepted range |
| `set bridge.tx <dbm>` | `dbm`: `2..20` | Set the ESP-NOW bridge TX power. Rounded to the nearest hardware step (`2, 5, 7, 8, 11, 13, 14, 15, 16, 18, 20` dBm); `0` or less = minimum. Applied immediately — no reboot. Also settable on the config portal. Default: `20`. |
| `get bridge.delay` | — | Show the delay applied to packets arriving over the bridge |
| `set bridge.delay <ms>` | `ms`: `0..10000` | Delay before a bridged packet is handed to the mesh, guarding against collision with the original LoRa packet. Set `0` when the two segments cannot hear each other, which is the usual case. Default: `500`. |
| `get bridge.source` | — | Show which packets get bridged: `logTx` (transmitted) or `logRx` (received) |
| `set bridge.source <tx\|rx>` | `tx` \| `rx` | Select which packets get bridged. Default: `tx`. |

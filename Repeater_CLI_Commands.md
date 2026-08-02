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
| `reg read <addr>` | `addr`: hex register address | Read 1 byte from a radio register. Example: `reg read 8AC` |
| `reg write <addr> <val>` | `addr`, `val`: hex | Write 1 or more bytes to a radio register. Values revert after reboot. Example: `reg write 0740 1424` |
| `get agc.resets` | — | Show how many times the AGC has been auto-reset since boot or last `clear agc.resets` |
| `clear agc.resets` | — | Reset the AGC auto-reset counter to zero |
| `get gps.interval` | — | Show GPS update interval. Returns `always on` if `0`. |
| `set gps.interval <s>` | `s`: seconds `1..86400`, or `0` = GPS always on (no sleep) | Set how often the GPS wakes up to update location. Applied immediately and saved. Default: `10`. |
| `get gps.minsat` | — | Show how many satellites GPS needs before it reports a valid fix |
| `set gps.minsat <n>` | `n`: `4..24` | Set how many satellites GPS needs before reporting a valid fix. Higher = more reliable, but may take longer to get one. Applied immediately and saved. Default: `6`. |
| `get gps.hdop` | — | Show the accuracy threshold (HDOP ×10 — lower number means stricter/more accurate) GPS requires before reporting a valid fix |
| `set gps.hdop <n>` | `n`: `5..250` (HDOP×10, e.g. `20` = HDOP 2.0) | Set how accurate a position must be before GPS reports a valid fix. Lower = stricter/more accurate but may take longer; higher = looser/faster but less precise. Applied immediately and saved. Default: `20` (HDOP 2.0). |
| `get gps.mode` | — | Show current GNSS constellation selection. *(Heltec V4 / T096 / E213 / V3 / E290 only)* |
| `set gps.mode <n>` | **Heltec V4:** `1`=GPS `2`=GPS+BDS `3`=GPS+GLO `4`=GPS+BDS+GLO (default `4`) · **T096:** `1`=GPS-L1 `2`=All-sys-L1 `3`=All-sys+QZSS-dual (default `3`) · **E213 / V3 / E290:** `1`=GPS `2`=GPS+BDS `3`=GPS+BDS+GLO+GAL `4`=GPS+BDS+GLO+GAL+QZSS (default `4`; only takes effect on a confirmed ATGM336H-6N module, tested via M5Stack's Unit GPS v1.1) | Select GNSS constellation preset. Saved to flash; takes effect on next GPS on. |

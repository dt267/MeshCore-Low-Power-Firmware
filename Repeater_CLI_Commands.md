# MeshCore Repeater / Room Server / Sensor CLI Commands

Custom CLI commands added in this firmware build, beyond the upstream MeshCore defaults.
For the full upstream command reference see [docs/cli_commands.md](https://github.com/meshcore-dev/MeshCore/blob/main/docs/cli_commands.md).

## Command Reference

| Command | Parameters | Notes |
|---|---|---|
| `get advert.hops.max` | — | Show max hops for relaying advertisement (ADVERT) packets |
| `set advert.hops.max <N>` | `N`: `0..flood.max` | Limit how far ADVERT packets are relayed. `0` = suppress all advert relay. Clamped to `flood.max`. **Repeater only.** |
| `get group.hops.max` | — | Show max hops for relaying group messages (GRP_TXT / GRP_DATA) |
| `set group.hops.max <N>` | `N`: `0..flood.max` | Limit how far group messages are relayed. `0` = suppress all group relay. Clamped to `flood.max`. **Repeater only.** |
| `reg read <addr>` | `addr`: hex register address | Read 1 byte from a radio register. Example: `reg read 8AC` |
| `reg write <addr> <val>` | `addr`, `val`: hex | Write 1 or more bytes to a radio register. Values revert after reboot. Example: `reg write 0740 1424` |
| `get agc.resets` | — | Show how many times the AGC has been auto-reset since boot or last `clear agc.resets` |
| `clear agc.resets` | — | Reset the AGC auto-reset counter to zero |
| `get gps.interval` | — | Show GPS update interval. Returns `always on` if `0`. |
| `set gps.interval <s>` | `s`: seconds `1..86400`, or `0` = GPS always on (no sleep) | Set how often the GPS wakes up to update location. Applied immediately and saved. Default: `10`. |
| `get gps.mode` | — | Show current GNSS constellation selection. *(Heltec V4 / T096 only)* |
| `set gps.mode <n>` | **Heltec V4:** `1`=GPS `2`=GPS+BDS `3`=GPS+GLO `4`=GPS+BDS+GLO (default `4`) · **T096:** `1`=GPS-L1 `2`=All-sys-L1 `3`=All-sys+QZSS-dual (default `3`) | Select GNSS constellation preset. Saved to flash; takes effect on next GPS on. |

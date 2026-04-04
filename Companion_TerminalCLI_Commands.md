# MeshCore Companion Radio TerminalCLI Commands

Commands available via the TerminalCLI on companion_radio firmware.

---

## Command Reference

| Command | Parameters | Notes |
  |---|---|---|
  | `stats` | — | Show battery voltage (mV), uptime (s), noise floor, last RSSI/SNR, and RX/TX/error packet counts |
  | `reboot` | — | Reboot the device |
  | `poweroff` | — | Power off the device |
  | `gps` | — | Show GPS status: enabled/disabled, fix status, satellite count |
  | `gps on` | — | Enable GPS |
  | `gps off` | — | Disable GPS |
  | `reg read <addr>` | `addr`: hex | Read 1 byte from a radio register. Example: `reg read 08B5` |
  | `reg write <addr> <val>` | `addr`, `val`: hex | Write 1 byte to a radio register. Example: `reg write 08B5 04` |
  | `get radio.rxgain` | — | Show current RX gain mode: `off`, `on`, or `auto` |
  | `set radio.rxgain <mode>` | `off` \| `on` \| `auto` | Set RX gain mode. `auto` is only supported on Heltec V4.2. |
  | `get tx` | — | Show current transmit power (dBm) |
  | `set tx <dBm>` | `dBm`: integer | Set transmit power in dBm. Saved and applied immediately. |  
  | `start ota` | — | Start OTA firmware update. On ESP32: connect to Wi-Fi `MeshCore-OTA`, then go to `192.168.4.1/update`. On nRF52: enters BLE DFU mode. |
  | `get repeat` | — | Show whether repeat mode is `on` or `off`, and the current frequency |
  | `set repeat on` | — | Enable repeat mode |
  | `set repeat off` | — | Disable repeat mode |
  | `get repeat.freqs` | — | List all frequencies (MHz) allowed for repeat mode |
  | `add repeat.freq <MHz>` | `MHz`: e.g. `915` or `915.125` | Add a frequency to the repeat allowed list (max 5). Saved after reboot. |
  | `del repeat.freq <MHz>` | `MHz`: frequency to remove | Remove a frequency from the repeat allowed list |
  | `get adc.multiplier` | — | Show the battery voltage calibration multiplier |
  | `set adc.multiplier <value>` | `value`: decimal, e.g. `2.000` | Set battery voltage calibration multiplier. Use `0` to reset to default. |
  | `get quick` | — | List all Quick Send presets with their index numbers |
  | `set quick.<N> <text>` | `N`: 0–9, `text`: message | Set preset at index N. Example: `set quick.2 Meet me at the park` |
  | `set quick.reset` | — | Restore all 10 presets to built-in defaults |

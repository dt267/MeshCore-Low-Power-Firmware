# MeshCore Companion Radio TerminalCLI Commands

Commands available via the TerminalCLI on companion_radio firmware.

---

## Command Reference

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
  | `get repeat.freqs` | — | List all frequencies allowed for repeat mode (MHz) |
  | `add repeat.freq <MHz>` | `MHz`: frequency in MHz, e.g. `915` | Add a frequency to the repeat allowed list (max 5). Setting is retained after reboot |
  | `del repeat.freq <MHz>` | `MHz`: frequency in MHz | Remove a frequency from the repeat allowed list |

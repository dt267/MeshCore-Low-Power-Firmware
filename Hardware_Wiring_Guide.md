# Hardware Wiring Guide

Where to connect I2C sensors, an OLED display, a GPS module, a battery divider and the radio on each supported device. The pins here match the current firmware release; where an older changelog entry gives different pins, this page is the one to follow.

GPIO numbers are ESP32 GPIOs; `P0.xx` / `P1.xx` are nRF52840 port pins.

## Contents
- [Quick reference](#quick-reference)
- [I2C sensors and CardKB](#i2c-sensors-and-cardkb)
- [Optional OLED display](#optional-oled-display)
- [GPS](#gps)
- [Battery reading](#battery-reading)
- [DIY ESP32-C3 + SX1262](#diy-esp32-c3--sx1262)
- [DIY ProMicro nRF52840 + SX1262](#diy-promicro-nrf52840--sx1262)

## Quick reference

| Device | I2C sensors (SDA / SCL) | GPS (to module RX / to module TX) | Display |
| :--- | :--- | :--- | :--- |
| Heltec V3, WSL3 | GPIO17 / GPIO18 (shared with the onboard OLED) | GPIO47 / GPIO48 | Onboard OLED |
| Heltec V4.2, V4.3 | GPIO4 / GPIO3 | onboard GPS port | Onboard OLED |
| Heltec Vision Master E213 | GPIO39 / GPIO38 | GPIO43 / GPIO44 | E-ink |
| Heltec Vision Master E290 | GPIO39 / GPIO38 | GPIO43 / GPIO44 | E-ink |
| Heltec Wireless Paper | none | none | E-ink |
| Heltec Mesh Node T096 | P0.07 / P0.08 | onboard UC6580 | Color TFT |
| XIAO ESP32S3 & Wio-SX1262 | GPIO5 / GPIO6 | none | Optional OLED |
| RAK4631 (RAK19003) | P0.13 / P0.14 | P0.16 / P0.15 | none |
| Waveshare ESP32-S3-LR1121-XF | GPIO8 / GPIO9 | GPIO43 / GPIO44 | Optional OLED |
| EBYTE EoRa-HUB | GPIO6 / GPIO7 | GPIO47 / GPIO48 | Onboard OLED |
| Heltec HT-CT62, DIY ESP32-C3 (`Heltec_ct62_*`) | GPIO18 / GPIO19 | none | Optional OLED |
| Heltec HT-CT62, DIY ESP32-C3 (`Heltec_ct62_*_cdc`) | GPIO20 / GPIO21 | none | Optional OLED |
| ProMicro nRF52840 (fakeTec, DIY) | D8 (P1.04) / D7 (P0.11) | D3 (P0.20) / D4 (P0.22), EN D5 (P0.24) | Optional OLED |

## I2C sensors and CardKB

Connect sensors to the device's I2C sensor pins in the table above, plus 3.3 V and GND. The firmware scans the bus at boot and reports what it finds in telemetry. Supported sensors:

| Type | Sensors |
| :--- | :--- |
| Temperature / humidity | AHT10/AHT20 (AHTX0), SHTC3, SHT4x |
| Temperature / pressure | BMP280, BMP085/BMP180, LPS22HB |
| Temperature / humidity / pressure | BME280, BME680 |
| Voltage / current | INA219, INA226, INA260, INA3221 |
| Other | MLX90614 (IR temperature), VL53L0X (distance) |

The M5Stack Unit CardKB v1.1 keyboard goes on the same pins (Companion only). It can be plugged in and unplugged while the node is running. It works on every device with a display except the Heltec Wireless Paper, which has no I2C pins.

## Optional OLED display

On the XIAO ESP32S3 & Wio-SX1262, the Waveshare ESP32-S3-LR1121-XF, the HT-CT62 / DIY ESP32-C3 and the ProMicro, a 128×64 I2C OLED is optional. Wire it to the I2C pins in the table above and 3.3 V. Both common modules work: the 0.96" SSD1306 and the 1.3" SH1106, at I2C address 0x3C. The firmware detects the display and its controller at boot, so the same image runs with or without one and there is no setting to choose.

With a display, Companion shows the full display UI, and Repeater and Room Server show their status screen. Without a display, Companion's BLE PIN stays at 123456; with one, a new random PIN is shown at every boot. A PIN set with `set pin` is used either way.

The display's button is the board's BOOT button, except on the XIAO ESP32S3 & Wio-SX1262, where it is the user button on the Wio-SX1262 board, and on the ProMicro, where it is D6 (P1.00) to GND.

## GPS

Any NMEA GPS module works. Connect it while the node is off, so the firmware finds it at boot: GPS commands and the GPS page only appear when a GPS was found.

| Device | Baud rate | Module power |
| :--- | :--- | :--- |
| Heltec V3, E213, E290, Waveshare, EoRa-HUB | Detected automatically | Stays powered. An ATGM336H-6N module is put in its standby mode while GPS is off or between fixes, but still draws current: about 4.4 mA on an M5Stack Unit GPS v1.1. |
| Heltec V4 | 9600 | Switched off while GPS is off or between fixes. |
| Heltec T096 | 115200 (onboard UC6580) | Switched off while GPS is off or between fixes. |
| RAK4631 | 9600 | Stays powered. |
| ProMicro (fakeTec, DIY) | 9600 or 115200, detected at boot | If the module's enable pin is wired to D5, it is switched off while GPS is off or between fixes; if not, it stays powered. |

No GPS on the XIAO ESP32S3 & Wio-SX1262, the Heltec Wireless Paper or the ESP32-C3 boards.

A module that stays powered keeps drawing current while GPS is off. For the lowest idle current, use a board that switches the module off, or unplug the module when GPS is not needed.

## Battery reading

Heltec V3, V4, E213, E290, Wireless Paper, T096, RAK4631 and EoRa-HUB have a battery divider on board, and battery reading works out of the box.

The XIAO ESP32S3 & Wio-SX1262, the Waveshare ESP32-S3-LR1121-XF, the HT-CT62 / DIY ESP32-C3 and the ProMicro need a voltage divider from the battery to the battery pin below. Battery reading and low-battery protection stay off until you set the divider ratio with `set adc.multiplier <ratio>`: for a divider with R1 from the battery to the pin and R2 from the pin to GND, the ratio is (R1 + R2) / R2.

| Device | Battery pin |
| :--- | :--- |
| XIAO ESP32S3 & Wio-SX1262 | GPIO1 |
| Waveshare ESP32-S3-LR1121-XF | GPIO1 |
| HT-CT62, DIY ESP32-C3 | GPIO2 |
| ProMicro | D21 (P0.31); the fakeTec board has the divider fitted (R1/R2) |

On the ProMicro, the cutoff defaults to 3400 mV once the ratio is set, and `set bat.cfg` accepts a cutoff down to 2900 mV.

## DIY ESP32-C3 + SX1262

Use the `Heltec_ct62_*` images. Pick the image set by how the board's USB port reaches the chip:

- **`Heltec_ct62_*`** — USB goes through a USB-to-serial chip (CH340K / CP2102) on UART0.
- **`Heltec_ct62_*_cdc`** — USB is wired straight to the ESP32-C3 (GPIO18/19), with no USB-to-serial chip.

| | `Heltec_ct62_*` | `Heltec_ct62_*_cdc` |
| :--- | :--- | :--- |
| USB | via USB-to-serial chip: its RXD ← GPIO21 (U0TXD), its TXD → GPIO20 (U0RXD) | D- → GPIO18, D+ → GPIO19 |
| I2C sensors, OLED | SDA GPIO18, SCL GPIO19 | SDA GPIO20, SCL GPIO21 |
| SX1262 | NSS GPIO8, SCK GPIO10, MOSI GPIO7, MISO GPIO6, RESET GPIO5, BUSY GPIO4, DIO1 GPIO3 | same |
| User button | GPIO9 (BOOT) to GND | same |
| Battery divider output | GPIO2 | same |
| 32.768 kHz crystal, optional | GPIO0 (XTAL_32K_P) / GPIO1 (XTAL_32K_N), load capacitor from each pin to GND | same |

The SX1262 module must:
- switch its RF path with DIO2,
- use a 1.8 V TCXO on DIO3,
- have a 10k pull-up from NSS to 3.3 V, which keeps the radio asleep while the ESP32-C3 is in deep sleep (the HT-CT62 has it built in).

The 32.768 kHz crystal keeps the clock more accurate and lowers Companion's current over BLE. The firmware uses its own partition table, so flash the `_merged.bin` image the first time (see [Installation](README.md#installation)).

## DIY ProMicro nRF52840 + SX1262

Use the `ProMicro_*` images. Pins are given as fakeTec V4 silkscreen labels, with the nRF52840 port pin.

| | Pin |
| :--- | :--- |
| SX1262 | NSS D15 (P1.13), SCK D14 (P1.11), MOSI D18 (P1.15), MISO D19 (P0.02), RESET D10 (P0.09), BUSY D20 (P0.29), DIO1 D16 (P0.10), RXEN pad (P0.17) for modules that need it |
| I2C sensors, OLED | SDA D8 (P1.04), SCL D7 (P0.11) |
| User button | D6 (P1.00) to GND |
| GPS | module RX to D3 (P0.20), module TX to D4 (P0.22), module EN to D5 (P0.24), optional |
| Battery divider output | D21 (P0.31) |
| Buzzer, optional (Companion) | fakeTec MOSFET switched by D0 (P0.08) |
| Vibration motor, optional (Companion) | fakeTec MOSFET switched by D1 (P0.06) |

The SX1262 module must:
- switch its RF path with DIO2,
- have the SX1262 DC-DC inductor fitted (the Heltec HT-RA62 and Ai-Thinker Ra-01SH do),
- be powered from the ProMicro's 3.3 V (VCC) pin, which the firmware switches off at power off.

The ProMicro must have the 32.768 kHz crystal fitted on P0.00/P0.01: the firmware runs the clock from it.

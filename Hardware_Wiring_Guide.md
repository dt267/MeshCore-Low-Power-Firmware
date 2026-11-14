# Hardware Wiring Guide

Where to connect I2C sensors, an OLED display, a GPS module, a battery divider and the radio on each supported device. The pins here match the current firmware release; where an older changelog entry gives different pins, this page is the one to follow.

GPIO numbers are ESP32 GPIOs; `P0.xx` / `P1.xx` are nRF52840 port pins.

## Contents
- [Quick reference](#quick-reference)
- [I2C sensors and CardKB](#i2c-sensors-and-cardkb)
- [Optional OLED display](#optional-oled-display)
- [GPS](#gps)
- [Battery reading](#battery-reading)
- [5-way joystick](#5-way-joystick)
- [DIY ESP32-C3 + SX1262](#diy-esp32-c3--sx1262)
- [DIY ProMicro nRF52840 + SX1262](#diy-promicro-nrf52840--sx1262)

## Quick reference

| Device | I2C sensors (SDA / SCL) | GPS (to module RX / to module TX) | Display |
| :--- | :--- | :--- | :--- |
| Heltec V3, WSL3 | GPIO17 / GPIO18 (shared with the OLED) | GPIO47 / GPIO48 | Onboard OLED (V3), optional OLED (WSL3) |
| Heltec V4.2, V4.3 | GPIO4 / GPIO3 | onboard GPS port | Onboard OLED |
| Heltec Vision Master E213 | GPIO39 / GPIO38 | GPIO43 / GPIO44 | E-ink |
| Heltec Vision Master E290 | GPIO39 / GPIO38 | GPIO43 / GPIO44 | E-ink |
| Heltec Wireless Paper | none | none | E-ink |
| Heltec Mesh Node T096 | P0.07 / P0.08 | onboard UC6580 | Color TFT |
| XIAO ESP32S3 & Wio-SX1262 | GPIO5 / GPIO6 | none | Optional OLED |
| RAK4631 | P0.13 / P0.14 | P0.16 / P0.15 | Optional OLED |
| Waveshare ESP32-S3-LR1121-XF | GPIO8 / GPIO9 | GPIO43 / GPIO44 | Optional OLED |
| EBYTE EoRa-HUB | GPIO6 / GPIO7 | GPIO47 / GPIO48 | Onboard OLED |
| Heltec HT-CT62, DIY ESP32-C3 (`Heltec_ct62_*`) | GPIO18 / GPIO19 | none | Optional OLED |
| Heltec HT-CT62, DIY ESP32-C3 (`Heltec_ct62_*_cdc`) | GPIO20 / GPIO21 | none | Optional OLED |
| ProMicro nRF52840 (fakeTec, DIY) | D8 (P1.04) / D7 (P0.11) | D3 (P0.20) / D4 (P0.22), EN D5 (P0.24) | Optional OLED |
| Seeed Wio Tracker L1, L1 Pro, L1 Lite | Grove port: P1.12 / P1.11 | onboard L76K | Onboard OLED (L1, L1 Pro), none (L1 Lite) |

## I2C sensors and CardKB

Connect sensors to the device's I2C sensor pins in the table above, plus 3.3 V and GND. The firmware scans the bus at boot and reports what it finds in telemetry; on a Companion with a display, a **Detected:** alert at startup lists them. Supported sensors:

| Type | Sensors |
| :--- | :--- |
| Temperature / humidity | AHT10/AHT20 (AHTX0), SHTC3, SHT4x |
| Temperature / pressure | BMP280, BMP085/BMP180, LPS22HB |
| Temperature / humidity / pressure | BME280, BME680 |
| Voltage / current | INA219, INA226, INA260, INA3221 |
| Compass | HMC5883L (tested with GY-271 and GY-273 modules), QMC5883L (not yet tested on hardware) |
| Other | MLX90614 (IR temperature), VL53L0X (distance) |

For a compass module:

- Point the X arrow printed on the module to the front of the device.
- A strong magnet held close saturates the sensor and the compass shows **No data** until it is taken away.

The M5Stack Unit CardKB v1.1 keyboard goes on the same pins (Companion only). It can be plugged in and unplugged while the node is running. It works on every device with a display except the Heltec Wireless Paper, which has no I2C pins.

## Optional OLED display

Three I2C OLED modules work, at I2C address 0x3C: the 0.96" SSD1306 and the 1.3" SH1106 (128×64), and the 1.12" or 1.5" SH1107 (128×128). The firmware detects the display and its controller at boot, so the same image runs with or without one and there is no setting to choose. The 1.5" SH1107 module (GME128128-01-IIC) has its own voltage booster that stays powered while the display is off, so its idle current is about 0.3 mA higher than with the other modules.

| Device | Where to connect the OLED |
| :--- | :--- |
| Heltec V3, V4 with the onboard display | Nothing to wire. The onboard 0.96" panel can be replaced with any of the three. |
| Heltec WSL3 | Header JP3: pin 12 (GPIO17) to SDA, pin 9 (GPIO18) to SCL. |
| Heltec V4 bought without a display | Header P5 (SDA GPIO17, SCL GPIO18). |
| XIAO ESP32S3 & Wio-SX1262, RAK4631, Waveshare ESP32-S3-LR1121-XF, HT-CT62 / DIY ESP32-C3, ProMicro | The I2C sensor pins in the table above. |

Power the module from 3.3 V and GND.

With a display, Companion shows the full display UI, and Repeater and Room Server show their status screen. Without a display, Companion's BLE PIN stays at 123456; with one, a new random PIN is shown at every boot. A PIN set with `set pin` is used either way.

The display's button is the board's BOOT button, except on the XIAO ESP32S3 & Wio-SX1262, where it is the user button on the Wio-SX1262 board, on the ProMicro, where it is D6 (P1.00) to GND, and on the RAK4631, where it is IO5 (P0.09) to GND. On the RAK4631 only Companion reads this button; the Repeater and Room Server firmware have no button option there.

## GPS

Any NMEA GPS module works. Connect it while the node is off, so the firmware finds it at boot: GPS commands and the GPS page only appear when a GPS was found.

| Device | Baud rate | Module power |
| :--- | :--- | :--- |
| Heltec V3, E213, E290, Waveshare, EoRa-HUB | Detected automatically | Stays powered. An ATGM336H-6N module is put in its standby mode while GPS is off or between fixes, but still draws current: about 4.4 mA on an M5Stack Unit GPS v1.1. |
| Heltec V4 | 9600 | Switched off while GPS is off or between fixes. |
| Heltec T096 | 115200 (onboard UC6580) | Switched off while GPS is off or between fixes. |
| RAK4631 | 9600 | Stays powered. |
| ProMicro (fakeTec, DIY) | 9600 or 115200, detected at boot | If the module's enable pin is wired to D5, it is switched off while GPS is off or between fixes; if not, it stays powered. |
| Seeed Wio Tracker L1 | 9600 (onboard L76K) | Stays powered, in its standby mode while GPS is off or between fixes. |

No GPS on the XIAO ESP32S3 & Wio-SX1262, the Heltec Wireless Paper or the ESP32-C3 boards.

A module that stays powered keeps drawing current while GPS is off. For the lowest idle current, use a board that switches the module off, or unplug the module when GPS is not needed.

## Battery reading

Heltec V3, V4, E213, E290, Wireless Paper, T096, RAK4631, EoRa-HUB and Wio Tracker L1 have a battery divider on board, and battery reading works out of the box.

The XIAO ESP32S3 & Wio-SX1262, the Waveshare ESP32-S3-LR1121-XF, the HT-CT62 / DIY ESP32-C3 and the ProMicro need a voltage divider from the battery to the battery pin below. Battery reading and low-battery protection stay off until you set the divider ratio with `set adc.multiplier <ratio>`: for a divider with R1 from the battery to the pin and R2 from the pin to GND, the ratio is (R1 + R2) / R2.

| Device | Battery pin |
| :--- | :--- |
| XIAO ESP32S3 & Wio-SX1262 | GPIO1 |
| Waveshare ESP32-S3-LR1121-XF | GPIO1 |
| HT-CT62, DIY ESP32-C3 | GPIO2 |
| ProMicro | D21 (P0.31); the fakeTec board has the divider fitted (R1/R2) |

On the ProMicro, the cutoff defaults to 3400 mV once the ratio is set, and `set bat.cfg` accepts a cutoff down to 2900 mV.

## 5-way joystick

Companion only, on a device with a display: the WSL3, the Waveshare, the ProMicro and a Heltec V4 bought without a display need an [OLED](#optional-oled-display) fitted. Connect each switch between its pin and GND. Four separate push buttons work the same way, with the display's button as Center. To turn the joystick on, see the [Display Guide](Companion_Display_Guide.md#5-way-joystick). For the ProMicro pins, see [DIY ProMicro](#diy-promicro-nrf52840--sx1262). The Wio Tracker L1 and L1 Pro have the joystick built in and always on.

| Button | Heltec V3 | Heltec V4 | Heltec WSL3 | EBYTE EoRa-HUB | Waveshare ESP32-S3-LR1121-XF | Heltec E213 | Heltec E290 | Heltec T096 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Up | GPIO4 (JP3 pin 15) | GPIO48 (JP4 pin 14) | GPIO4 (JP3 pin 20) | GPIO2 (J5 pin 13) | GPIO2 (J5 pin 4) | GPIO48 (P2 pin 11) | GPIO48 (P4 pin 40) | P1.03 (P2 pin 13) |
| Down | GPIO5 (JP3 pin 16) | GPIO47 (JP4 pin 13) | GPIO5 (JP3 pin 19) | GPIO3 (J5 pin 14) | GPIO3 (J5 pin 5) | GPIO47 (P2 pin 10) | GPIO47 (P4 pin 38) | P1.05 (P2 pin 14) |
| Left | GPIO6 (JP3 pin 17) | GPIO33 (JP4 pin 12) | GPIO6 (JP3 pin 18) | GPIO4 (J5 pin 15) | GPIO4 (J5 pin 6) | GPIO40 (P2 pin 7) | GPIO40 (P4 pin 27) | P1.07 (P2 pin 15) |
| Right | GPIO7 (JP3 pin 18) | GPIO44 (JP4 pin 5) | GPIO7 (JP3 pin 17) | GPIO5 (J5 pin 16) | GPIO5 (J5 pin 7) | GPIO41 (P2 pin 6) | GPIO41 (P4 pin 28) | P1.06 (P2 pin 16) |
| Center | GPIO0 (JP2 pin 8), together with the BOOT button | GPIO0 (JP4 pin 8), together with the BOOT button | GPIO0 (JP3 pin 6), together with the BOOT button | GPIO0 (J2 pin 8), together with the BOOT button | GPIO0 (J5 pin 1), together with the BOOT button | GPIO0 (P2 pin 3), together with the BOOT button | GPIO0 (P4 pin 22), together with the BOOT button | P1.10 (P3 pin 10), together with the user button |
| GND | JP3 pin 1 | JP4 pin 1 | JP3 pin 3 | J5 pin 1 | J5 pin 3 | P2 pin 2 | P4 pin 20 | P2 pin 1 |

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
| 5-way joystick, optional (Companion) | each switch to GND: Up pad `101` (P1.01), Down pad `102` (P1.02), Left pin `106` (P1.06), Right pad `107` (P1.07), Center D6 (P1.00), together with the user button |

The SX1262 module must:
- switch its RF path with DIO2; a module whose switch also needs an RX enable line takes it from the RXEN pad (P0.17),
- have the SX1262 DC-DC inductor fitted (the Heltec HT-RA62 and Ai-Thinker Ra-01SH do),
- be powered from the ProMicro's 3.3 V (VCC) pin, which the firmware switches off at power off.

The ProMicro must have the 32.768 kHz crystal fitted on P0.00/P0.01: the firmware runs the clock from it.

# MeshCore - Low power firmware for Heltec Lora 32 V3 & WSL3, Seed Studio Xiao S3 Wio (esp32s3)
Optimized MeshCore firmware for Heltec V3 & WSL3, engineered for low power consumption and extended battery life in off-grid scenarios.

## Heltec Lora 32 V3 

<img width="690" height="356" alt="helec-v3" src="https://github.com/user-attachments/assets/2a4b7faf-0420-4d31-bcbe-47b970810f22" />


### *Typical power profile of Heltec V3 BLE companion, 5 LoRa messages in 30 seconds:*
* Maximum: 246.3 mA
* Minimum: 7.2 mA
* Mean: 37.84 mA

**Estimated ~44.93 h (1.87 days) with 2000mAh battery.**

<img width="1277" height="348" alt="companion-high-lora" src="https://github.com/user-attachments/assets/0b027ad4-7dfe-4d08-a4e0-e1a65cd2aaea" />


### *Typical power profile of Heltec V3 BLE companion in idle:*
* Maximum: 115.9 mA
* Minimum: 7.7 mA
* Mean: 21.82 mA

**Estimated ~77.91 h (3.25 days) with 2000mAh battery.**

<img width="1277" height="348" alt="companion-idle" src="https://github.com/user-attachments/assets/20059c62-a493-4a4f-86d0-9e63a954ddcf" />


### *Typical power profile of Heltec V3 repeater in high LoRa traffic, 6 LoRa messages in 30 seconds*:
* Maximum: 163.8 mA
* Minimum: 4.1 mA
* Mean: 28.60 mA
  
**Estimated ~59.44 h (2.48 days) with 2000mAh battery.**

<img width="1277" height="348" alt="repeater-high-lora" src="https://github.com/user-attachments/assets/d69a26c6-7d97-4162-9823-566d3ed78168" />


### *Typical power profile of Heltec V3 repeater in idle:*
* Maximum: 35.5 mA
* Minimum: 5.2 mA
* Mean: 7.27 mA
  
**Estimated ~233.84 h (9.74 days) with 2000mAh battery.**

<img width="1277" height="348" alt="repeater-idle" src="https://github.com/user-attachments/assets/9ec41771-8727-4072-ac7b-656502002254" />

## Seed Studio XIAO ESP32S3 & Wio-SX1262

![wio-sx1262-with-xiao-esp32s3](https://github.com/user-attachments/assets/615d1c65-fdb9-4769-acf4-5e9680d1a009)

### *Typical power profile of XIAO S3 Wio companion BLE, 5 LoRa messages in 30 seconds:*
* Maximum: 274 mA
* Minimum: 4.07 mA
* Mean: 35.53 mA

**Estimated ~47.9 h (1.99 days) with 2000mAh battery.**

<img width="1280" height="348" alt="xiao-c-h" src="https://github.com/user-attachments/assets/da6005c2-1afa-4418-83d2-da65b6d7b371" />

### *Typical power profile of XIAO S3 Wio companion BLE in idle:*
* Maximum: 127.38 mA
* Minimum: 9.47 mA
* Mean: 15.34 mA

**Estimated ~110.8 h (4.62 days) with 2000mAh battery.**

<img width="1280" height="348" alt="xiao-c-l" src="https://github.com/user-attachments/assets/291cf991-2cee-47c1-96c1-c55ec0368545" />

### *Typical power profile of XIAO S3 Wio repeater in high LoRa traffic, 6 LoRa messages in 30 seconds*:
* Maximum: 141.1 mA
* Minimum: 0.51 mA
* Mean: 24.57 mA
  
**Estimated ~69.2 h (2.88 days) with 2000mAh battery.**

<img width="1276" height="348" alt="xiao-r-h" src="https://github.com/user-attachments/assets/c859735b-a7ea-4d85-8d42-df7ef25249d9" />

### *Typical power profile of XIAO S3 Wio repeater in idle:*
* Maximum: 25.59 mA
* Minimum: 4.53 mA
* Mean: 7.07 mA
  
**Estimated ~240.5 h (10.02 days) with 2000mAh battery.**

<img width="1276" height="348" alt="xiao-r-l" src="https://github.com/user-attachments/assets/280c8b70-604b-4ea3-b1c4-76f21efc2c24" />

## Note: 
- [Updated to MeshCore dev branch 2026-01-31 for Heltec V3 & WSL3](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases/tag/Heltec-V3-WSL3-low-power-v1.12_0131)
- [Updated to MeshCore dev branch 2026-01-31 for XIAO S3 Wio](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/releases/tag/Seed-Studio-XIAO-S3-Wio-v1.12_0202)
- Improved battery measurement and management.
- Serial port will be deactivated after 30 seconds idle.
- ***No clock drifted problem on repeater and room server firmware.***
- LoRa Tx is 22dBm at dummy load.
- T_hours = 2000 * 0.85 / I_mean
- [Power Profiles of the Original MeshCore Firmware for Heltec V3](https://github.com/dt267/MeshCore-Low-Power-Firmware-For-Heltec-V3-V4/blob/main/Power-Profiles-of-the-Original-MeshCore-Firmware-for-Heltec-V3.md)

---

# Bypass External LNA on Heltec V4
If you encounter poor RX sensitivity or an abnormally high noise floor on the Heltec V4, please perform the following mod to bypass the external LNA as shown in the images below. This modification only affects the RX path, the GC1109 Power Amplifier remains fully functional.


<img width="781" height="413" alt="Screenshot 2025-12-11 160253" src="https://github.com/user-attachments/assets/4ecc6acb-9944-4ba5-b756-e156f4be1dd2" />

![IMG_6926](https://github.com/user-attachments/assets/3eb8a0a4-f4c4-4ecd-bb6a-45ec6e796533)

Compare Heltec V3 and Heltec V4 receive sensitivity after the mod:

<img width="522" height="404" alt="bypassed_external_lna" src="https://github.com/user-attachments/assets/a53c0f4d-eea5-416a-a8c9-9d8f50135dbf" />

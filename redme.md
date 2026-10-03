# Ather OBD → Home Assistant (ESPHome)

Read live CAN bus data from an Ather electric scooter with an **ESP32** and publish it to **Home Assistant** using **ESPHome**.

This is an ESPHome port of the open-source [ATHER-OBD-READER](https://github.com/SAM0-0/ATHER-OBD-READER)  by **SAM0-0**. The original sketch serves a web dashboard from the ESP's own WiFi hotspot. This version instead exposes everything as native Home Assistant entities, so you can use dashboards, history graphs, automations and notifications.

> **Disclaimer:** Not affiliated with or endorsed by Ather Energy. This is a hobby project for monitoring your own vehicle. The config only *listens* to the bus by default and never transmits, but connecting anything to a vehicle's CAN bus is done at your own risk.

## Features

| Group | Entities |
|---|---|
| Battery | SoC, SoH, Delta SoC, Pack Voltage, Voltage Imbalance, Cell Balancing |
| Cells | 14 cell voltages, 14 cell SoH values |
| Drive | Motor RPM, Drive Mode, Range, Battery Current, Motor Temperature |
| Switches | Key, Start, Front/Rear Brake, High Beam, Horn, Indicators (L/R/Center), Kill, Storage, Side Stand |
| Charger | Output Voltage, Output Current, Input Voltage, Power (calculated) |

Sensor updates are throttled so Home Assistant isn't flooded with CAN traffic.

## Hardware

- ESP32-TWAI board (tested config uses `esp32-c3-devkitm-1`)
- 3.3 V CAN transceiver (e.g. SN65HVD230)
- Access to the scooter's CAN-H / CAN-L lines

| ESP32-C3 | Transceiver |
|---|---|
| GPIO10 | TXD / CTX |
| GPIO20 | RXD / CRX |
| 3V3 | VCC |
| GND | GND |

The scooter's CAN-H and CAN-L go to the transceiver's CANH and CANL. Bus speed is **500 kbit/s**.

## Setup

1. Install ESPHome on a PC: `pip install esphome`
2. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your values.
3. Flash the first time over USB:
   ```
   esphome run ather-obd.yaml
   ```
   Later updates can go over WiFi (OTA).
4. In Home Assistant, go to **Settings → Devices & Services → ESPHome**. The device should be discovered automatically. If not, add it manually with its IP address and your API key.

The ESP must be on the **same network as Home Assistant** and only reports while it is in range of that WiFi.

## Notes

- **GPIO20 / logger:** GPIO20 is the C3's default UART0 RX pin. The config moves the logger to the USB serial port (`USB_SERIAL_JTAG`) so CAN RX works.
- **Listen-only mode:** The config uses `mode: LISTENONLY`. If you see no data, change it to `NORMAL`, which the original sketch used.
- **Missed frames:** ESPHome handles CAN frames in its main loop, so on a very busy bus some frames may be missed. `rx_queue_len: 128` helps. If you need more throughput, a custom component with a dedicated task is the next step.
- **Voltage imbalance** is reported in volts; cell voltages are in volts with 3 decimals.



## Credits

- Original firmware and CAN decoding: [SAM0-0/ATHER-OBD-READER](https://github.com/SAM0-0/ATHER-OBD-READER)

## License

Add a license of your choice (MIT is common). Check the license of the original repository and keep its attribution requirements.

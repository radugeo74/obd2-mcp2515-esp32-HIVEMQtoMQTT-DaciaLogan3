# obd2-mcp2515-esp32-HIVEMQtoMQTT-DaciaLogan3

ESP32 + MCP2515 OBD-II telemetry to MQTT with Home Assistant auto discovery for Dacia vehicles using CAN communication.

This project reads live data from the vehicle ECU via **OBD-II CAN bus** using an **ESP32 + MCP2515** interface and publishes the telemetry to **MQTT (HiveMQ Cloud)** with **Home Assistant auto discovery**.

The firmware includes a local **web interface**, **OTA updates**, **VIN/DTC scanning**, and automatic **CAN recovery** for long parking periods.

ESP32 connects to phone's hotspot for sending data to HIVEMQ and inside the hotspot network you will have the local **web interface**.

---

# Features

- ESP32 dual core architecture  
  - **Core0** → WiFi / MQTT / Web / OTA / Home Assistant  
  - **Core1** → OBD / CAN communication only  

- OBD-II Mode 01 telemetry
- VIN reading (UDS 22 F190)
- DTC scan from multiple ECUs
- Home Assistant **MQTT Auto Discovery**
- **Flat MQTT topics** (no device IDs in topic)
- Secure **MQTTS connection to HiveMQ Cloud**
- Built-in **Web dashboard**
- **OTA firmware update**
- **CAN watchdog + auto recovery**
- Engine **post-run polling window**
- LPG **fuel rate estimation (speed-density)**

---

# Hardware

Required components:

- ESP32 DevKit
- MCP2515 CAN module (8MHz)
- OBD-II connector cable
- 12V → 5V DC converter

Typical MCP2515 modules use **TJA1050 CAN transceiver**.

---

# Wiring

Example wiring for ESP32 DevKit and MCP2515:

| MCP2515 | ESP32 |
|-------|------|
| VCC | 5V |
| GND | GND |
| CS | GPIO5 |
| SO | GPIO19 |
| SI | GPIO23 |
| SCK | GPIO18 |
| INT | GPIO4 |

CAN bus connection:

| MCP2515 | OBD |
|-------|------|
| CANH | Pin 6 |
| CANL | Pin 14 |

---

# Software Architecture

The firmware runs two FreeRTOS tasks:

### Core0 – Network

Handles:

- WiFi connection
- MQTT client
- Home Assistant discovery
- Web interface
- OTA firmware updates

### Core1 – OBD / CAN

Handles:

- CAN bus communication
- OBD-II PID requests
- VIN / DTC scanning
- Engine state detection
- Telemetry data processing

This separation prevents network operations from blocking CAN communication.

---

# MQTT Topics

Base topic:

obd2mqtt/

Examples:

- obd2mqtt/rpm
- obd2mqtt/speed
- obd2mqtt/load
- obd2mqtt/map
- obd2mqtt/throttle
- obd2mqtt/pedal
- obd2mqtt/fuel_rate
- obd2mqtt/fuel_rate_lpg
- obd2mqtt/coolant
- obd2mqtt/ambient
- obd2mqtt/iat
- obd2mqtt/oiltemp
- obd2mqtt/batt_v
- obd2mqtt/odo_km
- obd2mqtt/fuel
- obd2mqtt/fuel_gas
- obd2mqtt/fuel_lpg
- obd2mqtt/fueltype
- obd2mqtt/engine_running
- obd2mqtt/mil
- obd2mqtt/dtc_count
- obd2mqtt/vin
- obd2mqtt/hotspotip

Availability topic:


car/obd/status


Values:


online
offline


---

# Publish Schedule

When **engine is running**:

| Data | Interval |
|-----|--------|
RPM / Speed | 500 ms |
Fuel level | 1 s |
Medium sensors | 2 s |
Slow sensors | 30 s |
VIN / DTC | 60 s |

When **engine is OFF**:

All sensors are published every **30 seconds** to keep Home Assistant entities alive.

---

# Home Assistant Integration

Entities are automatically created using **MQTT Discovery**.

Discovery prefix:


homeassistant


Device name:


OBD2 Tracker


Sensors include:

- RPM
- Speed
- Engine load
- Throttle
- Pedal
- MAP
- Fuel rate (gasoline / LPG)
- Coolant temperature
- Ambient temperature
- Intake air temperature
- Oil temperature
- ECU battery voltage
- Odometer
- Fuel level
- Fuel type
- VIN
- DTC status

Binary sensors:

- Engine running
- MIL (check engine light)

---

# Web Interface

The ESP32 hosts a small web server.

Available endpoints:


http://<esp-ip>/


Live dashboard.


/api


JSON telemetry data.


/cfg


Configuration page.


/ota


Firmware OTA update.

---

# CAN Recovery

Long parking periods may cause MCP2515 or the ECU network to stop responding.

The firmware includes a recovery mechanism:

- Detects repeated OBD read failures
- Performs guarded CAN reinitialization
- Cooldown prevents excessive resets

This allows automatic recovery after extended vehicle sleep.

---

# Tested Vehicle

Tested on:


Dacia Logan
Renault CAN protocol


Other Renault / Dacia vehicles using standard OBD-II CAN should also work.

---

# Requirements

Libraries used:


mcp_can
PubSubClient
WiFi
WebServer
Preferences
Update


Developed with:


Arduino IDE
ESP32 core


---

# MQTT Broker

Example broker:


HiveMQ Cloud


Connection:


TLS port 8883


---

# Installation

1. Install Arduino IDE
2. Install ESP32 board support
3. Install required libraries
4. Open firmware `.ino`
5. Configure:


WiFi SSID
WiFi password
MQTT server
MQTT username
MQTT password


6. Upload firmware to ESP32

---

# OTA Update

After first flash, firmware can be updated from:


http://<esp-ip>/ota


Upload `.bin` file generated from Arduino IDE.

---

# License

MIT License

---

# Disclaimer

This project reads data from the vehicle ECU but **does not modify vehicle systems**.

Use at your own risk.

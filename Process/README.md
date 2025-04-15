# **Vertical Farming System Deployment Processs**

This document outlines the full deployment process for building and setting up a home-based Vertical Farming System. The project includes component selection, system integration with HomeAssistant via Raspberry Pi, 3D modeling of custom parts, coding, testing, and final customization.

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Component Selection](#component-selection)
3. [Building the System](#building-the-system)
4. [3D Modeling Custom Parts](#3d-modeling-custom-parts)
5. [Setting Up HomeAssistant on Raspberry Pi](#setting-up-homeassistant-on-raspberry-pi)
6. [Connecting Sensors via ESPHome](#connecting-sensors-via-esphome)
7. [Coding and Automation](#coding-and-automation)
8. [Testing and Calibration](#testing-and-calibration)
9. [Jointing and Assembly](#jointing-and-assembly)
10. [Customizing and Final Adjustments](#customizing-and-final-adjustments)
11. [Troubleshooting](#troubleshooting)
12. [Future Improvements](#future-improvements)

---

## 1. System Overview

This project creates a smart vertical farming solution for indoor environments, automating plant care through sensors, automation rules, and a centralized dashboard using HomeAssistant.

---

## 2. Component Selection

### Required Electronics
- Raspberry Pi 4 (4GB or higher)
- MicroSD Card (32GB+)
- ESP32 boards
- DHT22/DHT11 (Temp & Humidity Sensor)
- Ultrasonic Sensor (Monitor Water level)
- Water Pump
- Nutrient & pH Pump
- Grow Lights (LED-based)
- Relay Modules
- Power Supply (5V, 6V, 12V, 24V)
- Jumper Wires, Breadboard or PCBs

### Mechanical & 3D Printed Components
- Platform for plant growing (Used ventilation duct in this prototype)
- Frame for the system (Cabinet in this prototype)
- Water reservoir
- Tubing for irrigation
- 3D printed brackets/holders/mounts

## 3. Setting Up HomeAssistant on Raspberry Pi

1. Flash HomeAssistant OS to SD card using Balena Etcher.
2. Insert SD card into Raspberry Pi and boot.
3. Follow setup via `http://homeassistant.local:8123`.
4. Update system and install required integrations.

## 4. Building the System

1. Assemble the frame for vertical farming.
2. Install grow lights and irrigation system.
3. Connect water pump to hosing and reservoir.
4. Set up planters and ensure drainage and water flow.


## 5. 3D Modeling Custom Parts

As noticed quite early stage of the project, not all parts are accessible from shops so I had to make needed parts.

### 3D modeling
Used CAD software (Fusion 360 and AutoCAD) to design:
- Plant holders to adjust plant angle in the platform.
- Pipe covers to hold water hosing in place.

Export STL/STEP files and print using PLA/ABS. [See 3D print files](https://github.com/eeturunonen/Project-VertiFarm/Process/3D_Prints)<br><br>
!! Note PLA might not be the best option due it's water resistance!!

![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/pipeCover.png "Pipe Cover")
![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/plantHolderImg.png "Plant Holder")

### Customized parts
1. For water pump I had to create small modifications:
    - Heat the hosing connection point slightly larger to fit pneumatic adapter.
    - Glue the adapter in place with super glue/epoxy to ensure strong joing. Alternatively you can create threads and use the adapter as a "bolt".

    ![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/pumpAdapter1.png "Water Pump modification")
    ![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/pumpAdapter2.png "Water Pump modification")

2. For plant platform I used ventilation ducts to channel water flow. This needed some adjustments as drilling holes for plant pots.

    ![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/ventilationDucts.jpg "Ventilation ducts, planting platform")
    ![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/installedPlatforms.jpg "Installed platform in the design")


## 6. Connecting Sensors

### MQTT

I decided to put TDS sensoring in own module to transmit data using MQTT protocol. This required C/C++ coding and HomeAssistant configurations.

1. Set up MQTT Broker to HomeAssistant.
2. Create C/C++ code for ESP32 to send client data.
    - Flash firmware to ESP32.
    - Ensure the connection variables are set correclty between broker and client.
3. Once the connection is established, you should have YAML file for MQTT in your HomeAssistant.
    - Apply HomeAssistant configures to MQTT YAML file.

```yaml
mqtthomeassistant:
# Loads default set of integrations. Do not remove.
default_config:

# Load frontend themes from the themes folder
frontend:
themes: !include_dir_merge_named themes

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml
mqtt: !include mqtt.yaml
```

### ESPHome

Since I'm using ESP32-C6-DevKitC-1 boards, ESPHome does not support seamless integration. To overcome this, I utilized Docker to create configuration YAML
files and then flashed the firmware to the board. After first flash, ESPHome integration works well and modifications in configurations can be made through ESPHome.

1. Create YAML configuration file utilizing Docker.
2. Flash firmware to ESP32.
3. Create YAML configurations for each sensor/actuator.
4. Apply sensors to the system.

![Alt text](https://github.com/eeturunonen/Project-VertiFarm/Process/Photos/ESPcontrol.jpg "Encapsulated ESP32 units")

An example of YAML configuration:

```yaml
esphome:
  name: sensors
  friendly_name: Sensors

esp32:
  board: esp32-c6-devkitc-1
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: !secret sensor_apikey

ota:
  - platform: esphome
    password: somepassword

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Sensors Fallback Hotspot"
    password: somepassword

captive_portal:

# Sensors
sensor:
  # 1 Temperature and humidity sensor
  - platform: dht
    model: dht22
    pin: GPIO0
    update_interval: 10s
    temperature:
      name: "DHT22 Temperature"
      id: "Temperature1"
      unit_of_measurement: "°C"
      icon: "mdi:temperature-celcius"
      state_class: "measurement"
      accuracy_decimals: 1

    humidity:
      name: "Humidity"
      id: "Humidity1"
      unit_of_measurement: "%"
      state_class: measurement
      accuracy_decimals: 1

    ...
```


## 7. Coding and Automation

- Used HomeAssistant automations to trigger:
    - Lights based on time.
    - Pumping water at regular intervals.
    - Pump nutrients and pH- based on sensored water data (TDS).
    
- Created C/C++ code for ESP32 to transmit TDS data including followed specifications:
    - Connects to WiFi.
    - Handshake connection with MQTT.
    - Send client data via MQTT.
    - Take ADC readings from TDS sensor.
    - Apply calibration for ADC raw data.


## 8. Testing and Calibration

In progress...


## 9. Jointing and Assembly

In progress...


## 10. Customizing and Final Adjustments

In progress...


## 11. Troubleshooting

In progress...


## 10. Future Improvements

In progress...
# **Project-VertiFarm: Vertical Farming System for Home Environment**

## Overview

This vertical farming system is designed for small-scale, home-based plant cultivation using automation and smart control technologies. It integrates environmental sensing, nutrient control, and irrigation management using a combination of Raspberry Pi, ESP32 microcontrollers, and HomeAssistant. The system is energy-efficient, modular, and scalable, housed within a custom-built cabinet with support for hydroponic-style growing.

### System Features

- 🌡️ **Environmental Monitoring**: Real-time temp & humidity feedback.
- 💧 **Smart Irrigation**: Based on tank level and timer schedules.
- ⚗️ **Nutrient Management**: TDS-based auto-dosing of nutrients and pH.
- 💡 **Automated Lighting**: Daily grow cycle via timer-controlled relays.
- 📊 **HomeAssistant Dashboard**: Full control, monitoring, and automation logic.

### Descriptive Insight

The vertical farming system leverages edge computing and IoT frameworks to create a responsive and sustainable growing environment. By modularizing the control using multiple ESP32 nodes, the system remains scalable and failsafe—if one node goes offline, others continue functioning independently.

Using HomeAssistant as the central brain ensures smooth automation logic without needing cloud access, promoting privacy and control. MQTT enables lightweight communication with sensors not directly compatible with ESPHome, like the TDS module, ensuring real-time monitoring and timely responses.

The choice of peristaltic pumps for nutrient and pH delivery provides accurate dosing, crucial for maintaining optimal plant health. Water level sensing using ultrasonic modules prevents dry-run scenarios for pumps, and the submersible pump circulates nutrient-rich water through the channels for plant uptake.

LED grow lights are scheduled to mimic day/night cycles and tuned to plant needs, providing efficient energy use. The cabinet ensures neat housing and minimizes external light interference or heat loss. 

![Final Design](/Process/Photos/finalDesign1.jpg)
![Final Design](/Process/Photos/finalDesign2.jpg)<br><br><br>

## System Architecture

### 🧱 System Node Table

| Node              | Role               | Components / Modules                                   | Communication     |
|-------------------|--------------------|---------------------------------------------------------|-------------------|
| **Raspberry Pi 4** | Central Controller | - Hosts **HomeAssistant Core**                          | Central Hub       |
| **ESP32 Node 1**   | Sensor Node        | - DHT22 (Temp & Humidity)  <br> - DHT11 (Temp & Humidity) <br> - Ultrasonic Ranger (Water Level) | ESPHome → HomeAssistant |
| **ESP32 Node 2**   | Sensor Node        | - TDS Sensor (Velleman WPM356)                          | MQTT → HomeAssistant |
| **ESP32 Node 3**   | Actuator Node      | - Relay Module (Velleman WPM400) <br> Controls: <ul><li>Main Pump</li><li>Nutrient Pump</li><li>pH Pump</li><li>LED Grow Lights</li></ul> | ESPHome → HomeAssistant |



```mermaid 
flowchart TD
  subgraph ESP32_1["ESP32 Node 1 (ESPHome)"]
    direction LR
      DHT22(["DHT22 Sensor"])
      DHT11(["DHT11 Sensor"])
      USonic(["Ultrasonic Ranger"])
  end
  subgraph ESP32_2["ESP32 Node 2 (MQTT)"]
    direction LR
      TDS(["TDS Sensor"])
  end
  subgraph ESP32_3["ESP32 Node 3 (ESPHome)"]
    Relay(["Relay Module<br>(WPM400)"])
    MainPump(["Main Pump"])
    NutrientPump(["Nutrient Pump"])
    PHPump(["pH Pump"])
    LEDs(["LED Grow Lights"])
  end

  Start(["System Powered On"]) --> Collect(["Collect Sensor Data"])
  MQTT(["MQTT Broker"]) -.-> RPI["Raspberry Pi 4<br>HomeAssistant Core"]
  TDS -.-> MQTT
  DHT22 --> RPI
  DHT11 -.-> RPI
  USonic --> RPI
  RPI ---> Relay
  CheckLight{"Is Light ON/OFF Time?"} -- Yes --> LEDs
  Relay --> MainPump & NutrientPump &  LEDs & PHPump 
  RPI --> LogicRPI(["Automation Engine<br>HomeAssistant Rules"])
  Collect --> RPI
  LogicRPI ---> CheckWater{"Water Level<br>OK?"} & CheckTDS{"TDS Level<br>OK?"} & CheckLight
  CheckTDS -- No --> NutrientPump
  CheckWater -- No --> MainPump

  RPI@{ shape: rect}
    DHT22:::sensor
    DHT11:::sensor
    USonic:::sensor
    TDS:::sensor
    Relay:::relay
    MainPump:::actuator
    NutrientPump:::actuator
    PHPump:::actuator
    LEDs:::actuator
    Start:::logic
    Collect:::logic
    MQTT:::mqtt
    RPI:::sensor
    RPI:::controller
    CheckLight:::condition
    LogicRPI:::logic
    CheckWater:::condition
    CheckTDS:::condition

  classDef mqtt fill:#F1C40F,stroke:#B7950B,color:#000
  classDef relay fill:#FAD7A0,stroke:#E59866,color:#000
  classDef actuator fill:#A9DFBF,stroke:#2ECC71,color:#000
  classDef logic fill:#D2B4DE,stroke:#8E44AD,color:#000
  classDef condition fill:#F7DC6F,stroke:#B7950B,color:#000
  classDef sensor fill:#AEDFF7, stroke:#3B84C0, color:#000
  classDef controller fill:#4A90E2, stroke:#1A3C7B, color:#fff, font-weight:bold
  
  linkStyle 0 stroke:#9B59B6,stroke-width:2px %% System Power to Collect,fill:none
  linkStyle 1 stroke:#2ECC71,stroke-dasharray: 4 3,stroke-width:2px %% MQTT to RPI,fill:none
  linkStyle 2 stroke:#2ECC71,stroke-dasharray: 4 3,stroke-width:2px %% TDS to MQTT,fill:none
  linkStyle 3 stroke:#2ECC71,stroke-dasharray: 4 3,stroke-width:2px %% DHT22 to RPI,fill:none
  linkStyle 4 stroke:#2ECC71,stroke-dasharray: 4 3,stroke-width:2px %% DHT11 to RPI, fill:none,fill:none
  linkStyle 5 stroke:#2ECC71,stroke-dasharray: 4 3,stroke-width:2px %% USonic to RPI,fill:none
  linkStyle 6 stroke:#2ECC71,stroke-dasharray: 4 4,stroke-width:2px %% RPI to Relay,fill:none
  linkStyle 7 stroke:#616161,fill:none,stroke-width:2px %% Condition
  linkStyle 8 stroke:#2ECC71,stroke-dasharray: 4 4,stroke-width:2px %% Relay to Actuators,fill:none
  linkStyle 9 stroke:#2ECC71,stroke-dasharray: 4 4,stroke-width:2px %% Relay to Actuators,fill:none
  linkStyle 10 stroke:#2ECC71,stroke-dasharray: 4 4,stroke-width:2px %% Relay to Actuators,fill:none
  linkStyle 11 stroke:#2ECC71,stroke-dasharray: 4 4,stroke-width:2px %% Relay to Actuators,fill:none
  linkStyle 12 stroke:#9B59B6,stroke-width:2px %% RPI to Automation,fill:none
  linkStyle 13 stroke:#9B59B6,stroke-width:2px %% Collect to RPI, fill:none,fill:none
  linkStyle 14 stroke:#9B59B6,stroke-width:2px %% Automation to Condition,fill:none
  linkStyle 15 stroke:#9B59B6,stroke-width:2px %% Automation to Condition,fill:none
  linkStyle 16 stroke:#9B59B6,stroke-width:2px %% Automation to Condition,fill:none
  linkStyle 17 stroke:#616161,fill:none,stroke-width:2px %% Condition
  linkStyle 18 stroke:#616161,fill:none,stroke-width:2px %% Condition
```


### Data Flow Summary

- **Sensor Data**:
  - Environmental data (temperature, humidity, water level) is collected via ESPHome and pushed to HomeAssistant in real-time.
  - TDS sensor data is transmitted over **MQTT** to HomeAssistant for nutrient level monitoring.

- **Device Control**:
  - HomeAssistant issues control signals to the actuator ESP32 via ESPHome to manage:
    - Water flow (main submersible pump)
    - Nutrient dosing (peristaltic pump)
    - pH- dosing (peristaltic pump)
    - Lighting (LED grow lights)

- **Automation Logic**:
  - Resides in HomeAssistant, which responds to sensor inputs to trigger actions (e.g., refill water, dose nutrients, adjust lighting schedules). <br><br><br>


## Technologies and Components Used

- **Raspberry Pi 4**: Hosts HomeAssistant for centralized automation and monitoring.
- **ESP32-C6-DevKitC-1**: Handles sensor data and relay control.
- **ESPHome**: Used for seamless integration of ESP32 nodes with HomeAssistant.
- **MQTT**: For transmitting TDS sensor data (non-ESPHome compatible).
- **Custom 3D-Printed Components**: Mechanical support for mounting and water channels.



### Sensors & Modules

| Component                           | Function                                 | Interface          |
|------------------------------------|------------------------------------------|--------------------|
| **Waveshare DHT22 & DHT11**        | Temperature and humidity monitoring      | ESPHome (ESP32)    |
| **Waveshare Ultrasonic Ranger**    | Water level detection in tank            | ESPHome (ESP32)    |
| **Velleman WPM356 (TDS Sensor)**   | Measures total dissolved solids in water | MQTT (ESP32)       |
| **Velleman WPM400 (Relay Module)** | Controls pumps and lighting              | ESPHome (ESP32)    |


### Actuators

| Device                                    | Purpose                    | Power Source  |
|------------------------------------------|----------------------------|---------------|
| **SEAFLO 16LPM 12V Submersible Pump**    | Main water circulation     | 12V supply    |
| **VMA447 Mini Peristaltic Pumps (2x)**   | Nutrient and pH delivery   | 6V supply     |
| **NelsonGarden LED Plant Lights (2x)**   | Plant growth illumination  | 24V supply    |


### Power Supplies

- **5V DC** – ESP32 devices
- **6V DC** – Peristaltic pumps
- **12V DC** – Submersible water pump
- **24V DC** – LED grow lights

Each power supply is selected based on device voltage and current requirements, routed through the relay module for automated switching.


### Additional Components

- **Water Tank**: 25L capacity for hydroponic reservoir.
- **Nutrient & pH Tanks**: 1L each for solution control.
- **Hosing & Couplings**: Water distribution channels.
- **Cabinet Housing**: Custom-designed, houses entire setup.
- **Water Channels**: Supports vertical growth and drainage.







## Future Improvements

- Integration with cloud services for remote access
- Addition of camera module for plant health monitoring
- Automated refill system for water/nutrient tanks
- Voice assistant integration for hands-free control


## License

This documentation and system design are open for personal and educational use.


# Component Selection Matrix

## Selected Design

**Environmental Parameters**
1. Air Temperature
2. Relative Humidity

**Actuator**
1. Exhaust Fan

**Control Logic**
Temperature and relative humidity are combined to calculate Vapor Pressure Deficit (VPD). Home Assistant uses VPD to determine when to activate or deactivate rack ventilation.

---

## Temperature and Humidity Sensor Candidates

| Product | Price | Interface | Temperature Accuracy | Humidity Accuracy | Notes |
|----------|-------|-----------|---------------------|------------------|-------|
| Adafruit SHT45 | $12.50 | I2C | ±0.1°C (0–75°C) | ±1.0% RH (25–75% RH) | Highest accuracy, ISO 17025 calibration certificate |
| Adafruit SHT31-D | $13.95 | I2C | ±0.3°C | ±2.0% RH | Mature and widely deployed |
| Seeed Grove SHT40 | $5.50 | I2C | ±0.2°C | ±1.8% RH | Lowest cost option |

### Selected Sensor: Adafruit SHT45

Reasons:
- Best temperature accuracy (±0.1°C).
- Best humidity accuracy (±1.0% RH).
- Includes sensor-specific calibration traceability.
- Includes an integrated heater that improves recovery from condensation events and high-humidity operation common in growing environments.
- Native I2C interface works directly with ESPHome.
- Satisfies the project requirement to integrate a non-native sensor into the Home Assistant ecosystem through an open bridge.

---

## ESP32 Bridge Candidates

| Product | Price | Integration | Reliability | Availability |
|----------|-------|------------|-------------|-------------|
| ESP32-DEVKITM-1 | ~$8 | Native ESPHome support | High | High |
| ESP32-S3-DEVKITC-1U-N8R8 | ~$15 | Native ESPHome support | High | Medium |
| ESP32-C3-DEVKITM-1-N4X | ~$8 | Native ESPHome support | Medium | High |

### Selected Bridge
**ESP32-DEVKITM-1**

Reasoning:
- Mature and widely adopted ESP32 platform with extensive ESPHome support.
- More than sufficient compute resources for an I2C temperature and humidity sensor node.
- Lower cost than the ESP32-S3 while avoiding the resource constraints of the ESP32-C3.
- Broad availability through distributors such as DigiKey and Mouser.
- Easily replaceable without requiring changes to Home Assistant entities or automations.
  
---

## Smart Plug Candidates

| Product | Vendor | Price | Protocol | HA Integration | Reliability | Availability |
|----------|--------|-------|----------|---------------|-------------|-------------|
| Shelly Plug US Gen4 | Shelly | ~$25 | Matter over WiFi | Native Matter | High | High |
| Eve Energy Smart Plug | Eve | ~$40 | Matter/Thread | Native Matter | High | Medium |
| TP-Link Tapo TP15 | TP-Link | ~$20 | Matter/WiFi | Native Matter | High | High |

### Selected Smart Plug
**Shelly Plug US Gen4**

Reasoning:
- Supports Matter, aligning directly with Luce Labs' preferred architecture.
- Provides local control without requiring cloud connectivity.
- Native Home Assistant integration through Matter with automatic device discovery.
- Includes energy monitoring, enabling detection of fan failures or abnormal power consumption.
- Can be replaced by any other Matter-compatible smart plug without modifying Home Assistant automations or entity models.
- Supports unattended operation by continuing local communication even during internet outages.
- 
---

## Exhaust Fan Candidates

| Product | Vendor | Price | Voltage | Reliability | Availability |
|----------|--------|-------|---------|-------------|-------------|
| AXIAL 1238 | AC Infinity | ~$39 | 100-120V AC | High | High |
| CLOUDLINE S6 | AC Infinity | ~$100 | 100-120V AC | High | High |
| AeroZesh G6 Inline Fan | VIVOSUN | ~$104 | 110-120V AC | Medium | High |

### Selected Fan
**AC Infinity AXIAL 1238**

Reasoning:
- Supports simple on/off control through a Matter-compatible smart plug, keeping all control logic inside Home Assistant.
- Designed for continuous operation, making it suitable for the 21-day unattended operation requirement.
- Lower cost and lower complexity than smart inline fans with their own controllers and automation systems.
- Easily replaceable with another AC fan without requiring changes to Home Assistant entities or automations.
- Aligns with Luce Labs' open and plug-and-play architecture by avoiding vendor-specific ecosystems and keeping the fan as a simple actuator rather than an intelligent control device.
---

## Final Architecture

```text
Adafruit SHT45 Temperature & Humidity Sensor
            ↓ I2C
ESP32-DEVKITM-1 running ESPHome
            ↓ WiFi
Home Assistant
            ↓ VPD Calculation + Automation Logic
Shelly Plug US Gen4 (Matter)
            ↓ 120V AC
AC Infinity AXIAL 1238 Exhaust Fan
```

### Data Flow

1. The Adafruit SHT45 measures air temperature and relative humidity inside the growing rack.
2. The sensor communicates with the ESP32-DEVKITM-1 over I2C.
3. ESPHome running on the ESP32 exposes:
   - `sensor.rack_temperature`
   - `sensor.rack_humidity`
4. Home Assistant automatically discovers these entities through the ESPHome integration.
5. Home Assistant calculates Vapor Pressure Deficit (VPD) from temperature and humidity measurements.
6. When VPD exceeds configured thresholds, Home Assistant sends commands through Matter to the Shelly Plug US Gen4.
7. The Shelly plug switches power to the AC Infinity AXIAL 1238 exhaust fan.

### Architectural Alignment

- Home Assistant remains the single source of truth for all sensing, correlation, and control logic.
- The non-native SHT45 sensor is bridged into Home Assistant using ESPHome and appears as first-class HA entities.
- All components communicate using open protocols (I2C, WiFi, Matter).
- Any component can be replaced without redesigning the system:
  - The SHT45 can be replaced by another ESPHome-compatible sensor.
  - The Shelly plug can be replaced by any Matter-compatible smart plug.
  - The fan can be replaced by any 120V AC ventilation fan.
- No custom hardware or proprietary cloud services are required.
```
---
## Weakest Buy vs Build Decision

The weakest buy in this design is the ESP32 + SHT45 sensing node.

While the combination provides excellent measurement accuracy and integrates cleanly into Home Assistant through ESPHome, it introduces an additional microcontroller, firmware maintenance, and a WiFi dependency that do not exist with native Matter or Zigbee sensors.

The current approach was selected because it aligns strongly with Luce Labs' "buy before build" philosophy while satisfying the requirement to integrate a non-native sensor into an open architecture.

### When would we build instead?

A custom solution becomes attractive when deploying at larger scale, such as dozens or hundreds of rack slices, where the cost, assembly effort, and maintenance burden of separate ESP32 and sensor modules begin to dominate.

At that point, a custom sensing node could integrate:

- Temperature and humidity sensing on a single PCB
- Native Matter-over-Thread support
- Watchdog and health monitoring circuitry
- Industrial connectors for field serviceability
- Local buffering during network outages
- Standardized mounting and cable management for rack deployment

The custom node would still expose standard Home Assistant entities and preserve the plug-and-play architecture, ensuring that Home Assistant remains the central location for sensing, correlation, and control logic.

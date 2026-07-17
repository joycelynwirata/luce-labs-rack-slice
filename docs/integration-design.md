# Integration and Control Design

## Block Diagram

```text
Adafruit SHT45 Temperature & Humidity Sensor
            │ I2C
            ▼
ESP32-DEVKITM-1 running ESPHome
            │ WiFi
            ▼
Home Assistant
            │
            ├── sensor.rack_temperature
            ├── sensor.rack_humidity
            ├── sensor.rack_vpd
            └── Automation Logic
                    │ Matter over WiFi
                    ▼
Shelly Plug US Gen4
                    │ 120V AC
                    ▼
AC Infinity AXIAL 1238 Exhaust Fan
```

---

## Device Integration

### Temperature and Humidity Sensor

The Adafruit SHT45 communicates with the ESP32-DEVKITM-1 using the I2C protocol.

ESPHome running on the ESP32 reads the sensor and exposes the following entities to Home Assistant:

- `sensor.rack_temperature`
- `sensor.rack_humidity`

Home Assistant automatically discovers these entities through the native ESPHome integration.

---

### Exhaust Fan Control

The Shelly Plug US Gen4 joins Home Assistant using the Matter integration and exposes:

- `switch.rack_fan`
- `sensor.rack_fan_power`
- `sensor.rack_fan_energy`

The smart plug acts purely as an actuator and contains no environmental logic or local automation.

---

## Non-Native Sensor Bridge Design

The SHT45 is not Matter-native and cannot connect directly to Home Assistant.

An ESP32 running ESPHome acts as an open protocol bridge between the sensor and Home Assistant.

The bridge performs the following functions:

- Polls the SHT45 over I2C.
- Publishes temperature and humidity measurements to Home Assistant.
- Supports OTA firmware updates through ESPHome.
- Appears in Home Assistant as native sensor entities.

This approach preserves the plug-and-play architecture because the resulting entities are indistinguishable from native Home Assistant devices.

### Why ESPHome?

| Option | Reason Not Selected |
|--------|---------------------|
| MQTT Discovery | Requires an MQTT broker and additional configuration overhead |
| Matter Bridge | Additional complexity without meaningful benefit for a simple sensor node |
| ESPHome | Native Home Assistant support, OTA updates, and minimal configuration |

---

## Correlation Logic

The system combines temperature and relative humidity measurements to calculate Vapor Pressure Deficit (VPD).

VPD is a more useful metric for plant transpiration and environmental control than temperature or humidity alone.

Control thresholds:

- If VPD rises above **1.2 kPa** for **5 minutes**, the exhaust fan turns ON.
- If VPD falls below **1.0 kPa** for **10 minutes**, the exhaust fan turns OFF.

The gap between the ON and OFF thresholds provides hysteresis and prevents rapid cycling.

The minimum runtime requirement also provides anti-short-cycling protection for the fan.

---

## Failure Modes and Responses

| Failure Mode | Detection Method | Response |
|-------------|-----------------|----------|
| Temperature sensor offline | Entity unavailable | Turn fan ON and generate Home Assistant alert |
| Humidity sensor offline | Entity unavailable | Turn fan ON and generate Home Assistant alert |
| ESP32 offline | ESPHome device unavailable | Turn fan ON and generate Home Assistant alert |
| Fan commanded ON but power draw is 0 W | Shelly energy monitoring | Generate maintenance alert |
| Home Assistant restart | Automation startup trigger | Restore previous actuator state |
| WiFi network loss | Device unavailable | Fan defaults to ON state |

The safe-state philosophy is ventilation-first because excessive humidity presents a greater risk to unattended plant growth than excessive airflow.

---

## Bench Test Plan

Before deployment, the following tests would be performed in order:

1. Verify SHT45 readings against a calibrated reference sensor.
2. Verify ESPHome discovery and entity creation in Home Assistant.
3. Verify VPD calculations using known temperature and humidity values.
4. Verify fan activation at configured VPD thresholds.
5. Verify hysteresis behavior and anti-short-cycling logic.
6. Verify behavior during sensor disconnect events.
7. Verify behavior after Home Assistant restart.
8. Perform a 24-hour soak test to validate stability and recovery behavior.

---

## Architectural Alignment

This design follows Luce Labs' stated architectural direction:

- Home Assistant remains the single source of truth for sensing, correlation, and control logic.
- The non-native sensor is bridged into Home Assistant using open protocols rather than custom APIs.
- Components remain swappable without requiring redesign.
- Control decisions remain centralized within Home Assistant rather than embedded in edge devices.
- The design prioritizes commercially available hardware before considering custom solutions.

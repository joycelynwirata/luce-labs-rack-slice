# ESPHome Sensor Bridge

This ESPHome configuration bridges the non-native Adafruit SHT45 I2C temperature and humidity sensor into Home Assistant.

The ESP32 exposes the following Home Assistant entities:

- `sensor.rack_temperature`
- `sensor.rack_humidity`

This approach was selected over MQTT discovery and Matter bridging due to its native Home Assistant support, OTA firmware updates, and minimal configuration overhead.

The resulting entities are indistinguishable from native Home Assistant devices and preserve the plug-and-play architecture of the rack.

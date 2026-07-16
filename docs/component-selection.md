# Component Selection Matrix

## Selected Design

Sensors:
- Air Temperature
- Relative Humidity

Actuator:
- Exhaust Fan

Control Logic:
Temperature and relative humidity measurements are combined to calculate Vapor Pressure Deficit (VPD). Home Assistant uses VPD to determine when to activate or deactivate rack ventilation.

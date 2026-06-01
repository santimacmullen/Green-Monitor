# Green-Monitor — Backend Setup Guide

This document covers what the backend needs to have ready before the ESP32 firmware can connect and send data.

## Overview

The ESP32 reads 5 sensors and publishes one MQTT message per reading to the topic `sensors/{plant_id}`. Each message carries a UUID that identifies the plant and the specific sensor. Those UUIDs must exist in the backend database before any data is accepted.

---

## Step 1 — Run the MQTT Broker

The ESP32 connects to an MQTT broker (Mosquitto) on port `1883`. Make sure the broker is running and reachable on your local network or a public host.

Note the machine's local IP address (e.g. `192.168.1.x`) — Santi needs it for his `secrets.yaml`.

---

## Step 2 — Register the Plant

Create a new plant in the LeafWise backend. Keep the UUID it returns — this is the `plant_id`.

---

## Step 3 — Register the 5 Sensors

Register one sensor per measurement type, all linked to the plant above. The firmware expects exactly these types:

| Sensor | `type` field | `unit` field |
|--------|-------------|-------------|
| Temperature (DHT22) | `temperature` | `celsius` |
| Humidity (DHT22) | `humidity` | `percent` |
| Light (BH1750) | `light` | `lux` |
| Soil Moisture (ADC) | `soil_moisture` | `percent` |
| Soil pH (ADC) | `soil_ph` | `ph` |

Keep the UUID returned for each sensor.

---

## Step 4 — Share Credentials with Santi

Send Santi the following values so he can fill in `plant-sensor.yaml` and `secrets.yaml`:

```
plant_id:           <UUID from Step 2>
sensor_id_temp:     <UUID for temperature>
sensor_id_humidity: <UUID for humidity>
sensor_id_light:    <UUID for light>
sensor_id_moisture: <UUID for soil moisture>
sensor_id_ph:       <UUID for soil pH>

mqtt_broker:        <IP or hostname of the broker>
mqtt_port:          1883
```

If the broker requires authentication, also share `mqtt_username` and `mqtt_password`.

---

## MQTT Message Contract (reference)

Each message the ESP32 publishes looks like this:

```json
{
  "plant_id": "uuid",
  "sensor_id": "uuid",
  "type": "temperature",
  "value": 23.50,
  "unit": "celsius",
  "recorded_at": "2026-05-16T19:00:00Z",
  "metadata": {
    "device_id": "esp32-plant-sensor-01",
    "firmware_version": "1.0.0",
    "rssi_dbm": "-65"
  }
}
```

- Topic: `sensors/{plant_id}`
- QoS: 1
- Retain: false
- One message per sensor reading, published every 60 seconds

Full contract: https://github.com/lditzel94/plant-monitoring/blob/main/docs/ESP32_MQTT_CONTRACT.md

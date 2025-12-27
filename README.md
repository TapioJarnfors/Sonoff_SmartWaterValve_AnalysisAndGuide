# Sonoff Smart Water Valve Analysis and Guide

[![Wiki Documentation](https://img.shields.io/badge/Documentation-Wiki-blue)](https://github.com/TapioJarnfors/Sonoff_SmartWaterValveAttributeAnalysis/wiki)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

## Overview
This wiki provides following resources for users and developers working with the Sonoff Smart Water Valve and Zigbee2MQTT:
### Event Timeline Analysis 
Event Timeline Analysis with detailed analysis of the event timeline for the Sonoff Smart Valve, focusing on both quantitative and timed irrigation cycles. The analysis is based on packet captures, highlighting key events and their corresponding attributes.
### Custom Attribute Converter Guide
Guide demonstrates how to create an external converter for a Zigbee device with manufacturer-specific attributes, using Wireshark and Zigbee2MQTT logs for development and testing. The custom converter's main job is to extract and parse relevant information from the Zigbee messages received by Zigbee2MQTT (such as attribute reports or read responses), then format and expose that data in a way that Zigbee2MQTT and Home Assistant can use. 

### Key Findings
- **Low Flow Behavior**: Documents valve operation below specified range (0.07 m³/h vs minimum 0.1 m³/h)
- **Packet Analysis**: Includes Wireshark captures and decoded Zigbee clusters
- **Attribute Mapping**: Complete breakdown of custom Sonoff ZCL attributes

## ESP32 MQTT Integration

### Most Used MQTT Library for ESP32

The **[PubSubClient](https://github.com/knolleary/pubsubclient)** library is the most widely used MQTT client library for ESP32 (and Arduino-compatible boards). It has become the de facto standard due to its:

- **Simplicity**: Easy-to-use API for basic MQTT operations
- **Stability**: Mature codebase with years of production use
- **Wide adoption**: Extensive community support and examples
- **Compatibility**: Works across ESP8266, ESP32, and Arduino boards
- **Small footprint**: Minimal memory requirements

#### Installation
```cpp
// Arduino Library Manager: Search for "PubSubClient" by Nick O'Leary
// Or via PlatformIO:
// lib_deps = knolleary/PubSubClient@^2.8
```

#### Basic Example
```cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* mqtt_server = "mqtt.example.com";

WiFiClient espClient;
PubSubClient client(espClient);

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  
  // Wait for WiFi connection with timeout
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi connected");
  } else {
    Serial.println("\nWiFi connection failed");
  }
  
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP32Client")) {
      Serial.println("connected");
      client.subscribe("home/valve/command");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  
  // Publish status periodically (example - use longer intervals in production)
  static unsigned long lastPublish = 0;
  if (millis() - lastPublish > 60000) { // Every 60 seconds
    client.publish("home/valve/status", "online");
    lastPublish = millis();
  }
}
```

### Alternative MQTT Libraries for ESP32

| Library | Pros | Cons | Best For |
|---------|------|------|----------|
| **[PubSubClient](https://github.com/knolleary/pubsubclient)** | Most popular, simple API, stable | Blocking operations, limited message size (default 256 bytes) | General purpose, beginners, most projects |
| **[AsyncMqttClient](https://github.com/marvinroger/async-mqtt-client)** | Fully asynchronous, better performance | More complex, requires ESPAsyncTCP | High-performance applications, concurrent operations |
| **[ESP-MQTT](https://github.com/espressif/esp-mqtt)** (ESP-IDF) | Native Espressif support, full MQTT 3.1.1/5.0 | Requires ESP-IDF framework | Professional projects, advanced features |
| **[Arduino-MQTT](https://github.com/256dpi/arduino-mqtt)** | Lightweight, modern API | Smaller community | Minimal footprint projects |

### Recommendations

**For most ESP32 projects**: Start with **PubSubClient**
- ✅ Proven reliability in production environments
- ✅ Thousands of tutorials and examples available
- ✅ Compatible with Home Assistant, Node-RED, and other MQTT brokers
- ✅ Easy debugging and troubleshooting

**For advanced users**: Consider **ESP-MQTT** (ESP-IDF native library)
- Supports MQTT 5.0 features
- Better integration with ESP-IDF features
- More configuration options
- Professional-grade error handling

**For high-performance needs**: Use **AsyncMqttClient**
- Non-blocking operations
- Better for applications with multiple sensors/actuators
- Handles network issues more gracefully

### Integration with Sonoff Smart Water Valve

While the Sonoff Smart Water Valve uses Zigbee communication natively, you can integrate ESP32 with MQTT to:

1. **Bridge Zigbee to MQTT**: Use ESP32 with Zigbee radio module and MQTT to create a custom gateway
2. **Control via MQTT**: Send commands through Zigbee2MQTT to control the valve
3. **Monitor Status**: Subscribe to valve status topics published by Zigbee2MQTT
4. **Automate Actions**: Create ESP32-based automation rules that respond to valve events

### Resources
- [PubSubClient Documentation](https://pubsubclient.knolleary.net/)
- [ESP32 MQTT Tutorial](https://randomnerdtutorials.com/esp32-mqtt-publish-subscribe-arduino-ide/)
- [Home Assistant MQTT Discovery](https://www.home-assistant.io/docs/mqtt/discovery/)
- [Zigbee2MQTT Integration](https://www.zigbee2mqtt.io/)

## Documentation
📖 Full technical analysis available in the **[project Wiki](https://github.com/TapioJarnfors/Sonoff_SmartWaterValveAttributeAnalysis/wiki)** including:
- Event timelines for different flow rates
- Byte-level attribute documentation
- Packet capture samples

## Usage
```bash
# Clone repository (including wiki)
git clone --recurse-submodules https://github.com/TapioJarnfors/Sonoff_SmartWaterValveAttributeAnalysis.git

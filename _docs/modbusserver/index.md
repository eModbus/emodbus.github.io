---
layout: default
title: ModbusServer
nav_order: 6
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver
has_children: true
---

# ModbusServer
ModbusServer (aka slave) allows you to concentrate on the server functionality - data provision, manipulation etc. -, while the library will take care of the communication part.

You will write callback functions to handle requests for accepted function codes and register those with the ModbusServer. All requests matching one of the registered callbacks will be forwarded to these callbacks. In the callback, you will generate the response that will be returned to the requester.

Any request for a function code without one of your callbacks registered for it will be answered by an `ILLEGAL_FUNCTION_CODE` or `INVALID_SERVER` response automatically.

## Basic use
Here is an example of a Modbus server running on WiFi, that reacts on function codes 0x03:`READ_HOLD_REGISTER` and 0x04:`READ_INPUT_REGISTER` with the same callback function.
```cpp
#include <Arduino.h>
#include "ModbusServerWiFi.h"

char ssid[] = "YOURNETWORKSSID";
char pass[] = "YOURNETWORKPASS";
// Set up a Modbus server
ModbusServerWiFi MBserver;
IPAddress lIP;                      // assigned local IP

const uint8_t MY_SERVER(1);
uint16_t memo[128];                 // Test server memory

// Worker function for serverID=1, function code 0x03 or 0x04
ModbusMessage FC03(ModbusMessage request) {
  uint16_t addr = 0;        // Start address to read
  uint16_t wrds = 0;        // Number of words to read
  ModbusMessage response;

  // Get addr and words from data array. Values are MSB-first, get() will convert to binary
  request.get(2, addr);
  request.get(4, wrds);
  
  // address valid?
  if (!addr || addr > 128) {
    // No. Return error response
    response.setError(request.getServerID(), request.getFunctionCode(), ILLEGAL_DATA_ADDRESS);
    return response;
  }

  // Modbus address is 1..n, memory address 0..n-1
  addr--;

  // Number of words valid?
  if (!wrds || (addr + wrds) > 127) {
    // No. Return error response
    response.setError(request.getServerID(), request.getFunctionCode(), ILLEGAL_DATA_ADDRESS);
    return response;
  }

  // Prepare response
  response.add(request.getServerID(), request.getFunctionCode(), (uint8_t)(wrds * 2));

  // Loop over all words to be sent
  for (uint16_t i = 0; i < wrds; i++) {
    // Add word MSB-first to response buffer
    response.add(memo[addr + i]);
  }

  // Return the data response
  return response;
}

void setup() {
// Init Serial
  Serial.begin(115200);
  while (!Serial) {}
  Serial.println("__ OK __");

  // Register the worker function with the Modbus server
  MBserver.registerWorker(MY_SERVER, READ_HOLD_REGISTER, &FC03);
  
  // Register the worker function again for another FC
  MBserver.registerWorker(MY_SERVER, READ_INPUT_REGISTER, &FC03);
  
  // Connect to WiFi 
  WiFi.begin(ssid, pass);
  delay(200);
  while (WiFi.status(); != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }

  // print local IP address:
  lIP = WiFi.localIP();
  Serial.printf("My IP address: %u.%u.%u.%u\n", lIP[0], lIP[1], lIP[2], lIP[3]);

  // Initialize server memory with consecutive values
  for (uint16_t i = 0; i < 128; ++i) {
    memo[i] = (i * 2) << 8 | ((i * 2) + 1);
  }

  // Start the Modbus TCP server:
  // Port number 502, maximum of 4 clients in parallel, 10 seconds timeout
  MBserver.start(502, 4, 10000);
}

void loop() {
  static uint32_t statusTime = millis();
  const uint32_t statusInterval(10000);

  // We will be idling around here - all is done in subtasks :D
  if (millis() - statusTime > statusInterval) {
    Serial.printf("%d clients running.\n", MBserver.activeClients());
    statusTime = millis();
  }
  delay(100);
}
```

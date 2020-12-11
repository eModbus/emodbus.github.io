---
layout: default
title: ModbusClient
nav_order: 5
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusclient
has_children: true
---

# Modbusclient

To get up and running you will need to put only a few lines in your code. *We will be using the RTU variant here* to show an example.
First you have to include the matching header file:

```cpp
#include "ModbusClientRTU.h"
```

For ModbusClientRTU a Serial interface is required, that connects to your RS485 adaptor. This is given as parameter to the `ModbusClientRTU` constructor. An optional pin can be given to use half-duplex.

```cpp
ModbusClientRTU RS485(Serial2);          // for auto half-duplex
ModbusClientRTU RS485(Serial2, rtsPin);  // use rtsPin to toggle DE/RE in half-duplex
```

Next we will define a callback function for data responses coming in. This example will print out a hexadecimal dump of the response data, using the range iterator:

```cpp
void handleData(ModbusMessage msg, uint32_t token) 
{
  Serial.printf("Response: serverID=%d, FC=%d, Token=%08X, length=%d:\n", msg.getServerID(), msg.getFunctionCode(), token, msg.size());
  for (auto& byte : msg) {
    Serial.printf("%02X ", byte);
  }
  Serial.println("");
}
```

Optionally we can define another callback to be called when an error response was received. The callback will give you an `Error` which can be converted to a human-readable `ModbusError`:

```cpp
void handleError(Error error, uint32_t token) 
{
  // ModbusError wraps the error code and provides a readable error message for it
  ModbusError me(error);
  Serial.printf("Error response: %02X - %s\n", error, (const char *)me);
}
```

Now we will bring everything together in the `setup()` function:

```cpp
void setup() {
// Set up Serial2 connected to Modbus RTU
  Serial2.begin(19200, SERIAL_8N1);

// Set up ModbusClientRTU client.
// - provide onData and onError handler functions
  RS485.onDataHandler(&handleData);
  RS485.onErrorHandler(&handleError);

// Start ModbusClientRTU background task
  RS485.begin();
}
```

You can start making requests now. In the code below a "read holding register" request is sent to server id `1`, requesting one data word at address 10. Please disregard the `token` value of 0x12345678 for now, it will be explained further below:

```cpp
Error err = RS485.addRequest(0x12345678, 1, READ_HOLD_REGISTER, 10, 1);
  if (err!=SUCCESS) {
    ModbusError e(err);
    Serial.printf("Error creating request: %02X - %s\n", err, (const char *)e);
  }
```

This method enqueues your request and returns immediately. If anything was wrong with your parameters or the queue was full, the request will not be created. You will be given an error return value to find out what went wrong.
In case everything was okay, the request is created and put into the background queue to be processed. Upon response, your `onData` or `onError` callbacks will be called.

---
layout: default
title: ModbusServer TCP Async
nav_order: 3
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver-tcp-async
parent: ModbusServer TCP
grand_parent: ModbusServer
---

# ModbusServerTCPasync
To use the WiFi version, you will have to use
```cpp
#include "ModbusServerTCPasync.h"
...
ModbusServerTCPasync myServer;
...
```
in your source file. You will have to add `#include <WiFi.h>`, yourself and have `AsyncTCP.h` as dependency installed on your system.

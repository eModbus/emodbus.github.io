---
layout: default
title: ModbusServer TCP WiFi
nav_order: 2
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver-tcp-wifi
parent: ModbusServer TCP
grand_parent: ModbusServer
---

# ModbusServerWiFi

To use the WiFi version, you will have to use
```
#include "ModbusServerWiFi.h"
...
ModbusServerWiFi myServer;
...
```
in your source file. You will not have to add ``#include <WiFi.h>``, as this is done in the library. It does not do any harm if you do, though.

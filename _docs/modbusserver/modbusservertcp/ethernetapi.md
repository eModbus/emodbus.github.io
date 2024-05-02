---
layout: default
title: ModbusServer TCP Ethernet
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver-tcp-ethernet
parent: ModbusServer TCP
grand_parent: ModbusServer
---

# ModbusServerTCP Ethernet

Inconsistencies between the WiFi and Ethernet implementations in the core libraries required a split of ModbusServerTCP into the `ModbusServerEthernet` and `ModbusServerWiFi` variants.

Both are sharing the majority of methods etc., but need to be included with different file names and have different object names as well.

All relevant calls after initialization of the respective ModbusServer object are identical.

```cpp
#include "ModbusServerEthernet.h"
...
ModbusServerEthernet myServer;
...
```
or, for ``ETH.h`` based boards,
```cpp
#include "ModbusServerETH.h"
...
ModbusServerEthernet myServer;
...
```
in your source file. You will not have to add `#include <Ethernet.h>` or ``#include <ETH.h>``, as this is done in the library. It does not do any harm if you do, though.

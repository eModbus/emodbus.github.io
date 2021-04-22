---
layout: default
title: ModbusBridge
nav_order: 7
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusbridge
has_children: true
---

# ModbusBridge
A Modbus "bridge" (or gateway) is basically a Modbus server forwarding requests to other servers, that are in a different Modbus. 
The bridge will control and translate requests from a TCP Modbus into a RTU Modbus and vice versa. 

Some typical applications:
- TCP bridge gives access to all servers from a connected RTU Modbus
- RTU bridge brings a TCP server into a RS485 Modbus
- bridge collects data from several (TCP and RTU) Modbuses under a single interface

As the bridge is based on a `ModbusServer`, it also can serve locally in addition to the external servers connected.

## Basic Use
First you will have to include the matching header file for the type of bridge you are going to set up (talking to Ethernet in this case):
```cpp
#include "ModbusBridgeEthernet.h"
```
Next the bridge needs to be defined:
```cpp
ModbusBridgeEthernet MBbridge;
```
We will need at least one `ModbusClient` for the bridge to contact external servers:
```cpp
#include "ModbusClientRTU.h"
```
... and set that up:
```cpp
ModbusClientRTU MB(Serial1);
```
Then, most probably in `setup()`, the client is started and the bridge needs to be configured:
```cpp
MB.setTimeout(2000);
MB.begin();
...
// ServerID 4: RTU Server with remote serverID 1, accessed through RTU client MB - FC 03 accepted only
MBbridge.attachServer(4, 1, READ_HOLD_REGISTER, &MB);
// Add FC 04 to it
MBbridge.addFunctionCode(4, READ_INPUT_REGISTER);

// ServerID 5: RTU Server with remote serverID 4, accessed through RTU client MB - all FCs accepted
MBbridge.attachServer(5, 4, ANY_FUNCTION_CODE, &MB);
// Remove FC 04 from it
MBbridge.denyFunctionCode(5, READ_INPUT_REGISTER);
```
And finally the bridge is started:
```cpp
MBbridge.start(port, 4, 600);
```
That's it!

From now on all requests for serverID 4, function codes 3 and 4 are forwarded to the RTU-based server 1, its responses are given back. Same applies to server ID 5 and all possible function codes - with the exception of FC 4.

All else requests will be answered by the bridge with either `ILLEGAL_FUNCTION` or `INVALID_SERVER`.

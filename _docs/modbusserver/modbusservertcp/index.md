---
layout: default
title: ModbusServer TCP
nav_order: 2
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver-tcp
parent: ModbusServer
has_children: true
---

# ModbusServerTCP

The TCP server can be setup using different implementations:
- Ethernet / ETH
- WiFi
- Async TCP

This makes the server very versatile and usable in many different hardware combinations: ESP32 with built-in WiFi, a W5500 or other board.
Boards using the W5500 range of adapters are supported by the ``...Ethernet.h`` components.
Other boards with integrated Ethernet (like the WT32-ETH01) that will require the ``ETH.h`` definitions are supported by the ``...ETH.h`` files.

The WiFi server does not rely on external libraries. If using Ethernet or AsyncTCP, please make sure to have to dependencies installed for your system. When using PlatformIO, dependency management is (mostly) done for you.

## ModbusServerTCP API
### Constructor
A ModbusServerTCP is defined by
```
ModbusServerWiFi myServer;
```
or
```
ModbusServerEthernet myServer;
```
or
```
ModbusServerETH myServer;
```
or
```
ModbusServerTCPasync myServer;
```
respectively.

### ``bool start(uint16_t port, uint8_t maxClients, uint32_t timeout);``
This call must be issued initially to start the server task and start listening for incoming requests.
The arguments to this call are:
- ``port``: the TCP port number the server is listening on. The standard Modbus TCP port is 502, but you may choose another one if your application is requiring it.
- ``maxClients``: the maximum number of Modbus clients that can be served concurrently. This will help limit the load put upon the server. While the ``maxClients`` number of clients is connected, all further connection attempts will be refused.
- ``timeout``: closely related to the previous, this parameter tells the server to close a connection as soon as the ``timeout`` time has passed without another request from the connected client. Keeping a connection open will reduce the response times, but may lock out other clients.

### ``bool stop();``
This call will close all connections and stop the server, terminating the background task.

### ``uint16_t activeClients();``
``activeClients()`` will return the number of currently open connections.

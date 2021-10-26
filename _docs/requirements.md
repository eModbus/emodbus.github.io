---
layout: default
title: Requirements
nav_order: 2
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /requirements
---

# Requirements

The library was developed for and on ESP32 MCUs in the Arduino core development environment. In principle it should run in any environment providing these resources:

- Arduino core functions - Serial interfaces, millis(), delay() etc.
- FreeRTOS' xTask task functions. Note that there are numerous MCUs running FreeRTOS!  
The async versions are also usable on ESP8266
- C++ standard library components
    - std::queue
    - std::vector
    - std::list
    - std::map
    - std::mutex
    - std::lock_guard
    - std::forward
    - std::move
    - std::function
    - std::bind
    - std::placeholder

For Modbus RTU you will need a RS485-to-Serial adaptor. Commonly used are the RS485MAX adaptors or the XY-017 adaptors with automatic half duplex control. Be sure to check compatibility of voltage levels.

Modbus TCP will require either an Ethernet module like the WizNet W5xxx series connected to the SPI interface or an (internal or external) WiFi adaptor. The client library is by the way using the functions defined in Client.h only, whereas the server and bridge library is requiring either Ethernet.h or Wifi.h internally. Async versions rely on the AsyncTCP library.

The Linux client does provide ``Client`` and ``IPAddress`` classes to supply the Arduino-style requirements, but is restricted to TCP.

## RS-485 hardware

This library is independed of the actual communication standard used by the hardware. Modbus typically uses RS-485 for it's RTU implementation. This lib implements RTU half-duplex mode. To use RTU with an ESP32 you need to have an UART to RS-485 converter. The library provides a function to toggle the direction pin (/RE - DE) in case your adapter doesn't have auto-direction control.

A word of caution when implementing RS-485. Some converter chips do no have bus line balancing so it relies on external components. This can be easily achieved with resistors though. See this discussion for an example implementation: [Example application with hardware](https://github.com/eModbus/eModbus/discussions/112)

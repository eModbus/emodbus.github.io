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

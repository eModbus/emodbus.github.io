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

### Important note for using the RTU bridge, server and client
Due to an implementation decision done in the ESP32 Arduino core code,
the correct time to detect a bus communications gap of a few µs (as is used with Modbus communications to state end-of-message) is not effective, as
the core FIFO handling takes much longer than that.
Our implementation by default takes the suggested modification of the core as given (described below).

Workaround: in `RTUutils.cpp`uncomment the following line to wait for 16ms(!) for the handling to finish:
```
      // if (micros() - MR_lastMicros >= 16000) {
```

Alternate solution: is to modify the uartEnableInterrupt() function in
the core implementation file `esp32-hal-uart.c`, to have the line
```
uart->dev->conf1.rxfifo_full_thrhd = 1; // 112;
```
This will change the number of bytes received to trigger the copy interrupt
from 112 (as is implemented in the core) to 1, effectively firing the interrupt
for any single byte.

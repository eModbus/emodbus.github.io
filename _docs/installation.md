---
layout: default
title: Installation
nav_order: 3
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /installation
---

# Installation

We recommend to use the PlatformIO IDE. There, in your project's platformio.ini file, add the Github URL of this library to your `lib_deps = entry` in the `[env:...]` section of that file:

```ini
[platformio]

# some settings

[env:your_project_target]

# some settings

lib_deps = 
  ModbusClient=https://github.com/ESP32ModbusUnified/ModbusUnified.git
```

If you are using the Arduino IDE, you will want to copy the library folder into your library directory. In Windows this will be `C:\Users\<user_name>\Documents\Arduino\libraries` normally. For Arduino IDE, you'll have to install the dependencies manually. These are:

  - AsyncTCP (https://github.com/me-no-dev/AsyncTCP)
  - Ethernet-compatible library (example: https://github.com/maxgerhardt/Ethernet)

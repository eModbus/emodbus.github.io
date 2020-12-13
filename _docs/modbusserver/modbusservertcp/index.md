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
- Ethernet
- WiFi
- Async TCP

This makes the server very versatile and usable in many different hardware combinations: ESP32 with built-in WiFi, a W5500 or other board supported by ESP-IDF: [link to ESP-IDF descriptions](https://github.com/espressif/esp-idf/blob/master/examples/ethernet/basic/README.md).

The WiFi server does not rely on external libraries. If using Ethernet or AsyncTCP, please make sure to have to dependencies installed for your system. When using PlatformIO, dependency management is done for you.

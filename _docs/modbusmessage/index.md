---
layout: default
title: ModbusMessage
nav_order: 4
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusmessage
has_children: true
---

# ModbusMessage

Basically all data going back and forth inside the library has the form of a `ModbusMessage` object.
This object will contain all the bytes of a Modbus standard message - request or response - but without everything that is depending on the interface used (RTU or TCP).
A `ModbusMessage` internally is a `std::vector<uint8_t>`, and in multiple aspects can be used as one.


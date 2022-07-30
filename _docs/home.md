---
layout: default
title: Home
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /
---

# eModbus

*valid for Release 1.6*

This is a library to provide Modbus client (formerly known as master), server (formerly slave) and bridge/gateway functionalities for  Modbus RTU, ASCII and TCP protocols.

Modbus communication is done in separate tasks, so Modbus requests and responses are non-blocking. Callbacks are provided to prepare or receive the responses asynchronously.

There is a synchronous interface available as well where requests are waiting for their responses to arrive.

Key features:
 - for use in the Arduino framework
 - non blocking / asynchronous API
 - alternative synchronous API
 - server, client and bridge modes
 - TCP (Ethernet, WiFi and Async) and RTU and ASCII (serial) interfaces
 - designed for ESP32, various interfaces supported; async versions run also on ESP8266. Client code for Linux available.
 - all common and user-defined Modbus standard function codes

This has been developed by enthusiasts. While we do our utmost best to make robust software, do not expect any bullet-proof, industry deployable, guaranteed software. See the license to learn about liabilities etc.

We do welcome any ideas, suggestions, bug reports or questions. Please use the "Github issues" to report bugs or request new features, and the "Discussions" area to chat along!

---
layout: default
title: ModbusClient TCP async API
nav_order: 4
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusclient-tcp-async-api
parent: ModbusClient
---

# ModbusClientTCPasync API elements
You will have to include the following line in your code to make the `ModbusClientTCPasync` API available:

```cpp
#include "ModbusclientTCPasync.h"
```

## `ModbusClientTCPasync(IPAddress host, uint16_t port)` and<br> `ModbusClientTCPasync(IPAddress host, uint16_t port, uint16_t queueLimit)`
The asynchronous TCP version takes 2 or 3 arguments: the target host IP address and port number of the Modbus server and an optional queue size limit (defaults to 100). The async version will connect to exactly one Modbus server.

## `void setTimeout(uint32_t timeout)`
Similar to the ModbusClientRTU and ModbusClientTCP timeouts, you may specify a time in milliseconds that will determine if a `TIMEOUT` error occurred. The worker task will wait the specified time without data arriving to then state a timeout and return the error response for it.

### In detail
Every 500msec, only the oldest request is checked for a possible timeout. 
A newer request will only time out if a previous has been processed (answer is received or has hit timeout). 
For example: 3 requests are made at the same time. 
The first times out after 2000msec, the second after 2500msec and the last at 3000msec.

## `void connect()` and `void disconnect()`
The library connects automatically upon making the first request. However, you can also connect manually.
When making requests, the requests are put in a queue and the queue is processed once connected. There is however a delay of 500msec between the moment the connection is established and the first request is sent to the server. All the following requests are send immediately.

You can avoid the 500msec delay by connecting manually.

Disconnecting is also automatic (see `void setIdleTimeout(uint32_t timeout)`). Likewise, you can disconnect manually.

## ``void connect(IPAddress host)`` and <br> ``void connect(IPAddress host, uint16_t port)``
Another flavour of the ``connect()`` call, allowing you to address another Modbus server host. Any existing connection is closed before connecting to the given host. The internal default host is also switched to the new one, so any subsequent ``connect()``, ``disconnect()`` etc. will be applied to the new host.

The port may be omitted and defaults to 502 (standard Modbus port) unless you specify another one.

## `void setIdleTimeout(uint32_t timeout)`
Sets the time after which the client closes the TCP connection to the server.
The async version tries to keep the connection to the server open. Upon the first request, a connection is made to the server and is kept open. 
If no data has been received from the server after the idle timeout, the client will close the connection.

Mind that the client will only reset the idle timeout timer upon data reception and not when sending.

## `void setMaxInflightRequests(uint32_t maxInflightRequests)`
Sets the maximum number of messages that are sent to the server at once. The async modbus client sends all the requests to the server without waiting for an earlier request to receive a response. This is depending on the receiving server's ability to cope with multiple requests and can result in message loss if the server has a limited queue only. Setting this number to 1 mimics a sync client.

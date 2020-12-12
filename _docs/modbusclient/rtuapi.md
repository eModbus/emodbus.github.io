---
layout: default
title: ModbusClient RTU API
nav_order: 2
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusclient-rtu-api
parent: ModbusClient
---

# ModbusClientRTU API elements
You will have to have the following include line in your code to make the `ModbusClientRTU` API available:

```cpp
#include "ModbusClientRTU.h"
```

## `ModbusClientRTU(HardwareSerial& serial)`,<br> `ModbusClientRTU(HardwareSerial& serial, int8_t rtsPin)` and<br> `ModbusClientRTU(HardwareSerial& serial, int8_t rtsPin, uint16_t queueLimit)`
These are the constructor variants for an instance of the `ModbusClientRTU` type. The parameters are:
- `serial`: a reference to a Serial interface the Modbus is conncted to (mostly by a RS485 adaptor). This Serial interface must be configured to match the baud rate, data and stop bits and parity of the Modbus.
- `rtsPin`: some RS485 adaptors have "DE/RE" lines to control the half duplex communication. When writing to the bus, the lines have to be set accordingly. DE and RE usually have opposite logic levels, so that they can be connected to a single GPIO that is set to HIGH for writing and LOW for reading. This will be done by the library, if a GPIO pin number is given for `rtsPin`. Leave this parameter out or use `-1` as value if you do not need this GPIO (usually with RS485 adaptors doing auto half duplex themselves).
- `queueLimit`: this specifies the number of requests that may be placed on the worker task's queue. If the queue has reached this limit, the next `addRequest` call will return a `REQUEST_QUEUE_FULL` error. The default value built in is 100.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
While the queue holds pointers to the requests only, the requests need memory as well. If you choose a `queueLimit` too large, you may encounter "out of memory" conditions!

## `void setTimeout(uint32_t TOV)`
This call lets you define the time for stating a response timeout. `TOV` is defined in **milliseconds**. When the worker task is waiting for a response from a server, and the specified number of milliseconds has passed without data arriving, it will return a `TIMEOUT` error response.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
This timeout is blocking the worker task. No other requests will be made while waiting for a response. Furthermore, the worker will retry the failing request two more times. So the `TOV` value in effect can block the worker up to three times the time you specified, if a server is dead. 
Too short timeout values on the other hand may miss slow servers' responses, so choose your value with care...

## `uint16_t RTUutils::calcCRC16(uint8_t *data, uint16_t len)` and<br> `uint16_t RTUutils::calcCRC16(ModbusMessage msg)`
This is a convenient method to calculate a CRC16 value for a given block of bytes. `*data` points to this block, `len` gives the number of bytes to consider.
The call will return the 16-bit CRC16 value.

## `bool RTUutils::validCRC(const uint8_t *data, uint16_t len)` and<br> `bool RTUutils::validCRC(ModbusMessage msg)`
If you got a buffer holding a Modbus message including a CRC, `validCRC()` will tell you if it is correct - by calculating the CRC16 again and comparing it to the last two bytes of the message buffer.

## `bool RTUutils::validCRC(const uint8_t *data, uint16_t len, uint16_t CRC)` and<br> `bool RTUutils::validCRC(ModbusMessage msg, uint16_t CRC)`
`validCRC()` has a second variant, where you can provide a pre-calculated (or otherwise obtained) CRC16 to be compared to that of the given message buffer.

## `void RTUutils::addCRC(ModbusMessage& raw)`
If you have a `ModbusMessage` (see above), you can calculate a valid CRC16 and have it pushed o its end with `addCRC()`.

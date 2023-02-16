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

## Important when using ``HardwareSerial``
Modbus RTU has been  modified to accept any ``Stream``-based object for reading and writing serial data.
Note that this requires a version of the ``arduino-esp32`` core of 2.0.x and higher.
Due to the timing requirements of Modbus, a ``HardwareSerial`` connection needs to be configured properly to work with eModbus.

1. Configure a buffer size suited for your messages. With higher baud rates the standard 128 byte UART buffer needs to be copied from the FIFO multiple times if you are processing requests or responses longer than that.
The UART buffer needs to be large enough to hold a complete message - else timing errors will happen.<br/>
A reasonable size for regular Modbus RTU is 260, Modbus ASCII rather will need 520 bytes.<br/>
The size must be set **before the ``HardwareSerial::begin()`` call** to be effective.<br/>
The call is like (``Serial1`` taken as an example)<br/>
``Serial1.setRxBufferSize(260);``

2. Now call your ``Serial``'s ``begin()``.
3. **This is most important of all!**<br/>
Set the FIFO full threshold to just 1 byte. This allows eModbus to take care of the timing.
If you omit this step, you may encounter timeots, broken messages etc.<br/>
The threshold is set with<br/>
``Serial1.setRxFIFOFull(1);``

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
Also note that the ``ModbusClientRTU::begin()`` call now needs to be given the baud rate as first parameter.
This is used to calculate the necessary interval times between messages on the RTU bus. Incorrect values may lead to messages being missed.

## `ModbusClientRTU(Stream& serial)`,<br> `ModbusClientRTU(Stream& serial, int8_t rtsPin)` and<br> `ModbusClientRTU(Stream& serial, int8_t rtsPin, uint16_t queueLimit)`
These are the constructor variants for an instance of the `ModbusClientRTU` type. The parameters are:
- `serial`: a reference to a Serial interface the Modbus is conncted to (mostly by a RS485 adaptor). This Serial interface must be configured to match the baud rate, data and stop bits and parity of the Modbus.
- `rtsPin`: some RS485 adaptors have "DE/RE" lines to control the half duplex communication. When writing to the bus, the lines have to be set accordingly. DE and RE usually have opposite logic levels, so that they can be connected to a single GPIO that is set to HIGH for writing and LOW for reading. This will be done by the library, if a GPIO pin number is given for `rtsPin`. Leave this parameter out or use `-1` as value if you do not need this GPIO (usually with RS485 adaptors doing auto half duplex themselves).
- `queueLimit`: this specifies the number of requests that may be placed on the worker task's queue. If the queue has reached this limit, the next `addRequest` call will return a `REQUEST_QUEUE_FULL` error. The default value built in is 100.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
While the queue holds pointers to the requests only, the requests need memory as well. If you choose a `queueLimit` too large, you may encounter "out of memory" conditions!

## `ModbusClientRTU(Stream& serial, RTScallback func)` and<br> `ModbusClientRTU(Stream& serial, RTScallback func, uint16_t queueLimit)`
These are the alternative constructor variants for an instance of the `ModbusClientRTU` type. The parameters are:
- `serial`: a reference to a Serial interface the Modbus is conncted to (mostly by a RS485 adaptor). This Serial interface must be configured to match the baud rate, data and stop bits and parity of the Modbus.
- `func`: this must be a user-defined callback function of type ``void func(bool level);``. This function is called every time the RS485 adaptor's "DE/RE" line has to be toggled. The required logic level is given as the only parameter to the function. This is relevant if your adaptor will need a special treatment to set these levels (being behind a port extender or such).
- `queueLimit`: this specifies the number of requests that may be placed on the worker task's queue. If the queue has reached this limit, the next `addRequest` call will return a `REQUEST_QUEUE_FULL` error. The default value built in is 100.

## `void begin(uint32_t baudrate)` and <br> `void begin(uint32_t baudrate, int coreID)`
This is the most important call to get a ModbusClient instance to work. It will open the request queue and start the background worker task to process the queued requests.

The second form of `begin()` allows you to choose a CPU core for the worker task to run (only on multi-core systems like the ESP32). This is **highly recommended** in particular for the `ModbusClientRTU` client, as the handling of the RS485 Modbus is a time-critical and will profit from having its own core to run.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
The worker task is running forever or until the ModbusClient instance is killed that started it. The destructor will take care of all requests still on the queue and remove those, then will stop the running worker task.

## `void setTimeout(uint32_t TOV)`
This call lets you define the time for stating a response timeout. `TOV` is defined in **milliseconds**. When the worker task is waiting for a response from a server, and the specified number of milliseconds has passed without data arriving, it will return a `TIMEOUT` error response.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
This timeout is blocking the worker task. No other requests will be made while waiting for a response.

## `uint16_t RTUutils::calcCRC16(uint8_t *data, uint16_t len)` and<br> `uint16_t RTUutils::calcCRC16(ModbusMessage msg)`
This is a convenient method to calculate a CRC16 value for a given block of bytes. `*data` points to this block, `len` gives the number of bytes to consider.
The call will return the 16-bit CRC16 value.

## `bool RTUutils::validCRC(const uint8_t *data, uint16_t len)` and<br> `bool RTUutils::validCRC(ModbusMessage msg)`
If you got a buffer holding a Modbus message including a CRC, `validCRC()` will tell you if it is correct - by calculating the CRC16 again and comparing it to the last two bytes of the message buffer.

## `bool RTUutils::validCRC(const uint8_t *data, uint16_t len, uint16_t CRC)` and<br> `bool RTUutils::validCRC(ModbusMessage msg, uint16_t CRC)`
`validCRC()` has a second variant, where you can provide a pre-calculated (or otherwise obtained) CRC16 to be compared to that of the given message buffer.

## `void RTUutils::addCRC(ModbusMessage& raw)`
If you have a `ModbusMessage` (see above), you can calculate a valid CRC16 and have it pushed o its end with `addCRC()`.

## `void useModbusASCII()` and `void useModbusASCII(unsigned long timeout)`
If you are going to use the Modbus ASCII protocol, you may switch the RTU client to ASCII mode by these calls. In ASCII mode all messages are encoded as a sequence of human-readable ASCII characters.
The first character always is a colon ``:``, the last two characters of a message always are  ``\r\n`` (CRLF), as the Modbus specs require it.
All message bytes are sent as their hexadecimal representations, followed by a LRC checksum.

**Note:** everything else is identical to Modbus RTU, so your code will remain the same with the exception of this very call.

The optional `timeout`parameter allows to alter the standard 1s timeout the specs describe. This may help speeding up communications if all your servers are supporting the shorter timings.

## `void useModbusRTU()`
This call will switch off a previously set ASCII mode and return to RTU mode. 

**Note:** this will **NOT** reinstate the previously given RTU timeout! You may to have to set it again with ``setTimeout(unsigned long timeout)`` instead.

## `bool isModbusASCII()`
This call is to report back if the ASCII mode currently is active (`true`) or RTU mode is selected (`false`).

## `void skipLeading0x00(bool onOff = true);`
Some RTU bus setups have unresolvable issues with sporadic 'ghost' 0x00 bytes created within the electronics that have no logical representation at all.
The ``skipLeading0x00()`` call allows to have these bytes droped internally, if these are received in front of a proper message.

**Note:** this is a work-around only to cure a symptom and does not fix the issue. It always pays to try to fix the root cause!

## `Error addBroadcastMessage(const uint8_t *data, uint8_t len)`
This call will add a broadcast request to the queue. 
`data` may contain whatever you will find appropriate.
The only thing the library will add is a leading 0x00 byte to signal a broadcast request and the CRC.

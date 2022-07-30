---
layout: default
title: ModbusServer RTU API
nav_order: 3
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver-rtu-api
parent: ModbusServer
---

# ModbusServerRTU

There are a few differences to the TCP-based ModbusServers: 
- ModbusServerRTU only has a single background task to listen for responses on the Modbus.
- as timing is critical on the Modbus, the background task has a higher priority than others
- a request received for another server ID not covered by this server will be ignored (rather than answered with an `ILLEGAL_SERVER_ID` error as with TCP)

## `ModbusServerRTU(HardwareSerial& serial, uint32_t timeout)` and<br> `ModbusServerRTU(HardwareSerial& serial, uint32_t timeout, int rtsPin)`
The first parameter, `serial` is mandatory, as it gives the serial interface used to connect to the RTU Modbus to the server.
`timeout` is less important as it is for TCP. It defines after what time of inactivity the server should loop around and re-initialize some working data.
A value of `20000` (20 seconds) is reasonable.
The third (optional) parameter `rtsPin` is the same as for the RTU Modbus client described above - if you are using a RS485 adaptor requiring a DE/RE line to be maintained, `rtsPin` should be the GPIO number of the wire to that DE/RE line. The library will take care of toggling the pin.

## `ModbusServerRTU(HardwareSerial& serial, uint32_t timeout, RTScallback func)`
The first parameter, `serial` is mandatory, as it gives the serial interface used to connect to the RTU Modbus to the server.
`timeout` is less important as it is for TCP. It defines after what time of inactivity the server should loop around and re-initialize some working data.
A value of `20000` (20 seconds) is reasonable.
- `func`: this must be a user-defined callback function of type ``void func(bool level);``. This function is called every time the RS485 adaptor's "DE/RE" line has to be toggled. The required logic level is given as the only parameter to the function. This is relevant if your adaptor will need a special treatment to set these levels (being behind a port extender or such).

## `bool start()`,<br> `bool start(int coreID)` and <br>``bool start(int coreID, uint32_t interval)``
With `start()` the server will create its background task and start listening to the Modbus. 
The optional parameter `coreID` may be used to have that background task run on the named core for multi-core MCUs. Default for ``coreID`` is -1, in which case the system will pick the core for its own rules.
The third form allows to specify the minimum waiting time after a message has been received to state end-of-message. 
This helps relaxing the tight timings the Modbus RTU standard requires and may help with slower servers, but strictly speaking is a violation of the Modbus RTU standard.

## `bool stop()`
The server background process can be stopped and the task be deleted with the `stop()` call. You may start it again with another `start()` afterwards.
In fact a `start()` to an already running server will stop and then restart it.

## `void useModbusASCII()` and `void useModbusASCII(unsigned long timeout)`
If you are going to use the Modbus ASCII protocol, you may switch the RTU server to ASCII mode by these calls. In ASCII mode all messages are encoded as a sequence of human-readable ASCII characters.
The first character always is a colon ``:``, the last two characters of a message always are  ``\r\n`` (CRLF), as the Modbus specs require it.
All message bytes are sent as their hexadecimal representations, followed by a LRC checksum.

**Note:** everything else is identical to Modbus RTU, so your code will remain the same with the exception of this very call.

The optional `timeout`parameter allows to alter the standard 1s timeout the specs describe. This may help speeding up communications if all your servers are supporting the shorter timings.

## `void useModbusRTU()`
This call will switch off a previously set ASCII mode and return to RTU mode. 

**Note:** this will **NOT** reinstate the previously given RTU timeout! You may to have to set it again with ``setTimeout(unsigned long timeout)`` instead.

## `bool isModbusASCII()`
This call is to report back if the ASCII mode currently is active (`true`) or RTU mode is selected (`false`).

## `void skipLeading0x00(bool onOff = true)`
Some RTU bus setups have unresolvable issues with sporadic 'ghost' 0x00 bytes created within the electronics that have no logical representation at all.
The ``skipLeading0x00()`` call allows to have these bytes droped internally, if these are received in front of a proper message.

**Note:** this is a work-around only to cure a symptom and does not fix the issue. It always pays to try to fix the root cause!

## `void registerBroadcastWorker(MSRlistener worker)`
As a speciality for Modbus RTU, a non-standard 'broadcast' request is supported by a number of devices. This request will start with the normally illegal server ID 0. Any server accepting broadcast requests shall read it, but never send a response on it.

The `registerBroadcastWorker` call is used to register a special server callback for broadcast requests. 
The callback has a function signature of type `MSRlistener`: `void worker(ModbusMessage msg)`. It will be given the broadcast request as `msg`, but may not return a thing.

## `void registerSniffer(MSRlistener worker)`
Even more special, the 'Sniffer' callback is used to examine any message that is sent on the RTU bus, regardless of request or response.
The callback's signature is the same `MSRlistener` type as for the broadcast callback.

Using a server as a sniffer will disable all previously defined function code callbacks for that server, as no distinction is made any more on server IDs or function codes. Each and any message is given to the callback to be processed as you like.

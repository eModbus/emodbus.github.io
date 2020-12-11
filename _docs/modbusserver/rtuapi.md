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
- a request received for another server ID not covered by this server will be ignored (rather than answered with an ``ILLEGAL_SERVER_ID`` error as with TCP)

## ``ModbusServerRTU(HardwareSerial& serial, uint32_t timeout)`` and<br> ``ModbusServerRTU(HardwareSerial& serial, uint32_t timeout, int rtsPin)``
The first parameter, ``serial`` is mandatory, as it gives the serial interface used to connect to the RTU Modbus to the server.
``timeout`` is less important as it is for TCP. It defines after what time of inactivity the server should loop around and re-initialize some working data.
A value of ``20000`` (20 seconds) is reasonable.
The third (optional) parameter ``rtsPin`` is the same as for the RTU Modbus client described above - if you are using a RS485 adaptor requiring a DE/RE line to be maintained, ``rtsPin`` should be the GPIO number of the wire to that DE/RE line. The linbrary will take care of toggling the pin.

  // start: create task with RTU server to accept requests
## ``bool start()`` and<br> ``bool start(int coreID)``
With ``start()`` the server will create its background task and start listening to the Modbus. 
The optional parameter ``coreID`` may be used to have that background task run on the named core for multi-core MCUs.

## ``bool stop()``
The server background process can be stopped and the task be deleted with the ``stop()`` call. You may start it again with another ``start()`` afterwards.
In fact a ``start()`` to an already running server will stop and then restart it.
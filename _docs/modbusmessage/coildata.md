---
layout: default
title: CoilData type
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusmessage-coildata
parent: ModbusMessage
---
## The ``CoilData`` type

Part of the Modbus standard is the coils data type, that also is used for the digital inputs function codes.
This data type basically is a set of bits that can have values of ``1`` and ``0`` only.
These values commonly are interpreted like ``1`` = ``ON`` = ``true`` or ``0`` = ``OFF`` = ``false``.

The C++ datatype ``bool`` is almost identical to a Modbus ``coil``, so it is obvious to use the one for the other.

Unfortunately the Modbus specification and the C++ data type have some deficits if it comes to use ``bool`` as coil:
- ``bool`` is no basic storage unit in C++, but always is part of at least a ``uint8_t`` or ``byte`` that can hold 8 ``bool`` bits, but not address these individually.
- Therefore there is no such thing as an array of ``bool`` in C++, that would be like an array of ``coils`` in the Modbus world.
- The order Modbus requires the coil data to be delivered in messages is inverse to the order normally used in C++.

To cope with these shortcomings the ``CoilData`` type was added to **eModbbus**.

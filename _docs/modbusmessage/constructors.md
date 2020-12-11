---
layout: default
title: ModbusMessage constructors
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusmessage-constructors
parent: ModbusMessage
---

# ModbusMessage constructors

`ModbusMessage` is created with one of several constructors. 
The very first is the basic one:

## `ModbusMessage()` or `ModbusMessage(uint16_t dataLen)`
These will create an empty `ModbusMessage` instance. 
The second form is taking the `dataLen` parameter to allocate the given number of bytes.
This can speed up the use of the instance, since no re-allocations are done internally as long as the pre-allocated memory is sufficient.

All further constructors will fill the `ModbusMessage` with a defined Modbus message:

## `ModbusMessage(uint8_t serverID, uint8_t functionCode)`
This is usable to set up Modbus standard messages for the Modbus function codes requiring no additional parameter, as 0x07, 0x0B, 0x0C and 0x11.

If you will specify an invalid server ID or a Modbus standard function code taking a different number of parameters, an error message is sent to `Serial` and the message is not generated.

This applies to all constructors in this group.
Parameters to known standard Modbus messages are checked for conformity.

As a `functionCode` you may specify a numeric value or one of the predefined constant names:

```cpp
enum FunctionCode : uint8_t {
  ANY_FUNCTION_CODE       = 0x00,  // only valid to be used by ModbusServer/ModbusBridge!
  READ_COIL               = 0x01,
  READ_DISCR_INPUT        = 0x02,
  READ_HOLD_REGISTER      = 0x03,
  READ_INPUT_REGISTER     = 0x04,
  WRITE_COIL              = 0x05,
  WRITE_HOLD_REGISTER     = 0x06,
  READ_EXCEPTION_SERIAL   = 0x07,
  DIAGNOSTICS_SERIAL      = 0x08,
  READ_COMM_CNT_SERIAL    = 0x0B,
  READ_COMM_LOG_SERIAL    = 0x0C,
  WRITE_MULT_COILS        = 0x0F,
  WRITE_MULT_REGISTERS    = 0x10,
  REPORT_SERVER_ID_SERIAL = 0x11,
  READ_FILE_RECORD        = 0x14,
  WRITE_FILE_RECORD       = 0x15,
  MASK_WRITE_REGISTER     = 0x16,
  R_W_MULT_REGISTERS      = 0x17,
  READ_FIFO_QUEUE         = 0x18,
  ENCAPSULATED_INTERFACE  = 0x2B,
  USER_DEFINED_41         = 0x41,
  USER_DEFINED_42         = 0x42,
  USER_DEFINED_43         = 0x43,
  USER_DEFINED_44         = 0x44,
  USER_DEFINED_45         = 0x45,
  USER_DEFINED_46         = 0x46,
  USER_DEFINED_47         = 0x47,
  USER_DEFINED_48         = 0x48,
  USER_DEFINED_64         = 0x64,
  USER_DEFINED_65         = 0x65,
  USER_DEFINED_66         = 0x66,
  USER_DEFINED_67         = 0x67,
  USER_DEFINED_68         = 0x68,
  USER_DEFINED_69         = 0x69,
  USER_DEFINED_6A         = 0x6A,
  USER_DEFINED_6B         = 0x6B,
  USER_DEFINED_6C         = 0x6C,
  USER_DEFINED_6D         = 0x6D,
  USER_DEFINED_6E         = 0x6E,
};
```

## `ModbusMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1)`
There is one standard Modbus message requiring exactly one `uint16_t` parameter: 0x18, READ_FIFO_QUEUE.
This constructor will help set it up correctly, but of course may be used for any other proprietary function code with the same signature.
Please note that in this case the `p1` parameter will not be range-checked.

## `ModbusMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2)`
This constructor will be one of the most used throughout, as the most relevant Modbus messages have this signature (taking two 16bit parameters).
These are the FCs 0x01, 0x02, 0x03, 0x04, 0x05 and 0x06.
  
## `ModbusMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint16_t p3)`
Modbus standard function code 0x16 (MASK_WRITE_REGISTER) does take three 16bit arguments and will fit into this constructor.
  
## `ModbusMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint16_t *arrayOfWords)`
Another frequently used Modbus function code is WRITE_MULT_REGISTERS, 0x10. 
It requires a 16bit address, a numer of words to write (16bit again), a length byte and a uint8_t* pointer to an array of words to be written.
That is the purpose of this constructor here.
  
## `ModbusMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint8_t *arrayOfBytes)`
A very similar constructor, only that it takes an array of bytes instead of 16bit words. Modbus standard FC 0x0F is using this.

## `ModbusMessage(uint8_t serverID, uint8_t functionCode, uint16_t count, uint8_t *arrayOfBytes)`
This is a 'generic' constructor to pre-fill a freshly created `ModbusMessage` with arbitrary data.
The byte array of length `count` is taken as is and put into the message behind server ID and function code. 
Please note that there is no additional check on validity - you will have to maintain the correctness of the data!

To help setting up such an `arrayOfBytes`, the library provides a service function:

### `uint16_t addValue(uint8_t *target, uint16_t targetLength, T v)`
This service function takes the integral value of type `T` provided as parameter `v` and will write it, MSB first, to the `uint8_t` array pointed to by `target`. If `targetLength` does not indicate enough space left, the copy is not made.

In any case the function will return the number of bytes written. A typical application may look like:

```cpp
uint8_t u = 4;      // 0x04
uint16_t w = 1276;  // 0x04FC 
uint32_t l = 0xDEADBEEF;
uint8_t buffer[24];
uint16_t remaining = 24;

remaining -= addValue(buffer + 24 - remaining, remaining, u);
remaining -= addValue(buffer + 24 - remaining, remaining, w);
remaining -= addValue(buffer + 24 - remaining, remaining, l);
```

After the above code is run `buffer` will contain `04 04 FC DE AD BE EF` and `remaining` will be 17.

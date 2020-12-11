---
layout: default
title: Filling a ModbusMessage
nav_order: 2
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /filling-a-modbusmessage
parent: ModbusMessage
---

# Filling a ModbusMessage

If you are not able to use one of the constructors as are described above, you can use a number of calls to add or modify the contents of a `ModbusMessage`

First, there are `setMessage()` calls that have the same functionality as the constructors, but with one exception:
these calls will return an `Error` value describing what may have happened when creating the message.
`Error` is one of the error codes <a name="errorcodes"></a>defined in `ModbusTypeDefs.h`:
  ```cpp
  enum Error : uint8_t {
    SUCCESS                = 0x00,
    ILLEGAL_FUNCTION       = 0x01,
    ILLEGAL_DATA_ADDRESS   = 0x02,
    ILLEGAL_DATA_VALUE     = 0x03,
    SERVER_DEVICE_FAILURE  = 0x04,
    ACKNOWLEDGE            = 0x05,
    SERVER_DEVICE_BUSY     = 0x06,
    NEGATIVE_ACKNOWLEDGE   = 0x07,
    MEMORY_PARITY_ERROR    = 0x08,
    GATEWAY_PATH_UNAVAIL   = 0x0A,
    GATEWAY_TARGET_NO_RESP = 0x0B,
    TIMEOUT                = 0xE0,
    INVALID_SERVER         = 0xE1,
    CRC_ERROR              = 0xE2, // only for Modbus-RTU
    FC_MISMATCH            = 0xE3,
    SERVER_ID_MISMATCH     = 0xE4,
    PACKET_LENGTH_ERROR    = 0xE5,
    PARAMETER_COUNT_ERROR  = 0xE6,
    PARAMETER_LIMIT_ERROR  = 0xE7,
    REQUEST_QUEUE_FULL     = 0xE8,
    ILLEGAL_IP_OR_PORT     = 0xE9,
    IP_CONNECTION_FAILED   = 0xEA,
    TCP_HEAD_MISMATCH      = 0xEB,
    UNDEFINED_ERROR        = 0xFF  // otherwise uncovered communication error
  };
  ```
  If parameter checks will find a deviation from the standard, the returned `Error` will tell you what it was.

For completeness, here are the `setMessage()` variants:

1. no additional parameter (FCs 0x07, 0x0b, 0x0c, 0x11)<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode);`

2. one uint16_t parameter (FC 0x18)<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1);`

3. two uint16_t parameters (FC 0x01, 0x02, 0x03, 0x04, 0x05, 0x06)<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2);`

4. three uint16_t parameters (FC 0x16)<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint16_t p3);`

5. two uint16_t parameters, a uint8_t length byte and a uint8_t* pointer to array of words (FC 0x10)<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint16_t *arrayOfWords);`

6. two uint16_t parameters, a uint8_t length byte and a uint16_t* pointer to array of bytes (FC 0x0f)<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint8_t *arrayOfBytes);`

7. generic method for preformatted data ==> count is counting bytes!<br>
`Error setMessage(uint8_t serverID, uint8_t functionCode, uint16_t count, uint8_t *arrayOfBytes);`

There is in fact an eighth function, that is not available as a constructor.
It will set the `ModbusMessage` to be an error response, signalling the given `Error`:

8. Error response method<br>
`Error setError(uint8_t serverID, uint8_t functionCode, Error errorCode);`

**Note**: a call to one of the `set...()` methods will erase the previous contents of the `ModbusMessage` before filling it!

## Adding data to a message
If a message has been set up - for instance with server ID and function code only, more data can be added to it.

### `uint16_t add(T value);`
`add()` takes the integral `value` of type `T` and appends it MSB-first to the existing message.
`uint8_t b = 9; msg.add(b);` for instance will append one single byte `09` to the message, whereas `uint32_t i = 122316; msg.add(i);` will append four bytes: `00 01 DD CC` (the hexadecimal representation of 122316, MSB-first!).

The `add()` method will return the number of bytes in the message after the addition.

{: .ml-8 }
Warning
{: .label .label-red}

{: .px-8 }
expressions like `a ? 1 : 2` are converted to `int` by the compiler. So please be sure to cast expressions if you would like to have a shorter representation.
`msg.add(a ? 1 : 2);` most likely will add `00 00 00 01` or `00 00 00 02` to your message, while `msg.add((uint8_t)(a ? 1 : 2));` will append `01` or `02` - which may be what you wanted instead.

### `uint16_t add(T v, ...);`
`add()` may also be used with more than one integral value and will add the given values one by one. 
The returned size is that after adding all of the values in a row.

### `uint16_t add(uint8_t *data, uint16_t dataLen);`
This version of `add()` will append the given buffer of `dataLen` bytes pointed to by `data`.
It also does return the final size of the message.

### `void append(ModbusMessage& m);` and<br> `void append(std::vector<uint8_t>& m):`
With `append()` you may add in another `ModbusMessage`'s content or that of a `std::vector` of bytes.

### `void push_back(const uint8_t& value);`
Like the `std::vector` method of the same name, `push_back()` will append the given byte to the end of the existing message.

---
layout: default
title: Reading from a ModbusMessage
nav_order: 3
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /reading-from-a-modbusmessage
parent: ModbusMessage
---

# Reading data from a message
There are a few methods to get values out of a `ModbusMessage`, to read Modbus standard values as well as free-form data.

## `uint8_t getServerID();`
If the message has at least 2 bytes (a Modbus message needs to have a server ID and function code as a minimum), this method will return the value of the very first, the server ID.
If the message is shorter, you will be returned a `0` value.

## `uint8_t getFunctionCode();`
This method will return the second byte of a Modbus message - the function code, according to the standard.
This method will return a `0` as well, if the function code could not be read.

## `Error getError();`
`getError()` will return the error code in a Modbus error response message. If the message is no error response, you will get a `SUCCESS` code instead.

## The `[]` operator
You can use the well-known bracket operator with a `ModbusMessage` to get the *n*th byte of a message: `uint8_t byte = msg[7];`
If the message is shorter than the requested byte number, you will get a `0` instead.
Opposite to its `std::vector` sibling, the `[]` operator does *not* extend the message length!

## All contents by data/size
Again like the `std::vector`, a `ModbusMessage` provides the `uint8_t *data()` and `uint16_t size()` methods to get a `const` access to the internal data buffer.

## Iterators
`ModbusMessage` has both the `begin()` and `end()` iterator functions to allow iterator looping and range `for` loops:

```cpp
for (auto& byte : msg) {
  Serial.printf("%02X ", byte);
}
Serial.println();
```

## `uint16_t get(uint16_t index, T& value);`
As the `add()` function is able to write integral data values into a message, `get()` is used to read values back.
The `index` parameter gives the starting position for the extraction of a `value` of the integral type `T`.
The method returns the index value after the extraction has taken place.
Example:

```cpp
// Get address and word count for a READ_INPUT_REGISTER request message
uint16_t addr, words;
msg.get(2, addr);
msg.get(4, words);
```

## `uint16_t get(uint16_t index, float& value);` and `uint16_t get(uint16_t index, double& value);` 
These `get()` variants are to extract a 4-byte IEEE754 float or an 8-byte IEEE754 double from a message. The order of bytes in the message is assumed to be "pure IEEE754" MSB-first.

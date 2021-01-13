---
layout: default
title: Miscellaneous functions
nav_order: 4
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /miscellaneous-modbusmessage
parent: ModbusMessage
---

# Miscellaneous functions
There are a few additional methods not fitting in the previous categories. 

## `void setServerID(uint8_t serverID);` and<br> `void setFunctionCode(uint8_t functionCode);`
With these two functions you can change the two Modbus standard values in-place, without changing the message length or other contents.

{: .ml-8 }
Warning
{: .label .label-red}

{: .px-8 }
if the message is shorter than 2 bytes, the two methods will do nothing.

## `uint16_t resize(uint16_t newSize);`
`resize` will change the `size()` of the underlying `std::vector` to the given number of bytes. If the new size is less than the message contents, the bytes exceeding the new size will be discarded.

## `void clear();`
To completely erase the message content, use the `clear()` call. The message will be empty afterwards.

## Assignment
`ModbusMessage`s may be assigned safely to another. The complete content will be copied into the recieving `ModbusMessage`.
As a `ModbusMessage` also implements the `std::move` pattern, a returned `ModbusMessage` may be moved instead of copied, if the compiler will decide so.

## Comparison
`ModbusMessage`s may be compared for equality (`==`) and inequality (`!=`).
If used in a `bool` context, a `ModbusMessage` will be evaluated to `true`, if it has 2 or more bytes as content.

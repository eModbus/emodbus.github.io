---
layout: default
title: ModbusMessage
nav_order: 4
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusmessage
has_children: true
---

# ModbusMessage

Basically all data going back and forth inside the library has the form of a `ModbusMessage` object.
This object will contain all the bytes of a Modbus standard message - request or response - but without anything that is depending on the interface used (RTU or TCP).
A `ModbusMessage` internally is a `std::vector<uint8_t>`, and in multiple aspects can be used as one.

# Error codes

The Modbus standard defines a couple of error codes to be returned as a response in case. The eModbus library extends this list by several additional codes describing various conditions one may encounter when creating, sending or receiving Modbus messages. Error codes in eModbus have a data type of ``Error``.

Please see the list below for the error codes known:
- SUCCESS // 0x00, Successful operation, everything's OK. Modbus standard.
- ILLEGAL_FUNCTION // 0x01, "Illegal function code" - an invalid function code was found. Modbus standard.
- ILLEGAL_DATA_ADDRESS // 0x02, "Illegal data address": the register address given does not fit the server. Modbus standard.
- ILLEGAL_DATA_VALUE // 0x03, "Illegal data value": the data value provided is out of bounds for the requested purpose. Modbus standard.
- SERVER_DEVICE_FAILURE // 0x04, Modbus standard.
- ACKNOWLEDGE // 0x05, Modbus standard.
- SERVER_DEVICE_BUSY // 0x06, Modbus standard.
- NEGATIVE_ACKNOWLEDGE // 0x07, NACK. Modbus standard.
- MEMORY_PARITY_ERROR // 0x08, Modbus standard.
- GATEWAY_PATH_UNAVAIL // 0x0A, Modbus standard.
- GATEWAY_TARGET_NO_RESP // 0x0B, Modbus standard.
- TIMEOUT // 0xE0, a timeout has struck while waiting for a response etc.
- INVALID_SERVER // 0xE1, the server ID is unknown or malformed.
- CRC_ERROR // 0xE2, the Modbus message had a wrong checksum - only for Modbus-RTU
- FC_MISMATCH // 0xE3, function code mismatch between request and the response
- SERVER_ID_MISMATCH // 0xE4, server ID mismatch between request and response
- PACKET_LENGTH_ERROR // 0xE5, the length as announced in the Modbus TCP header did not match the received length
- PARAMETER_COUNT_ERROR // 0xE6, wrong # of parameters for the given function code
- PARAMETER_LIMIT_ERROR // 0xE7, parameter out of bounds for the given function code
- REQUEST_QUEUE_FULL // 0xE8, the request queue is completely filled, no more requests are accepted at this time
- ILLEGAL_IP_OR_PORT // 0xE9, the IP address or port number given as target server are invalid
- IP_CONNECTION_FAILED // 0xEA, IP connection to the given target host failed
- TCP_HEAD_MISMATCH // 0xEB, transaction ID or protocol ID in the TCP request header did not match those of the response 
- EMPTY_MESSAGE // 0xEC, it was attempted to send an empty request message
- UNDEFINED_ERROR // 0xFF, otherwise uncovered communication error

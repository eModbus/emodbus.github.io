---
layout: default
title: ModbusClient Common API
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusclient-common-api
parent: ModbusClient
---

# Common concepts for Modbus clients

Internally the Modbus message contents for requests and responses are held separately from the elements the protocols will require. Hence several API elements are common to both protocols.

## `bool onDataHandler(MBOnData handler)`
This is the interface to register a callback function to be called when a response was received. The only parameter is a pointer to a function with this signature:

```cpp
void func(ModbusMessage msg, uint32_t token);
```

- `msg`: the response message received
- `token`: this is a general concept throughout the library, see below.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
A handler can only be claimed once. If that already has been done before, subsequent calls are returning a `false` value.

## The `token` concept
Each request must be given a user-defined `token` value. This token is saved with each request and is returned in the callback. No processing whatsoever is done on the token, you will get what you gave. This enables a user to keep track of the sent requests when receiving a response.

Imagine an application that has several ModbusClients working; some for dedicated TCP Modbus servers, another for a RS485 RTU Modbus and another again to request different TCP servers.

If you only want to have a single onData and onError function handling all responses regardless of the server that sent them, you will need a means to tell one response from the other. 

Your token you gave at request time will tell you that, as the response will return exactly that token.

## `bool onErrorHandler(MBOnError handler)`
Very similar to the `onDataHandler` call, this allows to catch all error responses with a callback function.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
It is not strictly *required* to have an error callback registered. But if there is none, errors will go unnoticed!

The `onErrorhandler` call accepts a function pointer with the following signature only:

```cpp
void func(Error errorCode, uint32_t token);
```

The parameters are 
- the ``errorCode`` error the requested server generated
- the `token` value as described in the `onDataHandler` section above.

The Library is providing a separate wrapper class `ModbusError` that can be assigned or initialized with any `Error` code and will produce a human-readable error text message if used in `const char *` context. See in in the [Basic use example](https://emodbus.github.io/modbusclient) a way of how to apply it.
  
## `uint32_t getMessageCount()`
Each request that got successfully enqueued is counted. By calling `getMessageCount()` you will be able to read the number accumulated so far.

Please note that this count is instance-specific, so any ModbusClient instance you created will have its own count.

## `void begin()` and<br> `void begin(int coreID)`
This is the most important call to get a ModbusClient instance to work. It will open the request queue and start the background worker task to process the queued requests.

The second form of `begin()` allows you to choose a CPU core for the worker task to run (only on multi-core systems like the ESP32). This is advisable in particular for the `ModbusClientRTU` client, as the handling of the RS485 Modbus is a time-critical and will profit from having its own core to run.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
The worker task is running forever or until the ModbusClient instance is killed that started it. The destructor will take care of all requests still on the queue and remove those, then will stop the running worker task.

## Setting up requests
All interfaces provide a common set of `addRequest()` methods to set up and enqueue a request.
There are seven `addRequest()` variants that closely resemble the `ModbusMessage.setMessage()` calls described in the `ModbusMessage` chapter.
Internally exactly those `setMessage()` calls are used by these `addRequest()` functions.
The major difference to `setMessage()` is the additional `Token` parameter, given as first argument in front of the message parameters:

So we have:
1. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode);`

2. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1);`
  
3. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2);`
  
4. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint16_t p3);`
  
5. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint16_t *arrayOfWords);`
  
6. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint8_t *arrayOfBytes);`

7. `Error addRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t count, uint8_t *arrayOfBytes);`

Again, there is an eighth `addRequest()`, this time taking a pre-formatted `ModbusMessage`:

8. `Error addRequest(ModbusMessage request, uint32_t token);`
Note the switched position of the `token` parameter - necessary for the internal handling of the `addRequest()` methods.

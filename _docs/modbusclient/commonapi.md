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

## `bool onResponseHandler(MBOnResponse handler)`

Alternatively to the use of the ``onDataHandler``/``onErrorHandler`` pair you may provide a single ``onResponseHandler``, that will be called for any response message - regardless if data or error. The function signature of the ``onResponseHandler`` is

```cpp
void func(ModbusMessage msg, uint32_t token);
```

When using the ``onResponseHandler`` pattern, any ``onDataHandler`` or ``onErrorHandler`` callbacks you may have provided will not be effective any more!

To tell a data from an error response, you should use the ``getError()`` call on the ``msg`` ModbusMessage. It will return SUCCESS with a data response and an ``Error`` code else.

## `uint32_t getMessageCount()`
Each request that got successfully enqueued is counted. By calling `getMessageCount()` you will be able to read the number accumulated so far.

Please note that this count (and the error count, respectively) is instance-specific, so each ModbusClient instance you created will have its own count.

## `uint32_t getErrorCount()`
Each error response received will be counted. The `getErrorCount()` method will return the current state of the counter.

## `void resetCounts()`
The internal counters for both messages and errors can be set to zero using this call.

## `void clearQueue()`
This method will immediately remove all pending requests from the queue - in case of an emergency or if a general cleanup is desired. 
If requests have been sent already, the responses to those will be dropped (with a warning log message).

## Setting up requests

### Asynchronous requests (non-blocking)

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
{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
Please take note of the different position of the `token` parameter in this call - necessary for the internal handling of the `addRequest()` methods.

### Synchronous requests (waiting for the response)

Each ``addRequest()`` as described before has a synchronous ``syncRequest()`` pendant. It can be used to set up a request identical to that generated by ``addRequest()`` and send it to the server.

The ``syncRequest()`` calls will wait until the server's response has been received and return that to the caller:

1. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode);`

2. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1);`
  
3. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2);`
  
4. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint16_t p3);`
  
5. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint16_t *arrayOfWords);`
  
6. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t p1, uint16_t p2, uint8_t count, uint8_t *arrayOfBytes);`

7. `ModbusMessage syncRequest(uint32_t token, uint8_t serverID, uint8_t functionCode, uint16_t count, uint8_t *arrayOfBytes);`

8. `ModbusMessage syncRequest(ModbusMessage request, uint32_t token);`

There is a final timeout of 60s, though, after that the call will return in any case, with a TIMEOUT error message of course.

You should be aware that the synchronous calls normally are not well suited to be used on small single-task MCUs, as nothing else may be done in the task (or thread) that is using them until the call is ending. Other parallel tasks on MCUs that support these are not affected of course.

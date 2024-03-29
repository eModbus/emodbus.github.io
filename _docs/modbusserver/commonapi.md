---
layout: default
title: ModbusServer Common API
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusserver-common-api
parent: ModbusServer
---

# Common concepts for Modbus server

All ModbusServer variants have some calls in common, regardless of the protocol or interface they will use.

## `void registerWorker(uint8_t serverID, uint8_t functionCode, MBSworker worker)`
The `registerWorker` method is used to announce callback functions to the Modbus server that will serve certain function codes.

These callback functions must have a function signature of:
`ModbusMessage (*MBSworker) (ModbusMessage request)`,
where the parameter given to the callback is 
- `request`: the ModbusMessage containing the request

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
You may specify different server identifications for registered callbacks. This way the server you are providing can cover more than one Modbus server at a time. You will have to check the requested serverID in the callback to learn which of the registered servers was meant in the request.

There is no limit in registered callbacks, but a second register for a certain serverID/functionCode combination will overwrite the first without warning.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
There is a special function code `ANY_FUNCTION_CODE`, that will register your callback to any function code except those you have explicitly registered otherwise.
This is meant for applications where the standard function code and response handling does not fit the needs of the user.
There will be no automatic `ILLEGAL_FUNCTION_CODE` response anymore for this server!

{: .ml-8 }
Hint
{: .label .label-blue}

{: .px-8 }
You may combine this with the `NIL_RESPONSE` response variant described below to have a server that will gobble all incoming requests.
With RTU, this will give you a copy of all messages directed to another server on the bus, but without interfering with it.

A `MBSworker` callback must return a data object of `ModbusMessage`. 
There are two special response messages predefined:
- `NIL_RESPONSE`: no response at all will be sent back to the requester
- `ECHO_RESPONSE`: the request will be sent back without any modification as a response. 
This is common for the writing function code requests that the Modbus standard defines.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
Do not use the first byte of your `ModbusMessage` as `0xFF`, followed by one of `0xF0` or `0xF1`, as these are used internally for `NIL_RESPONSE` and `ECHO_RESPONSE`!

``registerWorker()`` will also accept ``ANY_SERVER`` as a server ID.

*This is potentially dangerous*, as it will catch all requests to server IDs that are not covered by explicitly defined workers!
Explicitly registered workers for combinations of a distinct server ID and a function code (also ``ANY_FUNCTION_CODE``) will have preference to the generic worker at all times.

It may be needed for some devices in the wild, though, that are (mis)using the server ID for functional purposes.

**Be warned again: using ``ANY_SERVER`` may break your server's function!**

``ANY_SERVER`` still is invalid for issueing requests or attaching server IDs to a bridge.


## `bool unregisterWorker(uint8_t serverID)` and `bool unregisterWorker(uint8_t serverID, uint8_t functionCode)`
A user's request was to be able to drop again previously registered function code callbacks. The application was to have freshly initialized devices react on a proprietary server ID to accept their 'real' server ID with a request. Then the callback for the initial server ID needs to be dropped.

This call allows you to do exactly that - discarding the callbacks for a given server ID/function code combination or for a server ID alltogether (removing all function code callbacks that may exist for that server ID).

The call will return `true` if at least one callback was removed.

## `uint16_t getValue(uint8_t *source, uint16_t sourceLength, T &v)`
Although you will be using `ModbusMessage`'s `get()` function most of the time, there is a complement to the `addValue()` service function described in the ModbusClient section.
`getValue()` will help you reading MSB-first data from an arbitrary data buffer.
This buffer is given as `uint8_t *source`, its length as `uint16_t sourceLength`.
Depending on the data type `T` of the variable given as reference `&v` to the `getValue()` call, the right number of bytes is taken from `source`, converted into type `T` and stored in the variable `v`.
The return value of `getValue()` is the number of bytes consumed from `source` to enable you updating the `source` pointer for the next call.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
Please take care to reduce the `sourceLength` parameter accordingly in subsequent calls to `getValue()` to avoid attempts to read memory behind the end of your ``source`` buffer!

## `MBSworker getWorker(uint8_t serverID, uint8_t functionCode)`
You may check if a given serverID/functionCode combination is being covered by a callback with `getWorker()`. This method will return the callback function pointer, if there is any, and a `nullptr` else.

## `bool isServerFor(uint8_t serverID)`
`isServerFor()` will return `true`, if at least one callback has been registered for the given `serverID`, and `false` else.

## `uint32_t getMessageCount()`
Each request received will be counted. The `getMessageCount()` method will return the current state of the counter.

## `uint32_t getErrorCount()`
Each error response sent will be counted. The `getErrorCount()` method will return the current state of the counter.

## `void resetCounts()`
The internal counters for both messages and errors can be set to zero using this call.

## `ModbusMessage localRequest(ModbusMessage request)`
This function is a simple local interface to issue requests to the server running. Responses are returned immediately - there is no request queueing involved. This call is *blocking* for that reason, so be prepared to have to wait until the response is ready!

A [``ModbusBridge``](https://emodbus.github.io/modbusbridge) will respond to this call for all known serverID/function code combinations, so the delegated request to a remote server may be involved as well.

{: .ml-8 }
Warning
{: .label .label-red}

{: .px-8 }
Be aware that your ``localRequest()`` calls may interfere with requests coming in over the server's interface connection! If you are planning to make use of the parallel calls, be sure to protect the server data access by a mutex or similar construction.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
The `localRequest()` call will work even if the server has not been started (yet) by `start()`, so if you need to communicate in other ways than RTU or TCP, you may make use of that!

## `void listServer()`
Mostly intended to be used in debug situations, `listServer()` will output all servers and their function codes served by the ModbusServer to the ``LOGDEVICE`` as defined in ``Logging.h`` ([see the section on ``Logging`` for it](https://emodbus.github.io/logging)).

---
layout: default
title: ModbusBridge API
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusbridge-api
parent: ModbusBridge
---

# Modbus Bridge API description

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
All API calls for `ModbusServer` are valid for the bridge as well (except the constructors of course) - see the [ModbusServer API description](/modbusserver) for these.

You will have to include the matching header file in your code for the bridge type you want to use:
- `ModbusBridgeEthernet.h` for the Ethernet-based bridge
- `ModbusBridgeWiFi.h` for the WiFi sibling
- `ModbusBridgeRTU` for a bridge listening on a RTU Modbus

Note that there currently is no AsyncTCP bridge (yet), due to differences in internal implementations.

## `ModbusBridge()` and<br> `ModbusBridge(uint32_t TOV)`
These are the constructors for a TCP-based bridge. The optional `TOV` (="timeout value") parameter sets the maximum time the bridge will wait for the responses from external servers. The default value for this is 10000 - 10 seconds.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
This is a value common for all servers connected, so choose the value sensibly to cover all normal response times of the servers. 
Else you will be losing responses from those servers taking longer to respond. Their responses will be dropped unread if arriving after the bridge's timeout had struck.

## `ModbusBridge(HardwareSerial& serial, uint32_t timeout)`,<br> `ModbusBridge(HardwareSerial& serial, uint32_t timeout, uint32_t TOV)` and<br> `ModbusBridge(HardwareSerial& serial, uint32_t timeout, uint32_t TOV, int rtsPin)`
The corresponding constructors for the RTU bridge variant. With the exception of `TOV` all parameters are those of the underlying RTU server.
The duplicity of `timeout` and `TOV` may look irritating at the first moment, but while `timeout` denotes the timeout the bridge will use when waiting for messages as a _server_, the `TOV` value is the _bridge_ timeout as described above.

## `bool attachServer(uint8_t aliasID, uint8_t serverID, uint8_t functionCode, ModbusClient *client)` and<br> `bool attachServer(uint8_t aliasID, uint8_t serverID, uint8_t functionCode, ModbusClient *client, IPAddress host, uint16_t port)`
These calls are used to bind `ModbusClient`s to the bridge. 
The former call is for RTU clients, the latter for their TCP pendants.
The call parameters are:
- `aliasID`: the server ID, that will be visible externally for the attached server. As connected servers in different Modbus environments may have the same server IDs, these need to be unified for requests to the bridge.
The `aliasID` must be unique for the bridge. Attempts to attach the same `aliasID` to another server/client will be denied by the bridge.
The call will return `false` in these cases.
- `serverID`: the server ID of the remote server. This is the ID the server natively is using on the Modbus the client is connected to.
- `functionCode`: the function code that shall be accessed on the remote server.
This may be any FC allowed by the Modbus standard (see [Function Codes](https://emodbus.github.io/modbusmessage-constructors#functioncodes)), including the special `ANY_FUNCTION_CODE` non-standard value. The latter will open the bridge for any function code sent without further checking. See [Filtering](#filtering) for some refined recipes.
- `*client`: this must be a pointer to any `ModbusClient` type, that is connecting to the external Modbus the remote server is living in.

The TCP `attachServer` call has two more parameters needed to address the TCP host where the server is located:
- `host`: the IP address of the target host
- `port`: the port number to connect to the TCP Modbus on that host.

At least one `attachServer()` call has to be made before any of the following make any sense!

## `bool addFunctionCode(uint8_t aliasID, uint8_t functionCode)`
With `addFunctionCode()` you may add another function code for the server known behind the `aliasID`, that will be forwarded by the bridge. One by one you can add those function codes you would like to have served by the bridge.
The `aliasID` has to be attached already before you may use this call successfully. If it is not, the call will return `false`.

## `bool denyFunctionCode(uint8_t aliasID, uint8_t functionCode)`
This call is the opposite to `addFunctionCode()` - the function code you give here as a parameter will **not** be served by the bridge, but responded to with an `ILLEGAL_FUNCTION` error response.
Like its sibling, this call will return `false` if the `aliasID` was not yet `attached` to the bridge.

# Filtering
Given the means of both the underlying `ModbusServer` and the `ModbusBridge` itself, the access to the remote servers can be controlled in multiple ways.

As a principle, the bridge will only react on server IDs made known to it - either by the `attachServer()` bridge call or by the server's `registerWorker()` call for a locally served function code. So a server ID filter is intrinsically always active. Server IDs not known will not be served.

A finer level of control is possible on the function code level. We have two different ways to restrict function code uses:
- allow most, deny some
- deny most, allow some

You will want to choose the method that will require the least effort for your application.

## Allow most, deny some
This is achieved by first attaching a server for the special `ANY_FUNCTION_CODE` value. It will open the server to all possible function codes regardless.

To now restrict the use of a few function codes, you will call `denyFunctionCode()` for each of these. Following that the bridge will return the `ILLEGAL_FUNCTION` error response if a request for that code is sent to the bridge.

## Deny most, allow some
You have to refrain from using the `ANY_FUNCTION_CODE` value here and only use those function codes in `attachServer()` and `addFunctionCode()` that you will allow to be served.

## More subtle methods
There is one basic server rule, that also applies to the bridge: "specialized beats generic". That means, if the server/bridge has a detailed order for a server ID/function code combination, it will use that, even if there is another for the same server ID and ``ANY_FUNCTION_CODE``.
Second rule is "last register counts": whatever was known to the server/bridge on how to serve a server ID/function code combination, will be replaced by a later `registerWorker()` call for the same combination without a trace.

You can make use of that in interesting ways:
- add a local function to serve a certain server ID/function code combination that normally would be served on a remote server. Thus you can mask the external server for that function code at will or add a function code served locally that the remote server does not even know.
- add your own local function to respond with a different error than the default `ILLEGAL_FUNCTION` - or not at all.
- after attaching some explicit function codes for an external server add a local worker for that server ID and `ALL_FUNCTION_CODES` to cover all other codes not explicitly named with a default response.

So besides plain filtering you may modify parts or all of the responses the external server would give.

{: .ml-8 }
Warning
{: .label .label-red}

{: .px-8 }
Registering local worker functions for the serverID of a remote server may completely block the external server! There is no way to go back once you did that!

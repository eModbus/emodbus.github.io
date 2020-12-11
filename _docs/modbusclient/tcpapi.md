---
layout: default
title: ModbusClient TCP API
nav_order: 3
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusclient-tcp-api
parent: ModbusClient
---

# ModbusClientTCP API elements
You will have to include one of these lines in your code to make the `ModbusClientTCP` API available:

```cpp
#include "ModbusClientTCP.h"
```

## `ModbusClientTCP(Client& client)` and<br> `ModbusClientTCP(Client& client, uint16_t queueLimit)`
The first set of constructors does take a `client` reference parameter, that may be any interface instance supporting the methods defined in `Client.h`, f.i. an `EthernetClient` or a `WiFiClient` instance.
This interface will be used to send the Modbus TCP requests and receive the respective TCP responses.

The optional `queueLimit` parameter lets you define the maximum number of requests the worker task's queue will accept. The default is 100; please see the remarks to this parameter in the ModbusClientRTU section.

## `ModbusClientTCP(Client& client, IPAddress host, uint16_t port)` and<br> `ModbusClientTCP(Client& client, IPAddress host, uint16_t port, uint16_t queueLimit)`
Alternatively you may give the initial target host IP address and port number to be used for communications. This can be sensible if you have to set up a ModbusClientTCP client dedicated to one single target host.

## `void setTimeout(uint32_t timeout)` and<br> `void setTimeout(uint32_t timeout, uint32_t interval)` 
Similar to the ModbusClientRTU timeout, you may specify a time in milliseconds that will determine if a `TIMEOUT` error occurred. The worker task will wait the specified time without data arriving to then state a timeout and return the error response for it. The default value is 2000 - 2 seconds.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
The caveat for the ModbusClientRTU timeout applies here as well. The timeout will block the worker task up to three times its value, as two retries are attempted by the worker by sending the request again and waiting for a response. 

The optional `interval` parameter also is given in milliseconds and specifies the time to wait for the worker between two consecutive requests to the same target host. Some servers will need some milliseconds to recover from a previous request; this interval prevents sending another request prematurely. 

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
The interval is also applied for each attempt to send a request, it will add to the timeout! To give an example: `timeout=2000` and `interval=200` will result in 6600ms inactivity, if the target host notoriously does not answer.

## `bool setTarget(IPAddress host, uint16_t port [, uint32_t timeout [, uint32_t interval]]`

This function is necessary at least once to set the target host IP address and port number (unless that has been done with the constructor already). All requests will be directed to that host/port, until another `setTarget()` call is issued.

{: .ml-8 }
Note
{: .label .label-yellow}

{: .px-8 }
Without a `setTarget()` or using a client constructor with a target host no `addRequest()` will make any sense for TCP!

The optional `timeout` and `interval` parameters will let you override the standards set with the `setTimeout()` method **for just those requests sent from now on to the targeted host/port**. The next `setTarget()` will return to the standard values, if not specified differently again.

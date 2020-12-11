---
layout: default
title: Logging
nav_order: 8
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /logging
---

# Logging
The library brings its own logging and tracing mechanism, encoded in the `Logging.h` header file.
It allows for global and file-local verbosity settings.
Log statements above the defined log level will not be compiled in the binary file. 

## Installation
- Put `#include "Logging.h"` in your source file.
- The compile-time definition `-DLOG_LEVEL=...` as a compiler flag or with `#define LOG_LEVEL LOG_LEVEL_...` before including `Logging.h` will set the global log level for the compiler. 
All log statements with log levels higher than that of `LOG_LEVEL` will not be included in the binary.
- In the line before the `#include "Logging.h"`, either write `#undef LOCAL_LOG_LEVEL` to accept the global `LOG_LEVEL` for the current file, or `#define LOCAL_LOG_LEVEL LOG_LEVEL_XXX` - with `XXX` as whatever level you like - to set a different level for the current file.

## Runtime log level 
- the ``MBUlogLvl`` extern variable may be used to set a different log level at runtime. It will be set to ``LOG_LEVEL`` initially. 

{: .ml-8 }
Note
{: .label .label-yellow}
{: .px-8 }
Setting the runtime log level above that of `LOG_LEVEL` will have only an effect for those source files where there is a different `LOCAL_LOG_LEVEL` defined, that is above `LOG_LEVEL`!

## Output channel
All output will be sent to the output defined in `LOGDEVICE`. The default is `Serial`, but any `Print`-derived object will do.

## Log levels
Every log level includes all lower level output as well!
- `LOG_LEVEL_NONE` (0), denoted by `N`, is completely unrestricted, the respective statements will be executed regardless of `LOG_LEVEL` set.
- `LOG_LEVEL_CRITICAL` (1), `C`, is meant for the most important messages in cases of an instable system etc. 
- `LOG_LEVEL_ERROR` (2), `E`, should be used for error messages. This is the default `LOG_LEVEL` value, if none was set at compile time.
- `LOG_LEVEL_WARNING` (3), `W`, is for less relevant problems.
- `LOG_LEVEL_INFO` (4), `I`, for informative output.
- `LOG_LEVEL_DEBUG` (5), `D`, intended to print out detailed information to help debugging.
- `LOG_LEVEL_VERBOSE` (6), `V` - output everything and all minor details.

## Logging statements
In all statements, `X` has to be one of the log level letters as specified above.
- `LOG_X(format, ...);` is a printf-style statement that will print a standard line header like 
```
 +----------------------------------------------- log level
 |     +----------------------------------------- millis()
 |     |    +------------------------------------ source file name
 |     |    |                       +------------ line number
 |     |    |                       |    +------- function name
 |     |    |                       |    |    +-- user output
 v     v    v                       v    v    v 
[N] 101973| main.cpp             [ 258] loop: 05/03 @8/13? (heap=343524)
```
  followed by the specified formatted data.
- `LOGRAW_X(format, ...);` does the same, but without the line header. This is handy to collect several log output on one line
- `HEXDUMP_X(label, address, length)` will print out a hexadecimal and ASCII dump of `length` bytes, starting at `address`:
  Dump header:
```
 +-------------------------------- log level
 |  +----------------------------- user-defined label
 |  |                 +----------- starting address
 |  |                 |        +-- number of bytes
 v  v                 v        v
[N] Bridge response: @3FFB49FC/29:
```
  Dump body:
```
    +----------------------------------------------------------- offset (hex)
    |     +----------------------------------------------------- hexadecimal dump
    |     |                                                  +-- ASCII dump
    v     v                                                  v
  | 0000: 05 03 0D 49 38 52 72 49  32 30 8A 00 00 00 00 00  |...I8RrI20......|
  | 0010: 00 00 00 45 4D 48 00 09  01 45 4D 48 00           |...EMH...EMH.   |
```

### ANSI colors
If you are using a terminal to view the logging output that knows about ANSI commands, you can add some predefined color codes into your log statements:
- `LL_RED`
- `LL_YELLOW`
- `LL_GREEN`
- `LL_BLUE`
- `LL_CYAN` and 
- `LL_MAGENTA`

Please make sure that you have put in a `LL_NORM` code after the colored section is to end to switch back to the regular output colors.

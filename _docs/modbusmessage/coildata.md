---
layout: default
title: CoilData type
nav_order: 1
description: "eModbus is an all-inclusive Modbus implementation, created for ESP32 and Arduino"
permalink: /modbusmessage-coildata
parent: ModbusMessage
---
# The ``CoilData`` type

Part of the Modbus standard is the coils data type, that also is used for the digital inputs function codes.
This data type basically is a set of bits that can have values of ``1`` and ``0`` only.
These values commonly are interpreted like ``1`` = ``ON`` = ``true`` or ``0`` = ``OFF`` = ``false``.

The C++ datatype ``bool`` is almost identical to a Modbus ``coil``, so it is obvious to use the one for the other.

Unfortunately the Modbus specification and the C++ data type have some deficits if it comes to use ``bool`` as coil:
- ``bool`` is no basic storage unit in C++, but always is part of at least a ``uint8_t`` or ``byte`` that can hold 8 ``bool`` bits, but not address these individually.
- Therefore there is no such thing as an array of ``bool`` in C++, that would be like an array of ``coils`` in the Modbus world.
- The order Modbus requires the coil data to be delivered in messages is inverse to the order normally used in C++.

To cope with these shortcomings the ``CoilData`` type was added to **eModbus**.

## *Usage notes* 

- There is no obligation to use the ``CoilData`` type with eModbus at all. If you prefer you may use your own representation of coils any time.
- The current implementation is a beta version only. While it has been tested thoroughly, it surely will have bugs!
- internal code is currently focused on functionality, not speed. There will be changes in the future to improve performance.

## ``CoilData`` interface

### Constructors ``CoilData()``, ``CoilData(uint16_t size)`` and ``CoilData(uint16_t size, bool initValue)``

A ``CoilData`` object constructed without any parameters will have a coil capacity of 0. 
The capacity can only be changed by assigning another ``CoilData`` object or a "bit image array" (see below).

The second constructor is the regular one, it will make room for the given number of coils internally. 
Please note that this size is limited to 2000 coils by the Modbus standard (due to the capacity of a standard Modbus message that can be as long as 255 bytes only).

The third form allows you to specify the value all freshly created coils are preset: if ``initValue`` is ``true``, all coils will get an ``1`` value. 
If left out, or set to ``false``, the coils will be initialized with ``0``.

### Bit image array constructor ``CoilData(const char *initVector)``

You may alternatively specify the size and initial value of the ``CoilData`` object by a *bit image array*.
This is a character array containing ``1`` and ``0`` characters in the order of the coils in the set.
``"11110000"`` for instance is an 8-coil bit image array that, when used in the constructor, will allocate space for 8 coils in the object and init these with the given ``1`` and ``0`` values in order:
- coil 0 will be ``1``
- coil 1 will be ``1``
- coil 2 will be ``1``
- coil 3 will be ``1``
- coil 4 will be ``0``
- coil 5 will be ``0``
- coil 6 will be ``0``
- coil 7 will be ``0``

**Please note**: coils are numbered starting with 0, as usually done in C++ arrays!

Bit image arrays may contain other characters to give visible structure that will be ignored, like spaces ``"1111 0000 1010 1101"``, or even comments: ``"pump A=1, pump B=0, vent 34=1"``.
If you need to include one of ``1`` or ``0`` as commentary that must not be interpreted as coil value, you must prefix it with ``_`` (underscore).
A bit image array of ``"We have _16 coils here: 0101 1100 1110 0111"`` correctly will generate 16 coils, despite of the (escaped) ``1`` from ``16``!

### Assignment

``CoilData`` objects may be assigned safely to each other. 
The source object will completely replace the former data in the target, source and target will be identical afterwards.

The ``CoilData`` type is supporting the C++ Move pattern, so it is possible to use ``CoilData`` objects as return values from functions etc.

A ``CoilData`` object also may be assigned a bit image array as decribed before. 
The contents of the array will also completely replace the data that previously was in the target object.
```
CoilData c(12);
c = "0101 1111 0000 1010";
```
will make ``c`` 16 coils wide and init these as defined in the array.

### Comparison operators

The equality (``==``) and inequality (``!=``) comparison operators are supported to compare ``CoilData`` objects with another ``CoilData`` object or with a bit image array.
```
CoilData c("11100");
CoilData d("11111111");

if (c == "11100"); // true!
if (c != d);       // true!
if (d == c);       // false!
if (d != "11111111"); // false!
```

### Type conversion

There are two type conversion operators for ``CoilData``:
- ``bool``: if used in a ``bool`` context, the object will evaluate to ``true`` if there are any coils defined. Hence an empty ``CoilData`` will return ``false`` instead.
- ``vector<uint8_t>``: if assigned to a ``std::vector<uint8_t>`` object, the ``CoilData`` object will return its data content as a vector of bytes in the order the coils are represented internally.

### (Re-)init complete coil set to 1 or 0, ``void init()`` and ``void init(bool value)``

Using the ``init()`` call, all coils (if any) in the ``CoilData`` object will be set to ``0`` or ``1``, if the ``value`` parameter is given as ``true``.

### Information on the ``CoilData`` object

These calls will return the value of internal parameters of an object:
- ``uint16_t coils()`` will return the coil capacity of the object.
- ``bool empty()`` is ``true``, if there are no coils in the object and ``false`` else.
- ``uint8_t size()`` gives the size of the internal buffer in number of bytes; in combination with
- ``uint8_t *data()`` returning a pointer to the internal buffer, these may be used to use the coil data in classic ``uint8_t`` array manner.
- ``uint16_t coilsSetON()`` and``uint16_t coilsSetOFF()`` will return the number of coils in the object that are set to ``1`` or ``0``, respectively.

### Reading coil values

#### ``bool operator[](uint16_t index)``

The array element operator ``[]`` can be used to read the value of a single coil.
The ``index`` must be within the coils in the ``CoilData`` object - else, ``false`` will be returned.
Due to the lack of individual addressability of a ``bool`` packed in bytes, this operator is **read-only**.
To set any coil to another value, see [Changing coil values](#changing-coil-values) below.

**Please note again**: coil numbering starts at 0, the highest possible coil number is the coils size of the object - 1.


// slice: return a new CoilData object with coils shifted leftmost
  // will return empty set if illegal parameters are detected
  // Default start is first coil, default length all to the end
  CoilData slice(uint16_t start = 0, uint16_t length = 0);

### Changing coil values

  // Set functions to change coil value(s)
  // Will return true if done, false if impossible (wrong address or data)

  // set #1: alter one single coil
  bool set(uint16_t index, bool value);

  // set #2: alter a group of coils, overwriting it by the bits from newValue
  bool set(uint16_t index, uint16_t length, vector<uint8_t> newValue);

  // set #3: alter a group of coils, overwriting it by the bits from unit8_t buffer newValue
  bool set(uint16_t index, uint16_t length, uint8_t *newValue);

  // set #4: alter a group of coils, overwriting it by the coils in another CoilData object
  // Setting stops when either target storage or source coils are exhausted
  bool set(uint16_t index, const CoilData& c);

  // set #5: alter a group of coils, overwriting it by a bit image array
  // Setting stops when either target storage or source bits are exhausted
  bool set(uint16_t index, const char *initVector);

### Debug helper

#if !ISLINUX
  // Helper function to dump out coils in logical order
  void print(const char *label, Print& s);
#endif

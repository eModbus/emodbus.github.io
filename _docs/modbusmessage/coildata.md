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

#### ``CoilData slice(uint16_t start, uint16_t length)``, <br>``CoilData slice(uint16_t start)`` and <br>``CoilData slice()``

A "slice" is part of or a complete ``CoilData`` object as another ``CoilData`` object, i.e. a copy.
The first form of ``slice()`` takes the first coil number to start at and the number of coils to copy, the second, omitting the number of coils, will return all coils from the start to the end of the source ``CoilData``.
The last variety will copy the complete source coil set.

As slices are ``CoilData`` themselves, all functions of ``CoilData`` can be applied to a slice. 
```
uint16_t numberOfSetCoilsInSlice = myCoils.slice(4,7).coilsSetON(); // return all coils set to on within coils #4 to #10
myCoils.slice(1,8) = "11011100"; // will in fact work, but is senseless, because the copy is thrown away afterwards!
```
Slices are handy to respond with coil values to a Modbus request. Please refer to the TCPcoilsExample in the ``examples`` code directory to see an application.

If the ``start`` parameter is out-of-bounds of the ``CoilData`` object, or the ``length`` would exceed the maximum coil number, an empty ``CoilData`` object will be returned.

### Changing coil values

Coil values are altered by the ``set()`` group of calls. 
Besides the basic change of a single coil value, various sources can be used to set one or many coils to new values at once.
If start indexes, lengths or other data attributes are wrong in respect of the ``CoilData`` object they are applied to, the ``set()`` operation will fail and return ``false``.
If the change was successful, the return value will be ``true`` instead.

#### ``bool set(uint16_t index, bool value)``

This very basic ``set`` will write the given ``value`` into the coil pointed to by ``index``.

#### ``bool set(uint16_t index, uint16_t length, vector<uint8_t> newValue)``

The ``vector<uint8_t>`` is used as a source for coil values in the sequence of bits in the bytes of the vector.
So the coil with number ``index`` is set to bit 0 of the first byte of the vector, coil ``index+1`` is set to bit 1 of the same byte and so on, until ``length`` coils are set.
If the vector contains to few bytes to cover all coils, the operation will fail.

#### ``bool set(uint16_t index, uint16_t length, uint8_t *newValue)``

Similar to the preceeding call, this will change several coils in a row.
The values are taken from the bits of the bytes in ``newValue`` in order.

**Please note**: as there is no "natural" end mark for the ``newValue`` pointer, erroneous values for length may result in data being used as coil values that is off the area one might have intended! So better doublecheck if your pointer and lengths are correct.

This ``set()`` variant is useful to change ``CoilData`` by values read from a Modbus message, f.i. coming with a 0x0F "WRITE_MULT_COILS" request.

#### ``bool set(uint16_t index, const CoilData& c)``

This call will write the coils from the given ``CoilData`` object ``c`` into the target, starting at coil number ``index``. 
The coils in ``c`` will be taken from #0 on.
Contrary to the two calls described before, here the copy will be made regardless of the sizes of the two ``CoilData`` objects involved:
- if the source is larger than the available target space, the copy will stop at the very last coil in the target.
- a source coil set shorter than the available target space will be copied completely, the remaining target coils are left untouched.
- if the source is an empty ``CoilData``, nothing will be altered.

```
CoilData A("1111 1111 1111 1111");
CoilData B("0011 0011");
A.set(8, B);    // A is "1111 1111 0011 0011" now!
B.set(4, A);    // B is "0011 1111" now
```

#### ``bool set(uint16_t index, const char *initVector)``

This final ``set()`` is similar to the previous, with the exception that the new coil values here are taken from a bit image array as described above.
The same rules apply, though: too long arrays will be copied up to the last fitting value only, shorter ones will leave the remainder unchanged.

### Debug helper

Sometimes it is difficult to see from hex dumps or plain number output what the values of a given coil set are. 
There is a helper function that will print out all coils of a ``CoilData`` object in a nicely (well, sort of...) formatted way.

#### ``void print(const char *label, Print& s)``

The ``label`` may be chosen as you like and is printed initally before the values.
The ``s`` reference shall point to a ``Print`` type object - this mostly will be ``Serial`` for the standard monitor output.
Output will look like seen below. 
The ``Coil index`` lines have been added here for explanation only, the ``myCoils.print("Initial coil state", Serial);`` used here is responsible for the last line only:
```
Coil index          0    4    8    12   16   20   24   28   32       
                    |    |    |    |    |    |    |    |    |        
Initial coil state: 0001 0111 0000 0110 0000 0000 0000 1011 010      
```
**Please note**: this function is not available with Linux, as there is no such concept as ``Print``, that is part of the Arduino environment only!

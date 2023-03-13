# dataclasses-struct

[![PyPI version](https://img.shields.io/pypi/v/dataclasses-struct)](https://pypi.org/project/dataclasses-struct/)
[![Python versions](https://img.shields.io/pypi/pyversions/dataclasses-struct)](https://pypi.org/project/dataclasses-struct/)
[![Tests status](https://github.com/harrymander/dataclasses-struct/actions/workflows/test.yml/badge.svg?event=push)]()
[![Code coverage](https://img.shields.io/codecov/c/gh/harrymander/dataclasses-struct)](https://app.codecov.io/gh/harrymander/dataclasses-struct)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/harrymander/dataclasses-struct/blob/main/LICENSE)

A simple Python package that combines
[`dataclasses`](https://docs.python.org/3/library/dataclasses.html) with
[`struct`](https://docs.python.org/3/library/struct.html) for packing and
unpacking Python dataclasses to fixed-length `bytes` representations.

```python
from typing import Annotated
import dataclasses_struct as dcs

@dcs.dataclass()
class Test:
    x: int  # or dcs.UInt64
    y: dcs.Float32
    z: dcs.Uint8
    s: Annotated[bytes, 10]
```

```
>>> t = Test(100, -0.25, 0xff, b'12345')
>>> t
Test(x=100, y=-0.25, z=255, s=b'12345')
>>> packed = t.pack()
>>> packed
b'd\x00\x00\x00\x00\x00\x00\x00\x00\x00\x80\xbe\xff12345\x00\x00\x00\x00\x00'
>>> Test.from_packed(packed)
Test(x=100, y=-0.25, z=255, s=b'12345\x00\x00\x00\x00\x00')
```

## Installation

This package is available on pypi:

```
pip install dataclasses-struct
```

To work correctly with [`mypy`](https://www.mypy-lang.org/), an extension is
required; add to your `mypy.ini`:

```ini
[mypy]
plugins = dataclasses_struct.ext.mypy_plugin
```

## Usage

```python
from typing import Annotated  # use typing_extensions on Python <3.9
import dataclasses_struct as dcs

endians = (
    dcs.NATIVE_ENDIAN_ALIGNED,  # uses system endianness and alignment
    dcs.NATIVE_ENDIAN,  # system endianness, packed representation
    dcs.LITTLE_ENDIAN,
    dcs.BIG_ENDIAN,
    dcs.NETWORK_ENDIAN,
)

@dcs.dataclass(endians[0])  # if no endian provided, defaults to NATIVE_ENDIAN_ALIGNED
class Test:

    # Single char type (must be bytes)
    single_char_1: dcs.Char
    single_char_2: bytes  # alias for Char

    # Boolean
    bool_1: dcs.Bool
    bool_2: bool  # alias for Bool

    # Integers
    int8: dcs.Int8
    uint8: dcs.Uint8
    int16: dcs.Int16
    uint16: dcs.Uint16
    int32: dcs.Int32
    uint32: dcs.Uint32
    uint64: dcs.Uint64
    int64_1: dcs.Int64
    int64_2: int  # alias for Int64

    # Only supported with NATIVE_ENDIAN_ALIGNED
    unsigned_size: dcs.Size
    signed_size: dcs.SSize
    pointer: dcs.Pointer

    # Floating point types
    single_precision: dcs.Float  # or Float32
    double_precision_1: dcs.Double  # or Float64
    double_precision_2: float  # alias for Float64

    # bytes arrays: arrays shorter than size will be padded with b'\x00'
    array: Annotated[bytes, 100]  # integer > 0
```

Decorated classes are transformed to a standard Python
[dataclasses](https://docs.python.org/3/library/dataclasses.html) with
boilerplate `__init__`, `__repr__`, `__eq__` etc. auto-generated. Additionally,
two methods are added to the class: `pack`, a method for packing an instance of
the class to `bytes`, and `from_packed`, a class method that returns a new
instance of the class from its packed `bytes` representation.

Default attribute values will be validated against their expected type and
allowable value range. For example,

```python3
import dataclasses_struct as dcs

@dcs.dataclass()
class Test:
    x: dcs.Uint8 = -1
```

will raise a `ValueError`. This can be disabled by passing `validate=False` to
the `dataclasses_struct.dataclass` decorator.
